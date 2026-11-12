---
type: Note
domain: java
title: String vs StringBuilder vs StringBuffer
aliases: [String StringBuilder StringBuffer, 字符串拼接性能]
description: String 不可变循环拼接 O(n²)；StringBuilder 可变最快（首选）；StringBuffer 同步线程安全略慢
tags: [java基础, string, stringbuilder, 性能, 面试]
status: learning
created: 2026-08-11
updated: 2026-08-11
---

# 概述

三者都用于处理字符串，区别在**可变性**与**线程安全**。String 不可变，循环里 `+=` 每次新建对象导致 O(n²) 灾难；StringBuilder/StringBuffer 可变，内部用可扩容数组，append 不新建对象。循环拼接首选 StringBuilder。

# 详细内容

- **对比表**：
  - String：不可变、不可变故线程安全、循环拼接最慢、JDK1.0。
  - StringBuilder：可变、非线程安全、**最快**、JDK1.5。
  - StringBuffer：可变、方法加 `synchronized` 线程安全、略慢于 Builder、JDK1.0。
- **String += 为何慢**：String 不可变，`+=` 实为造全新 String。编译器把 `s += "a"` 翻译成 `s = new StringBuilder().append(s).append("a").toString()`（`javap -c` 实证），循环 N 次 = new N 个 StringBuilder + N 个临时 String，且每次拷贝越来越长的内容，呈 O(n²)。
- **StringBuilder 为何快**：内部维护可扩容 char 数组，append 只是往末尾加字符，不新建对象。
- **选型决策**：
  - 循环反复拼接、单线程（绝大多数）→ **StringBuilder 首选**；
  - 多线程共享同一拼接对象 → StringBuffer（实际极罕见）；
  - 不拼接只读 → String。
- **`+` 该不该用**：少量一次性 `"a"+b+"c"` 直接用 `+`（编译器自动优化成一个 StringBuilder，可读性好）；循环内禁止 `+`/`+=`。
- **诚实点**：小样本下 StringBuilder/StringBuffer 差距可能被 JVM 噪声掩盖（单线程锁无竞争开销≈0），原理上 StringBuilder ≥ StringBuffer（快 10%~20%）。
- 加分：可 `new StringBuilder(1024)` 预指定容量避免扩容拷贝；大字符串拼接小优化。
- **面试标准答法**：String 不可变，循环 += 每次新建对象性能差；编译器虽优化成 StringBuilder 但是每次循环 new 一个，仍 O(n²)。StringBuilder/StringBuffer 可变、用可扩容数组，append 不新建。区别是 StringBuffer 加 synchronized 线程安全略慢；实际开发优先 StringBuilder。

# 相关链接

- [String 不可变性与字符串常量池](./string-immutability-pool.md)
- [2026-08-11 学习日志](../../daily/2026-08-11.md)
