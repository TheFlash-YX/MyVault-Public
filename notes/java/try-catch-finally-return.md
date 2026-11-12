---
type: Note
domain: java
title: try-catch-finally：返回值与异常传播
aliases: []
description: 区分待返回值、finally 正常结束与覆盖异常，避免把清理完成当作异常已处理
tags: [java, exceptions, finally]
status: learning
created: 2026-09-08
updated: 2026-09-08
---

# 概述

- 区分待返回值、finally 正常结束与覆盖异常，避免把清理完成当作异常已处理
- 来源：2026-09-07 龙旗科技一面后的 2026-09-08 互动复习；记录已学内容，不表示已完成代码实践或熟练掌握。

# 详细内容

## 执行规则

- try 运行正常代码，catch 捕获匹配异常，finally 执行清理。
- try 中 return 先计算返回表达式，再执行 finally，最后才真正返回。
- finally 正常结束：恢复原本的返回或异常传播；finally 中 return 或抛出新异常：覆盖原本待完成的返回或异常。
- 一次方法调用在调用方表现为正常返回或抛异常，不会先返回 100 再返回 300，也不会同时交付值和异常。
- JVM 被终止等情况下不保证执行 finally；不要背成任何情况下都必然执行。

## 基本类型返回值

```java
static int test() {
    int i = 100;
    try {
        return i;
    } catch (Exception e) {
        return 200;
    } finally {
        i = 300;
    }
}
```

- 返回 100，修改局部变量不改变已计算的基本类型返回值。
- 若 finally 改成 return 300，则只返回 300；不要在 finally 中 return，以免隐藏异常。
- 该例讨论基本类型；不要把“暂存返回值”理解为对对象做深拷贝。

## 异常传播练习

```java
static int test() {
    try {
        throw new RuntimeException("出错了");
    } catch (Exception e) {
        throw e;
    } finally {
        System.out.println("清理完成");
    }
}
```

- 调用方若直接执行 int result = test()，赋值不会完成，同一顺序路径后面的打印不执行，异常继续向外传播。
- 若调用方用 catch 捕获且不再抛出，则先打印“清理完成”，再执行调用方 catch，随后可以继续执行 catch 后的语句。
- 本轮误区：finally 打印不等于捕获异常；catch 才是捕获位置，finally 块内与 finally 后的语句要分清。

# 相关链接

- [Java 语言基础面试题](../../interview/java/language-basics.md)
