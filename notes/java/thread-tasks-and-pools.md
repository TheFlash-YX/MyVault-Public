---
type: Note
domain: java
title: Java 线程、任务与线程池
aliases: []
description: 从 run/start 到 Callable/Future，再理解有界队列、扩容顺序与拒绝策略
tags: [java, concurrency, thread-pool]
status: learning
created: 2026-09-08
updated: 2026-09-08
---

# 概述

- 从 run/start 到 Callable/Future，再理解有界队列、扩容顺序与拒绝策略
- 来源：2026-09-07 龙旗科技一面后的 2026-09-08 互动复习；记录已学内容，不表示已完成代码实践或熟练掌握。

# 详细内容

## 线程与任务

- 直接调用 run() 是普通方法调用，在当前线程执行；start() 启动新线程执行任务。
- 继承 Thread 重写 run，或实现 Runnable 再交给 Thread；Lambda 可以描述 Runnable。
- Runnable.run() 返回 void，不能改成 int run() 来实现同一个接口方法。
- Callable<V>.call() 可以返回 V；Callable 本身不会启动线程，可通过 FutureTask 包装后交给 Thread。
- Future.get() 未完成时阻塞调用线程；任务失败可能抛 ExecutionException，等待也可能被中断。

```java
// 方法内部示例，外层需处理 InterruptedException、ExecutionException。
Callable<Integer> task = () -> 1 + 2;
FutureTask<Integer> future = new FutureTask<>(task);
new Thread(future).start();
Integer result = future.get(); // 3
```

## ThreadPoolExecutor 接收顺序

- 小于 corePoolSize：创建工作线程；达到核心数：优先入队。
- 入队失败且未达到 maximumPoolSize：创建额外线程；不能接收任务时触发拒绝策略。
- 假设核心数 2、最大数 4、队列容量 2，期间没有任务结束：任务 1/2 创建线程，3/4 排队，5/6 创建额外线程，7 被拒绝。
- 上述示例假设线程创建正常、线程池运行中；关闭也会导致拒绝。
- 无界队列下通常不会触发超过核心数的扩容，maximumPoolSize 不能替代队列容量控制。

## 过载与拒绝策略

- 大量创建线程消耗线程栈等资源，增加上下文切换成本，可能耗尽资源。
- 任务长期生产快于消费，会积压、增加等待时间，可能导致超时和内存溢出。
- AbortPolicy：默认策略，抛 RejectedExecutionException。
- CallerRunsPolicy：线程池未关闭时由提交线程执行，降低后续提交速度形成背压；会占用请求线程。关闭时丢弃任务。
- DiscardPolicy：丢弃任务；DiscardOldestPolicy：未关闭时丢弃队列头部任务再重试，关闭时丢弃。
- 队列已满不是再放一个“异步队列”就解决；需要明确容量、失败反馈和业务可靠性要求。
- 线程池复用线程，但仍需要管理 shutdown、任务失败和资源上限。

## 本轮边界

- 已理解核心数、最大数、队列和拒绝处理；keepAliveTime、TimeUnit、ThreadFactory 的参数细节留待后续复习。
- Executors 不是一律禁止；需了解具体工厂的队列和线程上限，不能把最大线程数当作无界队列的安全保障。

# 相关链接

- [Java 集合与并发题集](../../interview/java/collections-and-concurrency.md)
- [ThreadPoolExecutor 官方文档](https://docs.oracle.com/en/java/javase/21/docs/api/java/util/concurrent/ThreadPoolExecutor.html)
