---
type: Note
domain: algorithms
title: 两个有序数组的中位数：二分划分
aliases: [寻找两个正序数组的中位数, median of two sorted arrays, 二分划分]
description: 第 4 题在较短数组上二分分配数量，用左右组数量与交叉边界条件定位中位数
tags: [leetcode, binary-search, median, partition]
status: learning
created: 2026-09-21
updated: 2026-09-21
---

# 概述

- 两个非递减数组不同时为空，寻找合并后的中位数，不需要实际合并。
- 当前进度为学习参考代码及推导，待独立手写验证。

# 详细内容

## 分配数量与边界

- 保证 nums1 较短，长度 m<=n；在 `[0,m]` 上二分 i。
- i 是 nums1 分给左组的数量，不是某个元素的下标；0 和 m 都合法。
- 左组总数 `left_size=(m+n+1)//2`，nums2 分给左组的数量 `j=left_size-i`。
- 短数组约束保证任意 i∈[0,m] 都对应合法 j∈[0,n]。
- a_left=nums1[i-1]，a_right=nums1[i]；b_left=nums2[j-1]，b_right=nums2[j]。
- 某侧不存在元素时，左边界用负无穷、右边界用正无穷，避免越界且不干扰比较。

## 划分是否合法

- 两个数组内部已有序，只需检查 `a_left<=b_right and b_left<=a_right`。
- 合法时，整个左组的每个元素都不大于整个右组的每个元素。
- 总长奇数时左组多一个，中位数是左组最大值 `max(a_left,b_left)`。
- 总长偶数时取 `(max(a_left,b_left)+min(a_right,b_right))/2`。

## 二分方向及其理由

- `a_left>b_right`：nums1 分给左组太多，设置 `right=i-1`。
- 若继续增大 i，a_left 不会变小；j 减小使 b_right 不会变大，因此不可能修复这一违例。
- 否则（已经排除合法划分）必为 `b_left>a_right`：nums1 分给左组太少，设置 `left=i+1`。
- i 增大时 j 自动减小，两个分界联动，不是单独移动一个元素。
- 示例 nums1=[1,2]、nums2=[3,4]，i=j=1 时 3>2；改为 i=2、j=0 后左组 [1,2]、右组 [3,4]，得到 2.5。
- 在单位成本模型下，时间 `O(log(min(m,n)+2))`（覆盖较短数组为空的常数情况），额外空间 `O(1)`。

# 相关链接

- [2026-09-21 学习日志](../../daily/2026-09-21.md)
- [二分查找：闭区间、答案边界与 lower_bound](./binary-search-boundaries.md)
