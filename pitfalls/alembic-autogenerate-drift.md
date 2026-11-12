---
type: Pitfall
domain: backend
title: Alembic --autogenerate 在 schema 漂移的本地库上生成无关迁移
description: autogenerate 是对比"本地库当前状态 vs 模型"，本地/历史/测试库缺列或版本漂移时会 diff 出无关 DDL；改为手写最小条件迁移
tags: [alembic, migration, schema-drift, sqlalchemy]
created: 2026-08-24
updated: 2026-08-24
---

# 现象

- 用 `alembic revision --autogenerate` 生成本应只改一个列宽的迁移，结果夹带删除列、改类型等**无关 DDL**，差点误改库
- 同一个迁移脚本在干净的 CI 库能过，在本地库/历史库上执行失败（如缺列报 `Unknown column`）

# 排查过程

- 先怀疑模型定义错，逐列对比模型与迁移发现迁移内容与本次改动无关
- 意识到 autogenerate 的语义：**对比当前库的 schema 与模型定义**，本地开发库与其他环境长期漂移（历史测试建过又删的表、手工改过的列），差异全被当成"迁移内容"吐出来
- 本地 `lesson` 表缺 `source_platform_id` 列 → 直接 `ALTER` 失败 → 需要条件执行

# 根因

- `--autogenerate` 依赖本地库状态，库与模型漂移越久，噪声越大；本地库不等于生产库
- 无条件 `op.alter_column` 在缺列的历史/测试库上直接炸

# 解决方案

- **手写最小迁移**：只写本次要改的列，不用 autogenerate
- **条件执行**：`inspect(op.get_bind()).get_columns(table)` 判断列存在再 `ALTER`，兼容缺列的历史/测试库
- 迁移前确认 `alembic heads` 为**单头**（远程合并过会 fork 出多头，需先合链）
- 迁移后用集成测试断言 head 值（`assert heads == ["<新revision>"]`），防链断裂

# 相关链接

- [SQLite 与 MySQL 方言差异清单](../notes/backend/database/sqlite-mysql-dialect-differences.md)
- [VARCHAR 列宽两次调整](./source-platform-id-column-width.md)（本坑的直接背景）
