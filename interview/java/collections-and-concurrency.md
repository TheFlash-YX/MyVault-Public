---
type: interview
scope: topic
area: java
topics:
  - collections
  - concurrency
  - thread-pool
created: 2026-08-27
updated: 2026-09-08
---

# Java 集合与并发

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。

## Java 中 HashMap 的底层结构是什么？

<!-- canonical_key: java.collections-concurrency.java-zhong-hashmap-de-di-ceng-jie-gou-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## ConcurrentHashMap 怎么保证线程安全？

<!-- canonical_key: java.collections-concurrency.concurrenthashmap-zen-me-bao-zheng-xian-cheng-an-quan -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## synchronized 和 ReentrantLock 有什么区别？

<!-- canonical_key: java.collections-concurrency.synchronized-he-reentrantlock-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## volatile 能保证原子性吗？

<!-- canonical_key: java.collections-concurrency.volatile-neng-bao-zheng-yuan-zi-xing-ma -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Java 线程池的核心参数有哪些？
<!-- canonical_key: java.collections-concurrency.java-xian-cheng-chi-de-he-xin-can-shu-you-na-xie -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- 本轮重点是 corePoolSize、maximumPoolSize、workQueue 和拒绝策略：先建核心线程，再排队，队列满后扩至最大，再拒绝。
- 核心 2、最大 4、队列 2 且任务均未完成：1/2 运行，3/4 排队，5/6 扩容，7 拒绝。
- 默认 AbortPolicy 抛异常；CallerRunsPolicy 未关闭时由提交者执行形成背压；其他策略可能丢任务。
- keepAliveTime、unit、threadFactory 的详细配置尚未复习，不将此题视为完整掌握。

### 易追问

- 队列已满还能无限排队吗？CallerRuns 对请求延迟有什么影响？

### 关联笔记

- [线程与线程池](../../notes/java/thread-tasks-and-pools.md)

## 为什么不建议直接用 Executors 创建线程池？
<!-- canonical_key: java.collections-concurrency.wei-shen-me-bu-jian-yi-zhi-jie-yong-executors-chuang-jian-xian-cheng-chi -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- 关键是明确工厂方法的线程、队列和拒绝行为，并非所有 Executors 都禁止。
- 本轮学到：无界队列持续接收任务时，最大线程数通常不能推动核心数以上扩容；任务长期积压会增加内存和等待时间。
- 需按业务选择容量与失败处理；其他工厂的具体行为留待后续复习。

### 易追问

- 线程数受控为什么仍可能内存耗尽？

### 关联笔记

- [线程与线程池](../../notes/java/thread-tasks-and-pools.md)

## CompletableFuture 用过吗？

<!-- canonical_key: java.collections-concurrency.completablefuture-yong-guo-ma -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果让 Java 服务调用模型 API，你如何设计并发控制？

<!-- canonical_key: java.collections-concurrency.ru-guo-rang-java-fu-wu-diao-yong-mo-xing-api-ni-ru-he-she-ji-bing-fa-kong-zhi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Java 如何创建线程并取得任务结果？run 与 start 有什么区别？
<!-- canonical_key: java.collections-concurrency.thread-task-future -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- 继承 Thread 或把 Runnable 交给 Thread；run 是当前线程的普通调用，start 才启动新线程。
- 需要返回值可使用 Callable，经 FutureTask 包装后交给 Thread，也可提交到线程池取得 Future。
- get 未完成时阻塞等待；任务失败和等待中断需要处理，任务接口本身不创建线程。

### 易追问

- 大量 new Thread 有何风险？Runnable 能改成 int run() 吗？

### 关联笔记

- [线程、任务与线程池](../../notes/java/thread-tasks-and-pools.md)
