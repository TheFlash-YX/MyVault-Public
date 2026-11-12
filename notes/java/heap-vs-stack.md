---
type: Note
domain: java
title: JVM 堆（Heap）与栈（Stack）
aliases: [堆 栈, Heap Stack, 内存区域]
description: 堆存所有 new 的对象（共享、OutOfMemoryError）；栈存方法调用栈帧与局部变量（每线程一条、StackOverflowError）
tags: [java基础, jvm, 内存, oom, 面试]
status: learning
created: 2026-08-10
updated: 2026-08-10
---

# 概述

JVM 运行时内存两大核心区域：**堆**是所有 `new` 出来的对象本体所在（全线程共享），**栈**是方法调用链与局部变量所在（每个线程独立一条）。`==` 比较的本质就是比较栈中存放的堆地址。

# 详细内容

- **堆（Heap）**：
  - 存所有 `new` 的对象（Integer、String、数组、自定义对象……）；
  - 全线程共享，容量大（可用 `-Xmx` 调上限）；
  - 塞满抛 `OutOfMemoryError`（实验：`-Xmx10m` 时 new 第 7 个 1MB 对象即爆）。
- **栈（Stack）**：
  - 每调用一个方法压入一层**栈帧**（装该方法的局部变量），返回即弹出；
  - 每个线程一条，容量小；
  - 爆栈抛 `StackOverflowError`（实验：无终止递归压了 **47387 层**即爆——SOF 九成是递归没写终止条件）。
- **引用 = 遥控器**：对象本体在堆里，栈上的变量只存它的内存地址；`==` 比的就是栈里的地址，`equals` 才比对象内容。
- **基本类型 vs 对象**：`int x = 127` 值直接存栈上；`Integer n = ...` 引用在栈、本体在堆。
- **字符串常量池自 JDK 7 起位于堆中**（从方法区迁入），是堆里的共享专区。
- OOM 真实特征：堆满时连打印一行日志所需的字符串都分配不出（演示中 catch 块须先 `list = null` 释放引用 + `System.gc()` 才能打印）。
- 排查方向：OOM 用堆转储（heap dump）分析占内存大户；SOF 查递归终止条件。
- 铺垫：完整内存区域见 JVM 章节《Java 内存区域》（堆、栈、方法区/元空间、程序计数器等）。

# 相关链接

- [String 不可变性与字符串常量池](./string-immutability-pool.md)
- [JDK、JRE、JVM 的关系与 Java 跨平台原理](./jdk-jre-jvm.md)
- [2026-08-10 学习日志](../../daily/2026-08-10.md)
