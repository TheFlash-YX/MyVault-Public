---
type: Note
domain: algorithms
title: 二分查找：闭区间、答案边界与 lower_bound
aliases: [二分查找, binary search, lower_bound, 搜索旋转排序数组, x 的平方根, 查找元素首尾位置]
description: 704 闭区间、69 可行边界、33 有序半区、34 两种边界解法、162 峰值，以及清单外旋转数组扩展
tags: [leetcode, binary-search, lower-bound, boundary]
status: learning
created: 2026-09-21
updated: 2026-09-22
---

# 概述

- 二分的核心是明确搜索区间的含义，并证明每一步排除的部分不可能有更好的答案。
- 704、69、34 已完成手写代码审阅；33 用户报告通过。34 的 lower_bound 版本目前为阅读与讲解进度。

# 详细内容

## 704：闭区间查找目标

- 初始 `left=0, right=len(nums)-1`，搜索区间为 `[left,right]`。
- `left==right` 时还剩一个候选，所以循环条件为 `left<=right`。
- 相等直接返回 mid；偏小则 `left=mid+1`，偏大则 `right=mid-1`。
- mid 已确定不是目标，因此不保留它；循环结束返回 -1。
- 空数组的 right 为 -1，循环自然跳过。时间 `O(log n)`，空间 `O(1)`。

## 69：寻找最后一个可行值

- 答案是满足 `k*k<=x` 的最大整数 k，而不是必须找到平方恰好等于 x 的数。
- 先处理 `x<=1`，再使用 `[1,x//2]`；Python 边界需要整数除法 `//`。
- 若 `mid*mid<=x`，执行 `ans=mid` 和 `left=mid+1`，尝试更大候选。
- 否则 `right=mid-1`，不更新答案。
- 后续可行候选只会更大，因此无需 `ans=max(ans,mid)`。
- 可选写法：不维护 ans，结束时直接返回 right；此时 right 为最后可行位置。
- 时间 `O(log x)`，空间 `O(1)`；0、1 单独为常数时间。

## 33：先判断哪一半有序

- 前提为旋转后的升序数组，元素互不重复；每轮至少一半有序。
- 先判断 mid 是否等于目标。
- 若 `nums[left]<=nums[mid]`，左半有序；仅当 `nums[left]<=target<nums[mid]` 时向左搜索，否则向右。
- 否则右半有序；仅当 `nums[mid]<target<=nums[right]` 时向右搜索，否则向左。
- 不能只比较 target 与 nums[left]；必须同时检查有序半区的两个端点。
- 时间 `O(log n)`，空间 `O(1)`；含重复元素的变体不能直接沿用此判断。

## 34：解法一——遇到相等时记录，继续搜索

- 题意：非递减数组中返回目标的第一个和最后一个下标，不存在返回 `[-1,-1]`。
- 初始化 `first=last=-1`，两次搜索都使用 `[0,len(nums)-1]`。
- 第一次相等时记录 first，继续向左；第二次相等时记录 last，继续向右。
- 不相等时，两次搜索都遵循普通二分的大小关系。

```python
# 第一次搜索中的相等分支
first = mid
right = mid - 1

# 第二次搜索中的相等分支
last = mid
left = mid + 1
```

- 两次搜索之间必须重置 left、right。曾误写 `right=len(nums)`，导致可能访问 nums[n]。
- 最小反例：`nums=[8], target=8`。错误范围 `[0,1]` 找到下标 0 后，仍会继续访问下标 1。
- 可选优化：第一次搜索后 first 仍为 -1，直接返回，无需第二次搜索。
- 总时间 `O(log n)`，空间 `O(1)`。

## 34：解法二——统一寻找 lower_bound

- 定义：返回第一个满足 `nums[i]>=target` 的下标；若不存在返回 n，包括空数组时返回 0。
- 它返回边界/插入位置，并不保证该位置的元素恰好等于 target。

```python
def lower_bound(nums, target):
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] >= target:
            right = mid - 1
        else:
            left = mid + 1
    return left
```

- 循环不变量：`[0,left)` 中都 `<target`，`(right,n)` 中都 `>=target`，中间闭区间尚未判定。
- 当 mid 满足条件时继续向左；当 mid 不满足时，排除它及其左侧。
- 结束时 `left=right+1`，未判定区间为空，left 正好位于两类元素的分界，所以无需额外 ans。
- “nums[left-1] < target、nums[right+1] >= target”只在真实下标存在时才能直接读值；越界部分是逻辑边界，不应实际访问。

