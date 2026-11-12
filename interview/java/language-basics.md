---
type: interview
scope: topic
area: java
topics: [object-equality, exceptions, annotations]
created: 2026-09-08
updated: 2026-09-08
---

# Java 语言基础

- 来源：2026-09-08 面试复习；仅收录本轮已讨论问题。

## ==、equals 和 hashCode 有什么关系？
<!-- canonical_key: java.language.object-equality-contract -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- == 对基本类型比值、对引用比身份；Object.equals 默认比较身份，String 等类可重写为逻辑相等。
- equals 相等必须 hashCode 相同，反过来不成立；自定义相等性时通常配套重写哈希。
- 集合中的键要保持相等性字段稳定，否则可能无法查找；修改不参与比较的姓名没有这个问题。

### 易追问

- 引用复制与对象复制有什么区别？哈希相同是否等于同一对象？

### 关联笔记

- [对象相等性笔记](../../notes/java/equals-hashcode-double-equal.md)

## try 中 return 后 finally 是否执行？异常如何传播？
<!-- canonical_key: java.language.finally-return-exception -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- 通常先计算返回表达式，再执行 finally，最后返回；修改局部 int 不改变已计算的值。
- finally 正常结束则继续原返回/异常；finally return 会覆盖返回或异常，应避免。
- 调用方未捕获异常时跳过后续顺序语句；捕获且不再抛出可继续执行。JVM 终止等不保证 finally 执行。

### 易追问

- finally return 300 与 try throw 同时存在，调用方能否同时得到异常和值？

### 关联笔记

- [finally 与异常传播](../../notes/java/try-catch-finally-return.md)

## 如何自定义注解？加注解就会自动执行逻辑吗？
<!-- canonical_key: java.language.custom-annotation -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- 使用 @interface 定义，@Target 指定使用位置，@Retention 指定保留阶段，属性可以有默认值。
- 运行时读取用 RUNTIME；注解只是元数据，需要反射处理代码或框架机制赋予行为。
- 自定义 @AuditLog 不会自动记录日志，需配套处理实现。

### 易追问

- 运行时反射为什么需要保留注解？

### 关联笔记

- [注解基础](../../notes/java/annotations-basics.md)
