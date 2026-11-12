---
type: interview
scope: topic
area: backend
topics:
  - mysql
  - transactions
  - indexes
created: 2026-08-27
updated: 2026-09-08
---

# MySQL

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。

## MySQL 的事务四大特性是什么？

<!-- canonical_key: backend.mysql.mysql-de-shi-wu-si-da-te-xing-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 四种事务隔离级别分别是什么？

<!-- canonical_key: backend.mysql.si-zhong-shi-wu-ge-li-ji-bie-fen-bie-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## MySQL 默认隔离级别是什么？

<!-- canonical_key: backend.mysql.mysql-mo-ren-ge-li-ji-bie-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 什么是脏读、不可重复读、幻读？

<!-- canonical_key: backend.mysql.shen-me-shi-zang-du-bu-ke-zhong-fu-du-huan-du -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## MVCC 是什么？

<!-- canonical_key: backend.mysql.mvcc-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## InnoDB 如何实现 MVCC？

<!-- canonical_key: backend.mysql.innodb-ru-he-shi-xian-mvcc -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## undo log 是干什么的？

<!-- canonical_key: backend.mysql.undo-log-shi-gan-shen-me-de -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## redo log 是干什么的？

<!-- canonical_key: backend.mysql.redo-log-shi-gan-shen-me-de -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## binlog 是干什么的？

<!-- canonical_key: backend.mysql.binlog-shi-gan-shen-me-de -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## redo log 和 binlog 有什么区别？

<!-- canonical_key: backend.mysql.redo-log-he-binlog-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 什么是聚簇索引？

<!-- canonical_key: backend.mysql.shen-me-shi-ju-cu-suo-yin -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 为什么 InnoDB 主键最好不要太长？

<!-- canonical_key: backend.mysql.wei-shen-me-innodb-zhu-jian-zui-hao-bu-yao-tai-zhang -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 联合索引最左匹配原则是什么？
<!-- canonical_key: backend.mysql.lian-he-suo-yin-zui-zuo-pi-pei-yuan-ze-shi-shen-me -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- (a,b,c) 按索引定义的列顺序有序，SQL 条件书写顺序不影响等值前缀匹配。
- a=1 AND c=3 仍可用 a 定位；缺 b 不会让整个索引失效。
- a=1 AND b>2 AND c=3 的本例中，a/b 定范围，c 继续筛选；不能泛化为所有范围运算之后完全不能使用索引。

### 易追问

- 覆盖索引是否要求所有索引列都参加定位？

### 关联笔记

- [MySQL 索引与 EXPLAIN](../../notes/backend/database/mysql-index-and-explain.md)

## 什么情况下索引会失效？
<!-- canonical_key: backend.mysql.shen-me-qing-kuang-xia-suo-yin-hui-shi-xiao -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- 先区分不满足范围定位条件和优化器按成本未选索引；不存在一个字段失效则全索引失效的普遍规则。
- 不满足最左前缀可能无法做该前缀查找，但仍需看计划；小表或高命中率加大量回表可能选全表扫描。
- 本轮仅复习上述边界；函数、隐式转换等具体限制未系统学习，回答尚不完整。

### 易追问

- 索引存在为什么不一定更快？

### 关联笔记

- [MySQL 索引与 EXPLAIN](../../notes/backend/database/mysql-index-and-explain.md)

## Explain 看哪些字段？
<!-- canonical_key: backend.mysql.explain-kan-na-xie-zi-duan -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- type 看访问方式，key 看计划选择的索引，rows 是预估扫描量，不是实际返回量。
- Extra 中 Using index 表示覆盖，Using index condition 表示 ICP，Using where 表示条件过滤；后者不是索引失效标记。

### 易追问

- rows=20 是否必定返回20行？

### 关联笔记

- [MySQL 索引与 EXPLAIN](../../notes/backend/database/mysql-index-and-explain.md)

## 如何判断一个 SQL 是否走索引？
<!-- canonical_key: backend.mysql.ru-he-pan-duan-yi-ge-sql-shi-fou-zou-suo-yin -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- 通过 EXPLAIN 看 key/type 等，而不是仅看是否建过索引或 Extra 是否有 Using index。
- 覆盖是取列是否需要回表的问题，范围定位是如何缩小查找区间的问题；索引命中也不保证性能最优。

### 易追问

- ICP 把1000条候选筛到100条，减少的是扫描还是回表？

### 关联笔记

- [MySQL 索引与 EXPLAIN](../../notes/backend/database/mysql-index-and-explain.md)

## 数据量从十万增长到一亿以后，你会怎么优化？

<!-- canonical_key: backend.mysql.shu-ju-liang-cong-shi-wan-zeng-zhang-dao-yi-yi-yi-hou-ni-hui-zen-me-you-hua -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 分库分表什么时候需要做？

<!-- canonical_key: backend.mysql.fen-ku-fen-biao-shen-me-shi-hou-xu-yao-zuo -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Offset Pagination 有什么问题？

<!-- canonical_key: backend.mysql.offset-pagination-you-shen-me-wen-ti -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Cursor Pagination 怎么实现？

<!-- canonical_key: backend.mysql.cursor-pagination-zen-me-shi-xian -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 多张大表关联查询慢，如何排查和优化？
<!-- canonical_key: backend.mysql.large-table-join-optimization -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- 先 EXPLAIN 检查各表访问方式、索引和预估扫描量；再检查连接键、过滤条件及合适的联合索引。
- 只取必要列，评估覆盖索引和回表成本；不要为了覆盖无限扩宽索引。
- 结合数据量和命中比例评估，最后比较修改前后的执行计划与实际耗时，不把强制走索引当作目标。

### 易追问

- users.id 连接 orders.user_id 时，订单侧索引建在哪？

### 关联笔记

- [MySQL 索引与 EXPLAIN](../../notes/backend/database/mysql-index-and-explain.md)