```python
start = lower_bound(nums, target)
if start == len(nums) or nums[start] != target:
    return [-1, -1]
end = lower_bound(nums, target + 1) - 1
return [start, end]
```

- 整数数组中 `>=target+1` 等价于 `>target`；找到第一个更大元素后退一格，就是最后一个 target。
- `or` 短路保证 start 等于 n 时不访问 nums[start]。
- 示例 `[5,7,7,8,8,10]`：lower_bound(8)=3，lower_bound(9)=5，答案 `[3,4]`。
- 若 target 一直延伸到数组末尾，第二次 lower_bound 返回 n，end=n-1，依然正确。
- 若用于非整数有序数据，应直接二分第一个 `>target` 的位置，不能机械使用 target+1。
- 时间 `O(log n)`，空间 `O(1)`。

## 两种方法的区别

| 比较项 | 两次二分记录答案 | lower_bound |
| --- | --- | --- |
| 搜索目标 | 第一个/最后一个等于 target 的下标 | 第一个满足 >= 条件的位置 |
| 保存方式 | first、last，初值 -1 | 结束时返回 left |
| 找不到 | 保持 -1 | 可能返回更大元素位置或 n，需要验证 |
| 右端点 | 相等后继续向右 | 第一个 >target 的位置减一 |

## 162：寻找峰值——保留必含峰值的区间

- 数组非空，相邻元素不相等，数组外侧逻辑上视为负无穷；返回任意峰值的下标，不要求全局最大值。
- 初始 `[0,n-1]`，循环 `left<right`；比较 nums[mid] 和 nums[mid+1]。
- 向右上升时，右边继续上升最终到末尾，或者中途下降形成峰值，因此 `left=mid+1`。
- 向右下降时，左侧包含 mid 的区间保留峰值；mid 自身可能是峰值，故 `right=mid` 而非 mid-1。
- 更严格的区间保证：左边界外侧低于左端点，右边界外侧低于右端点（数组外用负无穷）；两种更新保持此性质，因而保留区间至少存在一个峰值。
- 结束时 left=right，只剩一个候选，返回 left 或 right 均可。
- 边界无需额外访问：循环中 mid<right，所以 mid+1<=n-1；首元素无须比较真实的左外邻居，尾元素不会作为 mid 进入循环。
- 单元素直接返回 0；递增数组收敛到末尾，递减数组收敛到开头。逻辑哨兵无需实际存储或读取。
- 用户已手写并报告通过；时间 O(log n)，空间 O(1)。

## 清单外扩展：153、154、81

- 这三题不在本轮固定 CodeTop 前 100 复习清单中；153、154 为额外完成，81 只讨论思路、未手写。不计入清单完成进度。

### 153：无重复的旋转数组最小值

- 比较 nums[mid] 与 nums[right]；大于时最小值在右侧，`left=mid+1`；小于时 mid 可能就是最小值，`right=mid`。
- 循环 left<right，结束返回 nums[left]，不是下标 left。
- 相邻候选位置不同且元素不重复，循环内不存在两者相等，因此原代码可用 else 处理小于。
- 时间 O(log n)，空间 O(1)，用户报告通过。

### 154：重复值造成方向歧义

- 大于、小于分支保持 153 的处理，只有相等时不能确定哪侧含更小元素。
- `[1,0,1,1,1]` 和 `[1,1,1,0,1]` 的中点与右端都相等，最小值却分居两侧。
- 相等时只做 `right-=1`：若右端不是最小值可以丢弃；若是最小值，mid 仍保留一个同值的最小值。
- 不能直接 right=mid，否则可能丢掉中间的 0。
- 最坏全相等时逐个排除，时间 O(n)，空间 O(1)，用户报告通过。

### 81：已讨论的判断要点

- 先检查 mid 命中，再处理 `nums[left]==nums[mid]==nums[right]`；三个值都不是目标，可同时 left+=1、right-=1，但不能直接丢弃整半区间。
- 排除歧义后按有序半区定位；左半目标范围为 `nums[left]<=target<nums[mid]`，不能只看上界。
- 该题未完成实现，后续不主动继续清单外扩展。

# 相关链接

- [2026-09-22 学习日志](../../daily/2026-09-22.md)

- [2026-09-21 学习日志](../../daily/2026-09-21.md)
- [两个有序数组的中位数：二分划分](./median-of-two-sorted-arrays.md)
- 用户提供的参考：灵茶山艾府，[第 34 题 lower_bound 题解](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/solutions/1980196/er-fen-cha-zhao-zong-shi-xie-bu-dui-yi-g-t9l9/)；本笔记基于会话中提供的代码与讨论整理。
