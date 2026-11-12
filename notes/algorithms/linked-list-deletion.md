---
type: Note
domain: algorithms
title: 链表删除家族：83 留一个与 82 整段删
aliases: [删除排序链表重复项, 删除重复元素, remove duplicates from sorted list]
description: 83 cur 不动式删除；82 pre 锚点 + cur 侦察整段删、「没重复才推进前驱」；删除题的 None 守卫与 dummy 双口径
tags: [linked-list, deletion, dummy-node, duplicates]
status: learning
created: 2026-08-28
updated: 2026-08-28
---

# 概述

- 删除题通用骨架：**锚点（已验证安全位）+ 侦察指针（探路）**；删除动作永远是「锚点不动、指针改接」
- 83（留一个）与 82（整段删）只差一步：82 连第一个重复节点也不保留 → 头可能被摘 → 必须 dummy

# 详细内容

## 83 删除排序链表重复项（留一个）

- 骨架：`cur` 扫链，`cur.val == cur.next.val` → `cur.next = cur.next.next`（**cur 原地不动**，下轮循环重新校验新邻居）；不等才 `cur = cur.next`
- 「cur 不动」的结构性好处：`while cur and cur.next` 每轮重新验活，None 安全内建于结构，无需外挂守卫
- **不用 dummy 是对的**：每组重复必留第一个 → 头节点永远存活，`return head` 安全——呼应 dummy 口径「头可能被摘才用」的反向印证

## 82 删除排序链表重复项 II（整段删）

- 双指针分工：**pre = 已验证安全段的尾**（锚点，只踩确认保留的节点），**cur = 侦察兵**（探路，可能被整段否决）
- 核心规则：**没重复才推进 pre**——pre 永远干净，`pre.next` 指向的永远是待验证节点
- 灵神版耦合设计：cur 停在最后一个安全节点上，发现重复时 `cur.next = cur.next.next` 逐个摘除（cur 不动），安全才 `cur = cur.next`——「谁在动、谁被查」合一，None 安全内建于结构
- ⚠️ 实测踩坑：内层 `while cur.val == cur.next.val` 缺 `cur.next and` 守卫——重复段延伸到链尾时 `cur.next = None` → `None.val` AttributeError（`[1,1]`、`[1,2,2]` 必炸；LC 官方两示例恰好每段重复后都跟非重复节点所以全过）——「示例全过、提交就挂」型 bug，见 [踩坑记录](../../pitfalls/leetcode-examples-pass-submit-fails.md)
- 修法：`while cur.next and cur.val == cur.next.val`

## dummy 双口径（删除/构造通用）

- ① **头节点可能被摘/换 → 用 dummy**：82（链首重复段整段删）、19（n==len 删头）、92/25/24（换头）
- ② **逐步造新链 → 用 dummy 统一首节点创建**：2 两数相加、21/23 合并——免去「第一个节点 vs 后续节点」特判
- 反例（不用）：指针只读 141/142/160；头必留 83

# 相关链接

- [反转链表家族：206 整链、92 区间与 25 分组翻转](linked-list-reversal.md)
- [链表快慢指针家族：141 判环、142 找入口与找中点](linked-list-fast-slow.md)
- [链表合并家族：21/23 合并、148 归并与 2 竖式相加](linked-list-merge.md)
- [踩坑：示例全过、提交就挂（链尾边界防御缺失）](../../pitfalls/leetcode-examples-pass-submit-fails.md)
- [2026-08-28 学习日志](../../daily/2026-08-28.md)
