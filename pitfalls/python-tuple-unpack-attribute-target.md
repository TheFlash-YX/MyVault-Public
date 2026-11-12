---
type: Pitfall
domain: python
title: Python 多重赋值把解包目标写成属性，静默改掉旧对象
description: node.val, i, node = heapq.heappop(heap) 语法合法，但每轮把弹出值写进上一轮绑定的旧节点，污染已输出数据且不报错
tags: [python, tuple-unpack, assignment-order, leetcode]
created: 2026-08-25
updated: 2026-08-25
---

# 现象

- 默写 23 合并 K 个链表堆解法，运行不报错、不死循环，但输出链表的值被打乱：`[1→4→5, 2→3→6]` 前两个节点输出成 `2→1`
- 出错行：`node.val, i, node = heapq.heappop(heap)`——把弃元 `_` 手滑写成了属性目标 `node.val`

# 排查过程

- 与存档的正确版本逐行 diff，定位到解包的第一个目标
- 用两节点小用例手推：弹出的三元组按目标从左到右赋值，第一目标 `node.val = val` 写在**上一轮绑定的旧 node** 上（第一轮则是 for 循环漏下来的最后一个表头）
- 即每弹出一个新节点，就把上一个已挂进结果链的节点的 val 覆盖掉

# 根因

- Python 多重赋值的目标**从左到右**依次求值/赋值，属性目标 `node.val` 作用在赋值那一刻的旧对象上，右侧同名变量的新绑定要到最后一步才生效
- 堆的排序键是入堆时拷贝的元组值，节点被改不影响堆序 → 没有任何异常信号，纯静默数据损坏

# 解决方案

- 用不到的值写弃元：`_, i, node = heapq.heappop(heap)`（保留 i：值相等时 heapq 会去比第三元素 ListNode，无 `<` 定义直接 TypeError）
- 白板/默写完必跑一个 2 节点小用例——这类「语法完全合法」的事故只有小用例能抓到
- 同类默写/IDE 事故：敲 `dummy` 回车太快选中自动补全，引入 `from stringprep import in_table_c11`、`from multiprocessing import dummy` 垃圾导入（25/92 两份题解实测）；共享编辑器面试同样会发生，回车前扫一眼补全弹窗

# 相关链接

- [链表合并家族：21 双链合并与 23 合并 K 个](../notes/algorithms/linked-list-merge.md)
