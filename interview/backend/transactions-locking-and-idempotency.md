---
type: interview
scope: topic
area: backend
topics:
  - transactions
  - locking
  - idempotency
created: 2026-08-27
updated: 2026-09-03
---

# 事务、锁与幂等

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。
- 关联知识与题集：
- [乐观锁：版本号 CAS 与并发取舍](../../notes/backend/database/optimistic-locking.md)
- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)
- [退款系统场景设计](./refund-system-design.md)

## 唯一索引如何实现幂等？

<!-- canonical_key: backend.locking.wei-yi-suo-yin-ru-he-shi-xian-mi-deng -->

### 考察点

- 幂等的目标不是“挡掉请求”，而是保证同一业务请求只执行一次，并让重复请求返回同一笔业务结果。
- 能否说清唯一键粒度、数据库唯一约束、并发创建和异常返回。

### 回答框架

- 唯一索引实现幂等的核心是先定义业务幂等键，例如支付退款里的 `refund_id`、`refund_order_id` 或 `merchant_refund_no`，而不是简单使用 `user_id + order_id`。
- 创建业务单时把幂等键落到数据库唯一索引；第一次插入成功就继续执行，重复请求插入失败或查到已有记录时，不再执行副作用操作。
- 重复请求应该返回已有业务单状态，例如 `PROCESSING`、`SUCCESS` 或 `FAILED`，而不是简单丢弃。
- 对支付、退款这类外部副作用场景，本地幂等键还要传给下游渠道，保证本地和渠道使用同一个业务单号查单、对账和排重。

### 易追问

- 如果第一次请求创建记录成功，但调用外部系统超时怎么办？
- 为什么一个订单不能只允许一个退款请求？
- 幂等和乐观锁分别解决什么问题？

### 关联笔记

- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)
- [乐观锁：版本号 CAS 与并发取舍](../../notes/backend/database/optimistic-locking.md)

## 数据库事务和分布式事务有什么区别？

<!-- canonical_key: backend.locking.shu-ju-ku-shi-wu-he-fen-bu-shi-shi-wu-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 乐观锁更新 SQL 一般怎么写？

<!-- canonical_key: backend.locking.le-guan-suo-geng-xin-sql-yi-ban-zen-me-xie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 为什么状态流转最好和版本号校验放在一个事务中？

<!-- canonical_key: backend.locking.wei-shen-me-zhuang-tai-liu-zhuan-zui-hao-he-ban-ben-hao-xiao-yan-fang-zai-yi-ge-shi-wu-zhong -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-
