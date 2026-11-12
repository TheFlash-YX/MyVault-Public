---
type: Note
domain: algorithms
title: LRU 缓存：哈希定位与双向链表维护顺序
aliases: [LRU Cache, LRU缓存, 最近最少使用, 双向链表缓存]
description: 146 手写 LRU 的节点设计、双哨兵、摘下与头插、字典同步，以及移动和淘汰的职责区别
tags: [leetcode, lru, hash-table, doubly-linked-list, cache]
status: learning
created: 2026-09-23
updated: 2026-09-23
---

# 概述

- 第 146 题属于本轮 CodeTop 清单；本次为用户主动要求的回练。
- 最终手写实现经代码审阅确认正确，尚未在会话中报告 LeetCode 提交结果。
- LRU 按最近使用时间淘汰，不按使用次数；成功读取、更新、新增都算使用，读取不存在的 key 不改变顺序。
- 哈希表负责快速定位，双向链表负责维护使用顺序；get、put 平均 O(1)，空间 O(capacity)。

# 详细内容

## 数据结构与不变量

- 字典存 `key → 节点`，而非仅存 value；二者都能快速查值，但前者还能直接定位并移动链表节点。
- 节点保存 key、val、pre、next；保留 key 是为了淘汰时反向定位字典记录。
- 双向链表在已知节点后能 O(1) 摘下；单链表通常还需要寻找前驱。
- head、tail 是哨兵，不存真实缓存；初始化为 `head.next=tail`、`tail.pre=head`。
- 节点构造函数给 key、value 默认值，便于无参数创建哨兵；真实节点必须保存传入的 key。
- 链表顺序为 `head ⇄ 最近使用 ⇄ …… ⇄ 最久未使用 ⇄ tail`。
- 每次公开操作完成后，字典与真实链表节点一一对应，数量不超过容量；节点之间的前后连接互相匹配。

## 两个链表基础操作

- `_remove(node)` 只摘下真实节点，不删除字典记录，因为移动节点也会复用它。

```python
node.pre.next = node.next
node.next.pre = node.pre
```

- `_add_to_head(node)` 接收节点本身，不是 `[key,node]`；前提是新节点或已摘下的节点。
- 头插需要四条连接，先保留旧首节点，最后再修改 head.next。

```python
node.next = self.head.next
node.pre = self.head
node.next.pre = node
self.head.next = node
```

- 移动节点必须先 remove 再 add；直接头插仍在链表中的节点可能产生环或破坏连接。
- 可选封装 `_move_to_head(node)` 复用这两步，不改变复杂度。

## get 与 put 的流程

- get 不存在的 key 返回 -1；存在时取节点、摘下、头插，返回 node.val。
- put 已存在的 key：更新 node.val，再摘下并头插，不增加数量，也不淘汰其他节点。
- put 新 key：本次采用“先淘汰、后插入”。若 `len(dic)>=capacity`，保存 `node=tail.pre`，摘下它并执行 `del dic[node.key]`。
- 随后创建 new_node，执行 `dic[key]=new_node`，将 new_node 头插。
- 该先淘汰方案依赖题目保证 capacity>0；不能把哨兵当真实节点删除。
- 另一种可行顺序是先插入、超容量后淘汰，但本次最终代码采用前一种，不混用判断条件。

## 本次手写错误与修正

- 构造函数要求参数却使用空参创建哨兵：给 key、value 默认值；同时补上 self.key。
- 写成 `tail.next=head`：应是 `tail.pre=head`。
- 头插漏了原首节点的反向连接：补上 `node.next.pre=node`。
- `_remove` 使用 `node in dic`：字典成员检查的是 key，不是 value 中的节点；已定位节点直接摘下即可。
- 使用 `dic.remove([node.key,node])`：字典没有该 remove 方法；只有真正淘汰时才用 `del dic[node.key]`。
- get 把 `[key,node]` 列表传给头插：应传 node，并先摘下。
- 已存在 key 的 put 只头插：同样必须先摘下，避免重复连接。
- 淘汰只更新链表：还需删除字典，否则 get 仍能找到过期节点且容量统计错误。
- 新节点误存为 `dic[key]=node`：应存 new_node；旧 node 可能未定义，也可能指向刚淘汰的节点。
- get 中头插调用多一个右括号：已修复语法错误。

## 自检重点

- 移动：只变顺序，字典条目和节点数量不变。
- 淘汰：链表和字典同时删除同一个 key。
- 容量 2：put(1,10)、put(2,20)、get(1)、put(3,30) 后应淘汰 2，get(2)=-1。
- 后续回练可检查容量 1、重复更新同一 key、连续读取头节点、未命中读取；这些是建议自测场景，不代表已运行测试。

# 相关链接

- [2026-09-23 学习日志](../../daily/2026-09-23.md)
