---
type: Note
domain: java
title: String 不可变性与字符串常量池
aliases: [String 不可变, 字符串常量池, String Pool, intern]
description: String 修改均返回新对象；字面量共享于常量池；new String("abc") 创建 1~2 个对象
tags: [java基础, string, 常量池, 不可变, 面试]
status: learning
created: 2026-08-10
updated: 2026-08-10
---

# 概述

String 是**不可变类**：任何"修改"方法都返回新对象，原对象不变。字面量进入**字符串常量池**被共享（与 Integer 缓存同一设计思想）。`new String("abc")` 视池中是否已有而创建 **1~2 个对象**——史上最经典 Java 面试题之一。

# 详细内容

- **字符串常量池（String Pool）**：JVM 堆中的共享专区；`String a = "hello"` 先入池，`String b = "hello"` 直接复用同一对象 → `a == b` 为 **true**。
- **`new String("abc")` 创建几个对象？** 答案：**1 个或 2 个**。字面量 `"abc"` 检查池（没有则池中建 1 个），`new` 在堆上再建 1 个：池中没有 → 2 个（池1+堆1），已有 → 1 个（仅堆）。
- **`intern()`**：返回池中的那个对象（池里没有就先放入）；`new String("a").intern() == "a"` 为 true。
- **不可变实证**：`s.concat(" world")` 不接收返回值等于白调用，`s` 不变；必须 `s = s.concat(...)` 接收新对象。`replace`/`substring`/`toUpperCase` 等同理。
- **设计不可变的 4 个原因（面试高分点）**：
  - 常量池共享的前提（内容会变就无法共享）；
  - hashCode 算一次即缓存，HashMap 用 String 当 key 又快又安全；
  - 不可变对象天生线程安全；
  - 传参安全（文件路径、URL、密码等不怕被调用方篡改）。
- 比较字符串内容一律用 `equals`；`==` 比的是栈中引用（地址）。
- 常用对象共享省内存：常量池（String）、IntegerCache（-128~127）是同一思想的两个实例。

# 相关链接

- [JVM 堆与栈](./heap-vs-stack.md)
- [自动装箱拆箱与 Integer 缓存陷阱](./boxing-unboxing-integer-cache.md)
- [2026-08-10 学习日志](../../daily/2026-08-10.md)
