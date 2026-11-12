---
type: Note
domain: java
title: 自动装箱拆箱与 Integer 缓存陷阱
aliases: [装箱 拆箱, Integer 缓存, boxing unboxing]
description: 装箱=valueOf、拆箱=intValue；Integer 缓存 -128~127 导致 == 在 127 为 true、128 为 false
tags: [java基础, integer, 缓存, equals, 面试]
status: learning
created: 2026-08-10
updated: 2026-08-10
---

# 概述

装箱/拆箱是基本类型与包装类之间的自动转换：装箱由编译器自动调用 `valueOf()`，拆箱调用 `intValue()`。`Integer.valueOf()` 默认缓存 **-128~127** 的对象，由此引出经典的 `==` 陷阱：127 相等、128 不相等。

# 详细内容

- **装箱**（值→对象）：`Integer a = 127;` 编译器改为 `Integer.valueOf(127)`（`javap -c` 可见 `invokestatic Integer.valueOf`）。
- **拆箱**（对象→值）：`int c = a;` 编译器改为 `a.intValue()`。
- **缓存池（IntegerCache）**：`valueOf()` 对 -128~127 返回池中同一对象；超出范围每次 `new` 新对象。
- **`==` 比对象内存地址，`equals` 比值**——贯穿 Integer 与 String 的核心主线。
- 实验结论（已跑代码验证）：
  - `Integer(127) == Integer(127)` → **true**（缓存内同一对象）
  - `Integer(128) == Integer(128)` → **false**（超缓存，两个对象）
  - `Integer(128).equals(Integer(128))` → true
  - `Integer(128) == int 128` → **true**（一边是基本类型则触发拆箱，变成比值）
  - `new Integer(127) == Integer(127)` → false（`new` 绕过缓存强制新建）
- `new Integer()` 在 **Java 9+ 已废弃**（`@Deprecated`，编译警告"已过时且标记为待删除 removal"）；所有包装类构造函数同命运；一律用自动装箱/`valueOf`。
- 加分项：缓存上限可用 `-XX:AutoBoxCacheMax` 调大；`Long`/`Short`/`Byte`/`Character` 有同范围缓存；`Double`/`Float` **无缓存**；循环中频繁装箱拆箱有性能开销，热点代码用基本类型。
- **面试标准答法**：装箱调 valueOf、拆箱调 intValue；valueOf 缓存 -128~127，范围内 == 为 true、范围外 false；包装类比较必须用 equals，除非一边是基本类型触发拆箱。

# 相关链接

- [Java 8 种基本数据类型与三大必考坑](./primitive-data-types.md)
- [String 不可变性与字符串常量池](./string-immutability-pool.md)
- [2026-08-10 学习日志](../../daily/2026-08-10.md)
