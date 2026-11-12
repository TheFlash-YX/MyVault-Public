---
type: Note
domain: python
title: map() 与内置函数组合的一行流
aliases: [map函数, 迭代器, 一行流]
description: map(f, it) 逐元素作用返回惰性迭代器；max(map(max, f)) 二维表全表最大等一行流写法与惰性陷阱
tags: [python, map, 迭代器, 内置函数]
status: learning
created: 2026-08-17
updated: 2026-08-17
---

# 概述

- `map(函数, 可迭代)` 把函数逐元素作用上去，返回**惰性迭代器**；与 `max`/`sum` 等聚合内置函数组合成一行流

# 详细内容

## map 的本质

- `map(f, xs)` ≈ 惰性的 `[f(x) for x in xs]`——不立即计算，被消费时才逐个产出
- 需要看内容时物化：`list(map(str, [1,2,3]))` → `['1','2','3']`
- 多序列形态：`map(f, xs, ys)` → `f(x, y)` 逐对作用，短的支持截断

## 典型一行流：二维表全表最大

```python
max(map(max, f))   # f 是 list[list[int]]
```

- 内层 `map(max, f)`：把 `max` 逐**行**作用 → 每行的行内最大值（迭代器）
- 外层 `max(...)`：对行最大值再取 max → 全表最大
- 等价朴素写法：
  ```python
  ans = 0
  for row in f:
      ans = max(ans, max(row))
  ```
- 出处：718 最长重复子数组全表扫答案（「以 i 结尾」型 DP 的标准收尾动作）

## 惰性陷阱：迭代器是一次性的

- map 产物被 `max()` 消费一次后就空了，再 `max()` 同一对象会抛 `ValueError: max() arg is an empty sequence`
- 需要**多次使用**结果时先 `list(...)` 物化

## 与列表推导的选择

- 只被聚合一次、不需要中间列表 → `map` + 聚合函数省内存（惰性）
- 需要变换表达式（lambda 逻辑）或立即得到列表 → 列表推导更直观

# 相关链接

- [2026-08-17 学习日志](../../daily/2026-08-17.md)
- [最大子数组问题：以 i 结尾型 DP](../algorithms/dp-max-subarray.md) —— max(map(max, f)) 的使用场景
