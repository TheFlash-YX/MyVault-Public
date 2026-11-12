---
type: Note
domain: java
title: Java 8 种基本数据类型与三大必考坑
aliases: [基本数据类型, primitive types]
description: byte/short/int/long/float/double/char/boolean 的字节数与范围；浮点精度、整数溢出、字面量后缀三大坑
tags: [java基础, 数据类型, bigdecimal, 面试]
status: learning
created: 2026-08-10
updated: 2026-08-10
---

# 概述

Java 有 **8 种基本数据类型**（primitive types）：4 种整型、2 种浮点、字符、布尔。它们存的是值本身而非对象。面试三大坑：浮点精度丢失、整数溢出、字面量后缀。

# 详细内容

- **整型四兄弟（字节数 1/2/4/8 翻倍）**：
  - `byte`（1 字节，-128~127）；`short`（2 字节，很少用）；`int`（4 字节，±21 亿，⭐默认整型最常用）；`long`（8 字节，字面量必须加 `L`）。
- **浮点**：`float`（4 字节，字面量加 `f`）；`double`（8 字节，小数默认类型）。
- **字符**：`char`（2 字节，Unicode，**可存中文**，单引号 `'中'`）。
- **布尔**：`boolean`（只有 `true`/`false`，**不能用 0/1**——与 C 不同；大小与 JVM 实现相关）。
- **坑 1：浮点精度**——`0.1 + 0.2 = 0.30000000000000004`，`(0.1+0.2) == 0.3` 为 `false`。原因：0.1、0.2 在二进制中是无限循环小数，只能近似存储。**金额等精确计算必须用 `BigDecimal`，禁用 float/double**。
- **坑 2：整数溢出**——`Integer.MAX_VALUE + 1 = -2147483648`（最小值），溢出**不报错不抛异常**，悄悄绕回。大数场景（毫秒时间戳、以分为单位的金额）用 `long`。
- **坑 3：字面量后缀**——`long a = 9223372036854775807L`（不加 L 按 int 解析会编译报错）；`float f = 3.14f`（不加 f 默认 double 塞不进 float）。
- 基本类型不是对象、无方法可调；每个基本类型有对应包装类（`int→Integer`、`long→Long`、`char→Character`……）。
- 成员变量默认值：数值 0 / 0.0、`char` 为 `'\u0000'`、`boolean` 为 `false`。

# 相关链接

- [自动装箱拆箱与 Integer 缓存陷阱](./boxing-unboxing-integer-cache.md)
- [JVM 堆与栈](./heap-vs-stack.md)
- [2026-08-10 学习日志](../../daily/2026-08-10.md)
