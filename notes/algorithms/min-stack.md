---
type: Note
domain: algorithms
title: 最小栈：辅助栈保存前缀最小值
aliases: [最小栈, MinStack, min stack]
description: 第 155 题用等长辅助栈保存各层最小值，避免弹栈后单个 cur_min 状态过期
tags: [leetcode, stack, auxiliary-stack]
status: learning
created: 2026-09-21
updated: 2026-09-21
---

# 概述

- 补记 2026-09-20 学习：用数据栈 data 和等长辅助栈 mini 实现最小值查询，最终代码经审阅正确。

# 详细内容

- 不变量：mini[i] 保存 data[0:i+1] 的最小值，而不是只保存当前全局最小值一次。
- push(value)：data 入栈；mini 非空时追加 `min(mini[-1],value)`，否则追加 value。
- pop()：两个栈同步弹出；getMin() 返回 mini[-1]；top() 返回 data[-1]。
- 重复最小值也要重复保存，以维持两个栈一一对应。
- 原错误：维护 self.cur_min，却在 pop 时不恢复；例如 push(2)、push(1)、pop()、push(3)，过期的 1 会污染新状态，实际最小值应为 2。
- 修复思路：不再维护冗余 cur_min，每次从辅助栈当前栈顶推导新的最小值。
- 题目保证 pop、top、getMin 在非空时调用；如果设计通用容器，需要另行明确空栈行为。
- 抽象栈模型每个操作 O(1)；Python list 的追加/尾部弹出按均摊 O(1) 理解，查询 O(1)，空间 O(n)。

# 相关链接

- [2026-09-21 学习日志（含前次补记）](../../daily/2026-09-21.md)
