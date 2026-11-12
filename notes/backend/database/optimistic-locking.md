---
type: Note
domain: backend
title: 乐观锁：版本号 CAS 与并发取舍
aliases: [乐观锁, Optimistic Locking, 版本号并发控制]
description: 乐观锁的版本号条件更新机制、与悲观锁的取舍、ABA/时间戳/冲突处理，以及"读-比-写 vs SQL CAS"的实现层次辨析
tags: [乐观锁, 并发控制, cas, 事务, mysql]
status: learning
created: 2026-08-17
updated: 2026-08-27
---

# 概述

- 乐观锁是一种**并发控制思想**：假设冲突很少，操作前不加锁，写入时用版本号做条件更新检测冲突
- 不是数据库的真实锁——MySQL 没有 `OPTIMISTIC` 关键字，靠版本列实现的**应用层协议**
- 与悲观锁的取舍核心在冲突率：冲突少用乐观（无锁、无死锁、吞吐高），冲突多或临界区大用悲观

# 详细内容

## SQL 三步（CAS 语义）

```sql
-- ① 读版本
SELECT trail_validity, revision FROM trace WHERE id = 123;  -- 读到 revision = 7
-- ② 条件更新
UPDATE trace SET trail_validity = 'valid',
    revision = revision + 1
WHERE id = 123 AND revision = 7;   -- ← 全部秘密在这一行
-- ③ 影响行数：1=成功；0=冲突（别人已改成 8）
```

- `WHERE ... AND revision = 7` 让数据库一次原子完成"比较并交换"，不存在"检查完、更新前"被插队的缝隙
- 版本号每次成功写入单调 +1，旧版本号永远匹配不上已改过的行

## 乐观 vs 悲观对比

| | 悲观锁 | 乐观锁 |
|---|---|---|
| 假设 | 冲突经常发生 | 冲突很少发生 |
| 做法 | 先加锁（SELECT FOR UPDATE/分布式锁） | 写时靠版本号检测 |
| 死锁 | 有风险 | 无（根本不加锁） |
| 适用 | 临界区长、冲突率高、写多读少 | 临界区短、冲突率低、读多写少 |

## 冲突处理两派

- **自动重试**：适合幂等的机器操作，配重试上限 + 随机退避（防惊群）
- **报给用户（409）**：适合人工权威判断类操作——冲突说明判断前提已变，静默重试等于拿旧判断覆盖新世界

## 追问要点

- **ABA 问题**：值变走又变回，纯值比较会误判；版本号单调递增永不复用，天然免疫
- **版本号 vs 时间戳**：选版本号——时间戳有精度问题（同毫秒两次写区分不开）且时钟可能回拨
- **其他形式**：状态条件更新（`WHERE status='pending'`，状态即版本）、Java `AtomicInteger.compareAndSet`（CPU 原生 CAS）、Redis WATCH/MULTI、MVCC 本质也是乐观思想
- **与幂等的关系**：正交但常配套——乐观锁防"丢失更新"，幂等防"重复执行"
- **框架一等公民**：Hibernate `@Version`、SQLAlchemy `version_id_col` 都是自动做这套协议
- **冲突率突然变高**：乐观锁失效场景；缓解靠重试退避/拆小临界区，根治换悲观锁——主动说"乐观锁不是万能"是加分项

## ⚠️ 实现层次辨析：读-比-写 ≠ SQL CAS（高段位点）

- KindVideo 025 轨迹清洗的乐观锁实为**事务内读-比-写**：应用层 SELECT 读 revision 比对通过后无条件 UPDATE，**不是** `UPDATE ... WHERE revision=?` 的数据库层 CAS
- 严格并发分析：两个事务同时读到同一 revision 都能通过应用层比对，后提交者的无条件 UPDATE 覆盖前者——**丢失更新窗口存在**
- 实践中没出事：业务互斥占用（一条轨迹同一时刻只分给一个人）把并发前提封死了
- 教训：**乐观锁的正确性不完全来自机制本身，还来自"冲突概率低"的假设**；一旦新写入口破坏假设（如 057 裁定复用该入口），要把 CAS 下沉到数据库层（`UPDATE ... WHERE revision=?` 检查影响行数）

# 相关链接

- [2026-08-17 学习日志](../../../daily/2026-08-17.md)
- [DDD 领域驱动设计：战略与战术两层](../architecture/ddd-domain-driven-design.md)
- [ORM 跨语言对照：SQLAlchemy 与 Java 生态](./orm-python-vs-java.md)
- [事务、锁与幂等面试题](../../../interview/backend/transactions-locking-and-idempotency.md)
- [KindVideo 项目答辩](../../../interview/resume/kindvideo.md)
