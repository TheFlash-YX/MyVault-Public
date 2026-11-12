---
type: interview
scope: topic
area: backend
topics:
  - kafka
  - messaging
  - async-tasks
created: 2026-08-27
updated: 2026-08-27
---

# Kafka 与消息队列

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。
- 关联知识与题集：
- [不引入 Redis/MQ 的架构取舍](../../notes/backend/architecture/cache-and-async-task-alternatives.md)

## Kafka 为什么吞吐量高？

<!-- canonical_key: backend.kafka.kafka-wei-shen-me-tun-tu-liang-gao -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Kafka 中 Topic、Partition、Consumer Group 分别是什么？

<!-- canonical_key: backend.kafka.kafka-zhong-topic-partition-consumer-group-fen-bie-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何保证消息不丢？

<!-- canonical_key: backend.kafka.ru-he-bao-zheng-xiao-xi-bu-diu -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何保证消息不重复消费？

<!-- canonical_key: backend.kafka.ru-he-bao-zheng-xiao-xi-bu-zhong-fu-xiao-fei -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Kafka 能保证 exactly once 吗？

<!-- canonical_key: backend.kafka.kafka-neng-bao-zheng-exactly-once-ma -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Agent 异步任务场景为什么可能需要消息队列？

<!-- canonical_key: backend.kafka.agent-yi-bu-ren-wu-chang-jing-wei-shen-me-ke-neng-xu-yao-xiao-xi-dui-lie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果做长耗时 Agent Task，你会怎么设计？

<!-- canonical_key: backend.kafka.ru-guo-zuo-zhang-hao-shi-agent-task-ni-hui-zen-me-she-ji -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## ask_user 的恢复通知更适合 Pub/Sub、Stream 还是普通 KV？为什么？

<!-- canonical_key: backend.kafka.ask-user-de-hui-fu-tong-zhi-geng-shi-he-pub-sub-stream-hai-shi-pu-tong-kv-wei-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-
