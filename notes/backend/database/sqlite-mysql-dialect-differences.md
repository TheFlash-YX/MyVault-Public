---
type: Note
domain: backend
title: SQLite 与 MySQL 方言差异清单
aliases: [SQLite MySQL 差异, 方言分支 SQL]
description: 测试跑 SQLite in-memory、生产 MySQL 场景下的三类坑：UPSERT 语义、跨表 UNION collation 1271、FK 级联默认行为
tags: [sqlite, mysql, upsert, collation, 外键, 测试策略]
status: learning
created: 2026-08-17
updated: 2026-08-21
---

# 概述

- "测试跑 SQLite in-memory、生产用 MySQL"的开发模式下，方言差异会让**测试全绿 ≠ 生产没问题**
- 三类已知差异：UPSERT 语义、跨表 UNION 的 collation 冲突、外键约束默认开关

# 详细内容

## UPSERT 语义

- MySQL：`ON DUPLICATE KEY UPDATE` —— 原地更新，行 id 不变，不触发 ON DELETE CASCADE
- SQLite：`ON CONFLICT(...) DO UPDATE` —— 同样是原地更新语义（正确选择）
- ⚠️ SQLite 的 `INSERT OR REPLACE` 是**删除再插入**：id 会变 + 触发子表 `ON DELETE CASCADE`——会把关联行悄悄删掉，勿用

## 跨表 UNION 的 collation 冲突

- 现象：MySQL 中两表 VARCHAR 列 collation 不同（如 utf8mb4_unicode_ci vs utf8mb4_0900_ai_ci）做 UNION/JOIN 时报 **1271 Illegal mix of collations**
- SQLite 无 collation 概念，UNION 直接通过——**测试永远不会暴露此问题**
- 解法：`CAST(col AS CHAR)` 统一 collation，或查询中显式 `COLLATE utf8mb4_unicode_ci`

## 外键默认行为

- MySQL InnoDB 外键默认启用；SQLite **默认关闭**，要 `PRAGMA foreign_keys = ON` 才生效
- 写方言分支 SQL 时，SQLite 测试路径必须带 FK 约束跑，否则级联删除的惊喜测不出来

## 通用准则

- 跨表 UNION/JOIN 涉及不同 collation 的 VARCHAR 列时，始终用 CAST 或显式 COLLATE 统一
- 方言分支 SQL 的每条分支都要在对应方言下测过
- 兜底手段：联调环境定期还原生产备份，让 MySQL 真身问题在上线前暴露

# 相关链接

- [2026-08-17 学习日志](../../../daily/2026-08-17.md)
- [ORM 跨语言对照：SQLAlchemy 与 Java 生态](./orm-python-vs-java.md)
- [SQLite 存储类型与外键约束](./sqlite-types-and-foreign-keys.md) —— PRAGMA foreign_keys 的机制详解（历史原因、按连接生效）
