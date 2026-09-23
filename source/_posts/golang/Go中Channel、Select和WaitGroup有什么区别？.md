---
title: Go中Channel、Select和WaitGroup有什么区别？
date: 2026-09-23 15:27:36
tags: [Golang, Channel]
categories: [Golang]
---

`Channel`、`select`和`sync.WaitGroup`都经常出现在Go并发代码中，但它们解决的不是同一个问题。

可以先记住三句话：

- `Channel`：在Goroutine之间传递数据或信号。
- `select`：同时等待多个Channel，哪个先就绪就处理哪个。
- `WaitGroup`：不关心结果，只等待一组任务全部结束。

## 一、Channel有什么用？

Channel是Goroutine之间的类型安全管道，可以传递普通数据，也可以只传递“某件事发生了”的信号。

```go
package main

import "fmt"

func main() {
	resultCh := make(chan int)

	go func() {
		resultCh <- 100
	}()

	result := <-resultCh
	fmt.Println(result)
}
```

`resultCh <- 100`表示发送数据，`<-resultCh`表示接收数据。

### 1、无缓冲Channel

```go
ch := make(chan int)
```

无缓冲Channel要求发送方和接收方同时准备好。发送不只是交数据，也会等待对方完成交接。

适合：

- 一个Goroutine把结果交给另一个Goroutine。
- 下游没有准备好时，上游应该暂停。
- 必须确认对方已经接收后才能继续。

### 2、有缓冲Channel

```go
ch := make(chan int, 10)
```

缓冲区没满时，发送方不需要等待接收方。它适合暂时吸收生产速度和消费速度的差异。

适合：

- 任务队列和Worker Pool。
- 汇总数量已知的并发结果。
- 限制同时执行的任务数量。

Channel缓冲区不是无限队列。如果数据可能长时间堆积，应该考虑明确的限流、持久化队列或消息中间件。

## 二、Select有什么用？

`select`专门用来等待Channel操作，写法类似`switch`：

```go
select {
case result := <-resultCh:
	fmt.Println("收到结果", result)

case <-ctx.Done():
	fmt.Println("任务取消")
}
```

如果没有Channel准备好，`select`会等待；如果多个Channel同时准备好，Go会从可执行的分支中选择一个。

### 1、同时等待结果和超时

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	resultCh := make(chan string, 1)

	go func() {
		time.Sleep(500 * time.Millisecond)
		resultCh <- "query success"
	}()

	select {
	case result := <-resultCh:
		fmt.Println(result)

	case <-ctx.Done():
		fmt.Println("query failed:", ctx.Err())
	}
}
```

这是`select`最常见的场景：业务结果、超时和取消中哪个先到就处理哪个。

上面的`resultCh`使用1个缓冲位，即使主Goroutine因超时先退出，工作Goroutine也不会永久卡在发送结果上。更完整的做法是把`ctx`继续传给底层查询，让实际工作也能停止。

### 2、同时监听多个数据源

```go
for {
	select {
	case order := <-orderCh:
		handleOrder(order)

	case refund := <-refundCh:
		handleRefund(refund)

	case <-ctx.Done():
		return
	}
}
```

这种写法适合事件循环、多个上游数据源和需要随时停止的后台任务。

### 3、非阻塞发送

```go
select {
case eventCh <- event:
	fmt.Println("发送成功")

default:
	fmt.Println("Channel已满，放弃本次事件")
}
```

`default`表示所有Channel都没有准备好时立即执行，适合允许丢弃的日志或通知。不要在无休止的循环中滥用`default`，否则容易让CPU空转。

## 三、WaitGroup有什么用？

`sync.WaitGroup`可以看成一个任务计数器：

- `Add(1)`：增加一个待完成任务。
- `Done()`：当前任务完成，计数减1。
- `Wait()`：等待计数变为0。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {
		wg.Add(1)

		go func(id int) {
			defer wg.Done()

			time.Sleep(time.Duration(id) * 100 * time.Millisecond)
			fmt.Printf("task %d finished\n", id)
		}(i)
	}

	wg.Wait()
	fmt.Println("all tasks finished")
}
```

`WaitGroup`只知道“还有几个任务没完成”，不会返回这些内容：

- 任务返回值。
- 哪个任务先完成。
- 任务错误。
- 超时和取消信号。

Go 1.25及以上还可以使用`WaitGroup.Go`，它会自动完成`Add`、启动Goroutine和`Done`：

```go
var wg sync.WaitGroup

wg.Go(func() {
	doWork(1)
})

wg.Go(func() {
	doWork(2)
})

wg.Wait()
```

需要兼容旧版Go时，继续使用`Add`、`Done`和`Wait`即可。

## 四、Channel和WaitGroup有什么区别？

| 需求 | Channel | WaitGroup |
| --- | --- | --- |
| 等待所有任务完成 | 可以实现，但通常更复杂 | 最适合 |
| 传递任务或结果 | 支持 | 不支持 |
| 持续生产和消费 | 适合 | 不适合 |
| 谁先完成就先处理谁 | 支持 | 不支持 |
| 配合`select`处理超时 | 支持 | 不直接支持 |
| 实现Worker Pool | 适合 | 只适合辅助等待结束 |

一句话概括：**WaitGroup管“做完没有”，Channel管“传了什么”。**

## 五、Channel和WaitGroup怎么组合使用？

并发任务既需要返回结果，又需要在全部完成后关闭Channel时，可以同时使用两者：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type Result struct {
	TaskID int
	Value  int
}

func calculate(id int) int {
	time.Sleep(time.Duration(4-id) * 100 * time.Millisecond)
	return id * id
}

