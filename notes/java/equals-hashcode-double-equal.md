---
type: Note
domain: java
title: equals、hashCode 与 ==（Object 核心三件套）
aliases: [equals, hashCode, equals与==区别, equals和hashCode, Object三件套]
description: 对象身份与逻辑相等、equals/hashCode 契约、引用赋值及可变键风险
tags: [java基础, equals, hashCode, Object, 哈希冲突, 面试]
status: learning
created: 2026-08-26
updated: 2026-09-08
---

# 概述

- == 对基本类型比较值，对引用比较对象身份；equals 的规则取决于类的实现，不能一概说“比内容”。
- Object.equals 默认等价于 this == obj；String 重写为内容比较。
- equals 相等要求 hashCode 相同；反向不成立。重写 equals 时通常同时重写 hashCode。
- 2026-09-08 面试复习：本轮已纠正引用复制、哈希冲突和可变键的误区，尚不表示完成代码实践。

# 详细内容

## 引用与对象

- 两次 new String("hello") 得到不同对象：== false，equals true。
- a、b 是属性相同的独立 Student 对象，类未重写 equals：== false，equals false。
- c=a 后，两者引用同一对象：a==c 和 a.equals(c) 都是 true（a 非 null）。
- b=a 只复制引用，不复制对象；b.setName("小王") 改变共享对象，通过 a 也看到“小王”。
- b=new Student("小王") 只改变 b 的指向，原 a 对象不变。

## equals 与 hashCode

- equals 应满足自反、对称、传递、一致，以及非空对象不等于 null。
- 实现可按 this==o、null/类型检查、字段比较组织；Objects.equals 可处理可空字段。
- getClass 与 instanceof 的选择需与继承和相等性设计一致，不能断言 instanceof 必然破坏对称性。
- String 的内容比较用 equals；对象身份比较才用 ==。
- hashCode 不是内存地址，默认实现也不保证不同对象一定不同。
- HashMap 根据哈希定位，再结合哈希及相等性判断键；平均查找效率依赖分布，不能笼统保证任何情况 O(1)。
- 相等对象必须同哈希；不同对象可同哈希，不同哈希也可能落同桶；仅落同桶不够判重。
- "FB" 与 "Ea" 的 String.hashCode 相同，但 equals 为 false，是哈希冲突。
- 只重写 equals 可能让逻辑相等的对象产生不同哈希，HashSet 无法保证去重。
- 推荐以决定相等性的稳定字段生成哈希，例如按学号相等时使用 Objects.hash(studentId)；契约要求相等必同哈希，并非语法上强制字段集合完全一致。
- String 的哈希按 31 倍累加，内容相同则哈希相同；不要求背成 JVM 总会进行某种位移优化。

## 可变键风险

- HashSet 元素/HashMap 键放入容器后，不应修改参与 equals/hashCode 的字段；容器不会因此自动重新定位。
- 学号从 001 改为 002 后，contains(s) 即便传入同一对象也不能保证成功；旧节点保留旧哈希信息。
- 必须修改时，先 remove，再修改，再 add；否则移除也可能找不到。
- 姓名不参与 equals/hashCode 时，仅修改姓名不会造成这个定位问题。
- 优先使用不可变键或稳定标识；final 是实现手段之一，引用 final 不等于其指向对象深度不可变。

## 关联易错点

- Integer 的 == 还涉及缓存和拆箱；详见关联笔记，不能一概理解为内容比较。
- 面试精简表达放到语言基础题集，避免继续维护两份完整回答。

# 相关链接

- [Java 语言基础题集](../../interview/java/language-basics.md)

- [String 不可变性与字符串常量池](./string-immutability-pool.md)
- [自动装箱拆箱与 Integer 缓存陷阱](./boxing-unboxing-integer-cache.md)
- [2026-08-26 学习日志](../../daily/2026-08-26.md)
