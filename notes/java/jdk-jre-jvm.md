---
type: Note
domain: java
title: JDK、JRE、JVM 的关系与 Java 跨平台原理
aliases: [JDK JRE JVM, Java 跨平台]
description: JDK⊃JRE⊃JVM 的包含关系；"一次编译到处运行"靠字节码 + 各平台 JVM 屏蔽操作系统差异
tags: [java基础, jvm, 跨平台, 面试]
status: learning
created: 2026-08-10
updated: 2026-08-10
---

# 概述

JDK / JRE / JVM 是 Java 体系的三个核心概念，呈包含关系：**JDK ⊃ JRE ⊃ JVM**。Java 的跨平台能力（一次编译，到处运行）本质上是 JVM 屏蔽了底层操作系统的差异。面试 warm-up 高频题。

# 详细内容

- **JVM（Java Virtual Machine）**：真正执行程序的虚拟计算机，负责把字节码解释/编译成机器码执行。
- **JRE（Java Runtime Environment）** = JVM + 核心类库（如 String、System、集合类等）；只想"运行"Java 程序装 JRE 就够。
- **JDK（Java Development Kit）** = JRE + 开发工具（`javac` 编译器、`jar`、`javadoc`、`javap` 等）；要"开发"Java 程序必须装 JDK。
- **包含关系**：JDK ⊃ JRE ⊃ JVM（JDK 最大，JVM 最核心）。
- **跨平台原理**：`.java` 源码经 `javac` 编译为 `.class` 字节码（平台无关的中间产物），再交给各平台对应的 JVM（Windows/Mac/Linux 各自的 JVM）转成机器码执行。
- "一次编译，到处运行"的载体是字节码，跨平台的重担由 JVM 承担。
- **面试标准答法**：JVM 执行字节码；JRE = JVM + 核心类库；JDK = JRE + 开发工具；三者是 JDK 包含 JRE、JRE 包含 JVM 的关系；跨平台依赖 JVM——源码编译成平台无关的字节码，由各平台 JVM 转机器码执行。
- 本机环境：OpenJDK 21.0.12（`brew install openjdk@21`），keg-only 需在 `~/.zshrc` 配置 `JAVA_HOME=/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home` 并加入 PATH。

# 相关链接

- [Java 是编译型还是解释型语言](./java-compile-interpret.md)
- [JVM 堆与栈](./heap-vs-stack.md)
- [2026-08-10 学习日志](../../daily/2026-08-10.md)
