---
type: Pitfall
domain: algorithms
title: 示例全过、提交就挂：链尾边界防御缺失
description: 链表题三次同型翻车——防御缺失只在「目标段延伸到链尾」时触发，LC 官方示例恰好不覆盖
tags: [leetcode, linked-list, edge-case, testing]
created: 2026-08-28
updated: 2026-08-28
---

# 现象

- 代码在 LeetCode 官方示例上全过，一提交就挂（AttributeError 或答案错误），挂的用例都带「目标段在链尾」特征
- 三次实录：143 `reverseList` 里 `while cur:` 忘写 `cur = head`（后半段 1 节点时提前返回救场、≥2 节点才炸）；206 递归出口 `if not head or head.next` 漏第二个 `not`（多节点原样返回、单节点 None 炸）；82 内层 `while cur.val == cur.next.val` 缺 `cur.next and` 守卫（`[1,1]`、`[1,2,2]` 必炸）

# 排查过程

- 手跑「重复段延伸到链尾」的最小用例（`[1,1]`、`[1,2,2]`）立刻复现 `None.val`
- 共性发现：LC 官方示例往往「温和」——82 两示例每段重复后都跟着非重复节点，恰好绕过链尾场景，给虚假信心

# 根因

- 循环/递归的防御条件只覆盖「段在中间」的路径；**段延伸到链尾时指针走到 None，裸解引用直接炸**
- 示例驱动开发（只跑官方示例）不等于边界覆盖：官方示例不是边界全集

# 解决方案

- 自检一句：**「重复段/目标段在链尾时，我的代码还活着吗？」**——写完专门构造段在链尾的最小用例
- 内层 while 一律带 None 守卫：`while cur.next and cur.val == cur.next.val`
- 结构优先：让 None 安全内建于循环条件（83 / 灵神 82 的「cur 不动、每轮重验」结构），少靠外挂守卫
- 固定 5-case 验收：普通 / 段在链首 / 段在链尾 / 单节点 / 无重复

# 相关链接

- [链表删除家族：83 留一个与 82 整段删](../notes/algorithms/linked-list-deletion.md)
- [反转链表家族：206 整链、92 区间与 25 分组翻转](../notes/algorithms/linked-list-reversal.md)
- [2026-08-28 学习日志](../daily/2026-08-28.md)
