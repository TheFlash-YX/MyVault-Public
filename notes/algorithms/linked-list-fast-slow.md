---
type: Note
domain: algorithms
title: 链表快慢指针家族：141 判环、142 找入口与找中点
aliases: [环形链表, 快慢指针, 环入口, 找中点, fast slow pointer]
description: 快慢指针相对速度必相遇；142 入口推导 a=(n-1)(b+c)+c；找中点 A/B 两种循环条件的偶数分叉
tags: [linked-list, fast-slow-pointer, cycle-detection, math]
status: learning
created: 2026-08-26
updated: 2026-08-28
---

# 概述

- 快慢指针是链表三大利器之一（另两个：反转、合并），一个套路管三类题：**判环（141）、找环入口（142）、找中点（143/234 前置）**
- 找中点的循环条件有 A/B 两版，**偶数链差一格**：判环用 A、切半用 B，选错在 234 上会真翻车

# 详细内容

## 141 判环：相对速度必相遇

- slow 走 1、fast 走 2，同从 head 出发；**有环必相遇**：进环后 fast 对 slow 的相对速度是 1 格/轮，环内距离每轮 -1，至多绕环一圈必然追上
- 不会「跳过不相遇」：相对速度 1 意味着距离逐格缩小，不存在跨越；步长 >2 也能相遇但 2 最简
- 无环退出：fast 冲到 None（`while fast and fast.next`）；O(n) 时间、O(1) 空间
- 哈希表法 O(n) 空间也能判，但面试要的是 O(1) 快慢指针

## 142 找环入口：相遇后 fast 回 head 同速走

- 做法：相遇后 fast 回到 head，两指针**同速**各走 1 步，再次相遇点即入口
- 推导：设头→入口 a、入口→相遇点 b、相遇点→入口 c。slow = a+b，fast = a+b+n(b+c)，fast = 2×slow → **a = (n-1)(b+c) + c**
- 读法：a = (n-1) 整圈 + c。从 head 走 a 步到入口；从相遇点走 a 步 = 绕 n-1 圈 + c 步也到入口 → 必在入口相遇
- 这题不用 dummy（指针只读不写），见 [反转链表家族](linked-list-reversal.md) 的 dummy 使用口径

## 找中点：A/B 条件的偶数分叉（重点辨析）

| 条件 | 含义 | 偶数链 slow 落点 |
|---|---|---|
| A：`while fast and fast.next` | 还能走 ≥1 步就走 | n/2+1（前半多一个） |
| B：`while fast.next and fast.next.next` | 能走满 2 步才走 | n/2（前半段末尾） |

- 分叉根源：fast 每轮跳 2 步，**偶数链最后一跳「走 1 步能到尾、走 2 步掉 None」**——A 放它跳出界、slow 被多拖一格；B 拦下这一跳
- 奇数链两版完全一致（fast 都停在尾节点）
- **选用口诀：判环用 A（跑死为止），切半用 B（慢一拍保整齐）**
- 反例：234 回文 `[1,2]` 用 A → 后半段为空、比对 0 次 → 误判 True（实际非回文）；143 重排用 A 侥幸对（合并只要求后半 ≤ 前半），但那是巧合不是设计
- **148 归并排序把「切半用 B」升级为生死线**：A 条件 n=2 时 slow 停在尾节点 → `head2 = None` → `sortList(head)` 对原样 2 节点链无限递归；B 保证 n≥2 时两半都非空，递归必终止（实测 9 case）

## 应用地图

- 143 重排链表 = 找中点（B）+ [反转后半](linked-list-reversal.md) + [交错合并](linked-list-merge.md)
- 234 回文链表 = 找中点（B）+ 反转后半 + 逐对比较
- 148 排序链表 = 找中点（B，生死线）+ [21 归并](linked-list-merge.md)
- 19 删除倒数第 N 个 = 快慢间距 N（fast 先走 N 步，再同步走到尾，slow 停在目标前驱）

## 19 删除倒数第 N 个：间距模型 + dummy 算术

- 「找目标」用间距 n（剑指22），**「删目标」要落在前驱 → 间距 n+1**；同步走到 fast=None，slow 正好在目标前驱
- **为什么从 dummy 开始**：n==链长时目标是 head、前驱不存在；dummy 把问题变成「在 len+1 节的扩展链里找倒数第 n+1 个」，而 `n ≤ len` 保证它**一定存在**——删头和删中间共用同一行 `slow.next = slow.next.next`，零特判
- 反例（无 dummy 版）：`slow=fast=head` + fast 先走 n 步 + `while fast.next`，n==len 时 fast 走成 None → `None.next` 直接 AttributeError（实测）
- 模板：`dummy=ListNode(0,head)` → `slow=fast=dummy` → fast 先走 n+1 步 → `while fast:` 同步 → `slow.next=slow.next.next` → `return dummy.next`

# 相关链接

- [反转链表家族：206 整链、92 区间与 25 分组翻转](linked-list-reversal.md)
- [链表合并家族：21 双链合并与 23 合并 K 个](linked-list-merge.md)
- [双指针：快慢指针、就位区与逆向合并](two-pointers.md)
- [2026-08-26 学习日志](../../daily/2026-08-26.md)
- [2026-08-27 学习日志](../../daily/2026-08-27.md)
- [2026-08-28 学习日志](../../daily/2026-08-28.md)
