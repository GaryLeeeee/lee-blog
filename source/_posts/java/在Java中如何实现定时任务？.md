---
title: 在Java中如何实现定时任务？
date: 2023-12-02 17:08:09
tags:
categories: [定时任务]
---

## 一、定时任务是什么？有什么用？
定时任务可以帮助我们在某个时间点触发某个动作，如凌晨0点要结算榜单、晚上8点要下发用户PUSH等。

总结一下在实际开发中定时任务的几个使用场景：
* **定时同步数据/刷新缓存**：比如活动配置等不需要数据强一致性的场景，就可以用定时任务定时更新的方式保证数据最终一致性即可，同时也能避免缓存击穿的问题
* **定时发送消息**：比如每天晚上8点要下发PUSH提醒用户签到等，这种时间点就比较固定
* **数据补偿**：为了避免正常的业务逻辑中可能出现异常导致数据异常的问题，我们可能还得定时进行数据补偿，比如异步消息消费失败后重试、定时下发全量数据进行同步防止中间因为某些网络等原因而导致的数据丢失问题等
* **定时开关活动**：比如活动要在8点开始、9点结束，那么就可以通过定时任务来做判断从而对活动的状态进行变更

## 二、在Java中如何实现定时任务？
在Java中实现定时任务主要有以下几种方式：
* **死循环等待**：借助Thread的sleep方法进行定时阻塞
* **Spring的@Scheduled注解**
* **使用Timer类**
* **使用ScheduleExecutorService**
* **使用Quartz框架**：Quartz是一个流行的开源任务调用框架，支持任务的并行执行和动态调用
* **使用xxl-job**：xxl-job是一个分布式定时任务调度平台，可以配置HTTP接口等进行定时回调。支持集群部署，可以满足高并发、大数据量的任务调用需求

### 1、死循环等待
**优点**：实现简单
**缺点**：不适用范围周期大的场景
**使用场景**：比如线程池定时打印线程池状态、延迟双删？
```java
public class TaskClass {
    public static void main(String[] args) {
        // 每10s执行1次
        Thread thread1 = new Thread(() -> {
            while (true) {
                System.out.println("...");
                try {
                    Thread.sleep(10 * 1000);
                } catch (InterruptedException e) {
                    // ...
                }
            }
        });
        thread1.start();
        
        // 10s后执行1次
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        executorService.submit(() -> {
            try {
                Thread.sleep(10 * 1000);
                // do sth
            } catch (InterruptedException e) {
                // ...
            }
        })
    }
}
```

### 2、Spring的@Scheduled注解
**优点**：简洁、方便，配合cron表达式即可使用
**使用场景**：需要指定某个具体时间点的场景、每个一段时间触发一次的场景等
```java
public class TaskClass {
    @Scheduled("0/10 * * * * ?")
    public void execute() {
        // do sth
    }
}
```

### 3、使用Timer类
```java
public class TaskClass {
    public static void main(String[] args) {
        int count = 0;
        Timer timer = new Timer();
        TimerTask task = new TimerTask() {
            @Override
            public void run() {
                // do sth
                count++;
                if (count > 100) {
                    // 关闭定时器
                    timer.cancel();
                    return;
                }
            }
        };
        // 延迟0s执行，10秒执行间隔
        timer.schedule(task, 0, 10*1000);
    }
}
```

### 4、使用ScheduleExecutorService
```java
public class TaskClass {
    public static void main(String[] args) {
        ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
        executorService.schedule(() -> {
            // do sth
        }, 10, TimeUnit.SECONDS);
    }
}
```

### 5、使用Quartz
// TODO

### 6、使用xxl-job
// TODO

## 三、Q&A
### 1、为什么定时任务可以定时执行？
// TODO

### 2、如何自己实现一个定时任务？
// TODO