func main() {
	var wg sync.WaitGroup
	resultCh := make(chan Result, 3)

	for i := 1; i <= 3; i++ {
		wg.Add(1)

		go func(id int) {
			defer wg.Done()

			resultCh <- Result{
				TaskID: id,
				Value:  calculate(id),
			}
		}(i)
	}

	// 只有在所有发送方结束后，才能关闭Channel。
	go func() {
		wg.Wait()
		close(resultCh)
	}()

	for result := range resultCh {
		fmt.Printf("task=%d value=%d\n", result.TaskID, result.Value)
	}
}
```

这段代码的分工是：

- `resultCh`传递每个任务的结果。
- `WaitGroup`确认所有生产者已经结束。
- 额外的Goroutine在所有生产者结束后关闭`resultCh`。
- `for range`持续读取，直到Channel关闭。

不能让某个工作Goroutine率先关闭`resultCh`，因为其他Goroutine可能还要发送数据，向已关闭Channel发送会直接Panic。

## 六、什么场景适合用？

### 适合使用Channel

- 生产者和消费者之间传递任务。
- 多个Goroutine汇总计算结果。
- 流水线中上一阶段向下一阶段发送数据。
- 通过关闭`chan struct{}`广播停止信号。
- 使用有缓冲Channel做简单并发限制。

### 适合使用Select

- 同时等待多个Channel。
- 同时等待业务结果和`ctx.Done()`。
- 同时处理结果、超时和取消。
- 发送数据时也要及时响应取消信号。
- 明确需要“尝试发送”或“尝试接收”的非阻塞操作。

### 适合使用WaitGroup

- 启动多个相互独立的任务，主流程等全部完成。
- 关闭结果Channel前，等待所有生产者退出。
- 程序退出前，等待后台收尾任务结束。

## 七、什么场景必须用？

从语言角度来说，大多数并发问题都有其他实现方式，所以Channel和`WaitGroup`并不是绝对的“必须”。

但以下情况基本应该使用`select`：

- 同一个Goroutine要等待多个Channel中的任意一个。
- 阻塞发送或接收时，还必须响应`ctx.Done()`。
- 某个依赖的API已经以Channel暴露数据、定时或取消事件。

例如，如果不使用`select`，下面的发送一旦阻塞，就无法及时响应取消：

```go
select {
case resultCh <- result:
	return nil

case <-ctx.Done():
	return ctx.Err()
}
```

只是“启动5个任务，全部完成后继续”时，`WaitGroup`虽然不是唯一选择，但通常是最简单的选择。

## 八、哪些场景不适合用Channel？

| 需求 | 更合适的工具 |
| --- | --- |
| 保护多个Goroutine共享的Map或结构体 | `sync.Mutex`、`sync.RWMutex` |
| 简单计数器 | `atomic` |
| 只等待一组任务结束 | `sync.WaitGroup` |
| 等待并发任务并收集第一个错误 | `errgroup` |
| 请求超时和取消 | `context.Context`配合`select` |
| 跨进程传递任务 | Kafka、RabbitMQ等消息队列 |
| 普通的同步函数调用 | 直接调用函数 |

`WaitGroup`也不会保护共享数据。多个Goroutine同时写Map或Slice时，即使最后调用了`Wait()`，仍然需要锁、Channel所有权转移或其他同步设计。

## 九、常见错误

### 1、没有接收方导致死锁

```go
func main() {
	ch := make(chan int)
	ch <- 1
}
```

无缓冲Channel的发送必须有接收方。上面只有主Goroutine，它会卡在`ch <- 1`，永远无法走到接收逻辑。

### 2、接收方关闭Channel

通常应由发送方关闭Channel，因为发送方才知道以后是否还会发送数据。多个发送方共用一个Channel时，由统一的协调者在所有发送方结束后关闭。

### 3、忘记关闭导致`range`不退出

```go
for value := range ch {
	fmt.Println(value)
}
```

`range`会一直接收，直到Channel被关闭。如果发送方已经退出却没有关闭Channel，接收方会永久等待。

### 4、在Goroutine内调用`Add`

不推荐：

```go
go func() {
	wg.Add(1)
	defer wg.Done()
	doWork()
}()

wg.Wait()
```

`Wait()`可能在新Goroutine执行`Add(1)`之前就看到计数为0并直接返回。使用传统写法时，应在启动Goroutine之前调用`Add`。

### 5、Goroutine泄漏

接收方提前退出，发送方却仍然阻塞在Channel上，这个Goroutine就可能永远无法结束。需要提前结束的流水线，应配合`context.Context`或专门的`done`Channel通知上游停止。

## 十、怎么选择？

可以按下面的顺序判断：

1. 只需要等待全部任务完成：使用`WaitGroup`。
2. 需要在Goroutine之间传递任务、结果或信号：使用Channel。
3. 需要等待多个Channel、超时或取消：使用`select`。
4. 需要并发任务的错误返回和统一取消：优先考虑`errgroup`。
5. 只是保护一份共享数据：优先考虑`Mutex`，不要为了使用Channel而使用Channel。

最后再记一次：**Channel管数据和信号，`select`管多路等待，`WaitGroup`管全部完成。**

## 参考资料

- [Go语言规范：Channel和Select](https://go.dev/ref/spec)
- [Go官方文档：sync.WaitGroup](https://pkg.go.dev/sync#WaitGroup)
- [Go Concurrency Patterns：Pipelines and cancellation](https://go.dev/blog/pipelines)
- [Go Concurrency Patterns：Context](https://go.dev/blog/context)
