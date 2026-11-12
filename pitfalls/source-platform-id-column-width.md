---
type: Pitfall
domain: backend
title: VARCHAR 列宽两次调整（20→64→32）：定宽前未核对真实数据长度
description: source_platform_id 先因 32 位十六进制 uid 超长从 String(20) 扩到 64，后又缩回 32；两次迁移折腾源于定列宽前没核对真实数据分布
tags: [mysql, varchar, migration, alembic, data-modeling]
created: 2026-08-24
updated: 2026-08-24
---

# 现象

- 外部平台 32 位十六进制 uid 写入 `source_platform_id` 时报 `Data too long for column`（原列宽 `String(20)`）
- 为快速兼容临时把列扩到 `String(64)`，次日又发现 64 是过度修正，真实 uid 恒为 32 位，需缩回 32
- 同一列一周内经历两次迁移（`b5963b8dedeb` 扩 64 → `a1b2c3d4e5f6` 缩 32），且每次都要动模型 / schema / 迁移三层

# 排查过程

- 先查高教课堂表 `source_platform_id` 定义：`String(20)`，而外部 uid 是 32 位 hex → 写不进去，需扩宽
- 扩到 64 后验证可行，但追问"真实数据最长到底多少"时发现：uid 固定 32 位 hex，20→64 都拍脑袋
- 缩列前先查库确认无超长行：`SELECT COUNT(*) FROM course WHERE CHAR_LENGTH(source_platform_id) > 32` 等三表均为 0 → 才敢缩

# 根因

- 定 VARCHAR 宽度前没先核对真实数据分布（`MAX(CHAR_LENGTH(col))`），先按"足够大"拍 64，属过度设计
- MySQL 缩列遇超长值会**直接报错而非截断**——安全但要先确认无超长行，否则线上迁移直接失败

# 解决方案

- 定列宽前先查真实数据：`SELECT MAX(CHAR_LENGTH(col)) FROM tbl`，按实际最大长度 + 余量定
- 缩列迁移前必须确认无超长行（三表 `CHAR_LENGTH > 32` 计数为 0）
- 改列宽的迁移写**条件执行**（`inspect().get_columns()` 判断列存在）+ **手写**，不用 `--autogenerate`（见 [alembic-autogenerate-drift](./alembic-autogenerate-drift.md)）
- 变更同时改：SQLAlchemy 模型、Pydantic schema（`max_length`）、Alembic 迁移，三层口径必须一致

# 相关链接

- [SQLite 与 MySQL 方言差异清单](../notes/backend/database/sqlite-mysql-dialect-differences.md)
