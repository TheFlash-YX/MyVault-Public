---
type: Note
domain: java
title: Java 自定义注解与运行时处理
aliases: []
description: 注解是元数据，使用 @interface 定义，通过处理代码赋予作用
tags: [java, annotations, reflection]
status: learning
created: 2026-09-08
updated: 2026-09-08
---

# 概述

- 注解是元数据，使用 @interface 定义，通过处理代码赋予作用
- 来源：2026-09-07 龙旗科技一面后的 2026-09-08 互动复习；记录已学内容，不表示已完成代码实践或熟练掌握。

# 详细内容

## 定义与使用

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface AuditLog {
    String value() default "";
}
```

- @interface 声明注解类型，value 是属性，default 提供默认值。
- @Target(METHOD) 限制标在方法上；@Retention(RUNTIME) 让运行时可以通过反射读取。
- 使用形式为 @AuditLog("删除用户")。
- 注解本身不执行日志操作；需要反射处理代码或框架机制读取它。
- Spring AOP 可以作为处理方案，但仅定义这个注解不会自动创建切面；本轮未实现切面代码。
- 本轮误区：看见 @ 标记，不等于自动获得业务行为。

# 相关链接

- [Java 语言基础题集](../../interview/java/language-basics.md)
- [Spring 配置读取](../backend/spring/value-configuration.md)
