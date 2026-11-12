---
type: Note
domain: java
title: Java 是编译型还是解释型语言（混合模式与 JIT）
aliases: [编译型 解释型, JIT, 字节码]
description: Java 先编译后解释的混合执行模式：javac 编译为字节码，JVM 解释执行 + JIT 编译热点代码
tags: [java基础, jvm, jit, 字节码, 面试]
status: learning
created: 2026-08-10
updated: 2026-08-10
---

# 概述

Java **既是编译型也是解释型**，属于"先编译、后解释"的混合模式：先由 `javac` 把源码编译成平台无关的字节码，再由 JVM 解释执行。现代 JVM（HotSpot）实际是**解释器 + JIT 编译器**的混合执行，因此启动慢、但长期运行性能逼近 C/C++。

# 详细内容

- **编译型语言**（C/C++/Go）：整本一次性翻译成机器码；运行快，但换平台必须重新编译。
- **解释型语言**（Python/JS）：逐句翻译逐句执行（同声传译）；跨平台容易（有解释器即可），但慢。
- **Java 两步走**：
  - 第一步（编译）：`javac Hello.java` → 产出 `Hello.class` 字节码。
  - 第二步（解释）：`java Hello` → JVM 将字节码一条条解释为机器码执行。
- **字节码**是 JVM 专属的指令集（如 `getstatic`、`ldc`、`invokevirtual`），既不是源码也不是 CPU 机器码；用 `javap -c -p 类名` 可反编译查看。
- 字节码是 Java 跨平台的基础：编译只进行到字节码就停下（与平台无关），剩下的翻译交给各平台 JVM。
- **JIT（Just-In-Time，即时编译器）**：JVM 运行时发现被频繁执行的"热点代码"，直接编译成本地机器码缓存，下次直接执行机器码。
- **混合模式的意义**：解释器保证启动快，JIT 保证长期运行快；这是 Java"启动慢、长期性能不输 C"的原因。
- **面试标准答法**：Java 是混合模式——先编译成字节码再由 JVM 解释执行；现代 JVM 为解释器 + JIT 混合，JIT 把热点代码编译成本地机器码提升性能；字节码支撑跨平台。
- 补充：编译型 vs 解释型并非非黑即白，JS（V8 JIT）、Python（.pyc 字节码）也在混合化。

# 相关链接

- [JDK、JRE、JVM 的关系与 Java 跨平台原理](./jdk-jre-jvm.md)
- [2026-08-10 学习日志](../../daily/2026-08-10.md)
