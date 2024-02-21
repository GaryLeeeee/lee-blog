---
title: 如何优雅停止Go进程？
date: 2024-02-21 19:35:43
tags:
categories: [Golang]
---

## 一、什么是优雅停止？
顾名思义，优雅停止就是优雅地停止正在运行的程序，目的是为了让程序停止的时候尽可能完成还没完成的逻辑，对于一个HTTP服务来说就是尽可能完成还没完成的请求并且不再接收新请求。

反之，如果不优雅停止，程序会将提供HTTP服务的goroutine协程直接中断，当前还没完成的请求也会被粗暴地停止。

## 二、如何优雅停止？
在Go中要做到优雅停止，核心是捕获停止**信号量**，再通过通道channel来完成协程间通信和控制。

优雅停止的大概流程如下：
1. 捕获进程停止信号，如`SIGINT`、`SIGTERM`
2. 捕获到信号后，需要在n秒后强制停止进程（足够让进程优雅停止了）
3. 停止接收新的请求
4. 等待进程中正在进行的请求完成
5. 请求全部完成/倒计时n秒技术，进程正式停止

简单代码如下：
```go
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
)

func main() {
	// ...
	// 创建错误通道
	errChan := make(chan error)
	// 监听ctrl+c kill
	go func() {
		s := make(chan os.Signal, 1)
		// 后面信号量不传默认全部
		signal.Notify(s, syscall.SIGINT, syscall.SIGTERM)
		errChan <- fmt.Errorf("%s", <-s)
	}()
	// ...
}

```

完成代码可查看[基于Go kit+Gin开发一个Web项目Demo](https://github.com/GaryLeeeee/lee-code-repository/tree/master/lee-http-code-go/main.go)

## 三、什么是信号量？
**信号量**是并发编程中常见的一种同步机制，它是一种变量活着抽象数据类型，用于控制并发系统中多个进程对公共资源的访问（原子性）。

### 1、信号量跟优雅停止有什么关系？
程序的停止实际上是接收到了某种**信号量**，比如：
* linux下`ctrl`+`c`实际上是发送了一个`SIGINT`信号（mac下是`control`+`c`）
* `kill pid`实际上是发送了一个`SIGTERM`信号
* `kill -9 pid`实际上是发送了一个`SIGKILL`信号（<font color=red>`SIGKILL`无法被捕获、阻塞或忽略</font>）
* `docker stop`实际上是向容器内进程发送了一个`SIGTERM`信号，10s后再发送一个`SIGKILL`信号（足够让程序做优雅停止了）

### 2、信号量有哪些？
![](/images/golang/如何优雅停止Go进程？.png)
