---
type: interview
scope: topic
area: python
topics:
  - asyncio
  - concurrency
  - event-loop
created: 2026-08-27
updated: 2026-08-27
---

# asyncio 与并发

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。
- 关联知识与题集：
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## async/await 的原理是什么？

<!-- canonical_key: python.asyncio.async-await-de-yuan-li-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Python 协程和线程有什么区别？

<!-- canonical_key: python.asyncio.python-xie-cheng-he-xian-cheng-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## event loop 是什么？

<!-- canonical_key: python.asyncio.event-loop-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 什么情况下 async 反而会变慢？

<!-- canonical_key: python.asyncio.shen-me-qing-kuang-xia-async-fan-er-hui-bian-man -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果 async 函数里面执行 CPU 密集任务会怎样？

<!-- canonical_key: python.asyncio.ru-guo-async-han-shu-li-mian-zhi-xing-cpu-mi-ji-ren-wu-hui-zen-yang -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果调用同步数据库驱动会怎样？

<!-- canonical_key: python.asyncio.ru-guo-diao-yong-tong-bu-shu-ju-ku-qu-dong-hui-zen-yang -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## asyncio.Queue 是怎么工作的？

<!-- canonical_key: python.asyncio.asyncio-queue-shi-zen-me-gong-zuo-de -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## asyncio.create_task 有什么作用？

<!-- canonical_key: python.asyncio.asyncio-create-task-you-shen-me-zuo-yong -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Task 抛异常没人 await 会怎样？

<!-- canonical_key: python.asyncio.task-pao-yi-chang-mei-ren-await-hui-zen-yang -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Python GIL 是什么？

<!-- canonical_key: python.asyncio.python-gil-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## GIL 为什么不会影响 asyncio I/O 并发？

<!-- canonical_key: python.asyncio.gil-wei-shen-me-bu-hui-ying-xiang-asyncio-i-o-bing-fa -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 多进程和多线程怎么选？

<!-- canonical_key: python.asyncio.duo-jin-cheng-he-duo-xian-cheng-zen-me-xuan -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## asyncio.Event、Queue、Future 分别适合什么场景？

<!-- canonical_key: python.asyncio.asyncio-event-queue-future-fen-bie-shi-he-shen-me-chang-jing -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果一个挂起的协程需要跨进程恢复，该怎么做？

<!-- canonical_key: python.asyncio.ru-guo-yi-ge-gua-qi-de-xie-cheng-xu-yao-kua-jin-cheng-hui-fu-gai-zen-me-zuo -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-
