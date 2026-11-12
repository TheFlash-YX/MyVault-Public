---
type: Note
domain: backend
title: SQLite 存储类型与外键约束：REAL 时间戳、ON DELETE CASCADE、PRAGMA foreign_keys
aliases: [SQLite REAL, SQLite 外键, PRAGMA foreign_keys, ON DELETE CASCADE, SQLite 时间戳]
description: SQLite 只有 5 种存储类型、无原生日期类型，时间戳用 REAL 存浮点秒；REFERENCES / ON DELETE CASCADE 逐段含义；SQLite 默认不强制外键的历史原因与按连接开启的坑
tags: [sqlite, 外键, 时间戳, 数据库]
status: learning
created: 2026-08-21
updated: 2026-08-21
---

# 概述

- SQLite 只有 5 种存储类型、**没有原生日期类型**，时间戳可用 REAL 存 `time.time()` 的浮点秒（带亚秒精度）
- `REFERENCES 表(列) ON DELETE CASCADE` = 外键校验 + 删父行时子行自动陪葬；但 SQLite **默认不强制外键**，必须每个连接 `PRAGMA foreign_keys = ON`

# 详细内容

## 存储类型与时间戳

- 5 种存储类型：`NULL` / `INTEGER` / `REAL` / `TEXT` / `BLOB`——`REAL` 即 8 字节双精度浮点（≈ Python float)
- 时间戳三种常见存法：`TEXT`(ISO8601 字符串）、`INTEGER`（整秒 epoch)、`REAL`(**浮点 epoch 秒**，小数部分是亚秒精度）
- REAL vs INTEGER 存时间戳：

| | INTEGER | REAL |
|---|---|---|
| 精度 | 整秒 | 毫秒/微秒级 |
| 适用 | 只需按秒排序 | 精确排序事件、算耗时（`finished_at - created_at`) |

- 实例（DeepTutor `turns` 表）：`created_at REAL` ← Python 侧 `time.time()` 直接写入；`turn_id` 里 `int(now * 1000)` 是同一时间戳换成毫秒嵌进 ID；读回用 `datetime.fromtimestamp(ts)`

## 外键声明逐段拆解

```sql
session_id TEXT NOT NULL REFERENCES sessions(id) ON DELETE CASCADE
```

| 片段 | 含义 |
|---|---|
| `TEXT NOT NULL` | 存父表 id，且不允许为空——每条子记录必须有所属 |
| `REFERENCES sessions(id)` | 外键：本列的值必须存在于 `sessions.id` 中，否则 INSERT 报 `FOREIGN KEY constraint failed` |
| `ON DELETE CASCADE` | 删父行时，引用它的子行**自动一起删**；默认是 `NO ACTION`——还有子行引用时**拒绝删父行** |

- 实例（DeepTutor):`sessions` 被 `messages` / `turns` / `turn_events` 三表引用且全部 CASCADE——删一个会话，三层数据一次清干净，不留孤儿

## 默认不强制：`PRAGMA foreign_keys = ON`

- **不开时，外键声明只是"注释"**：脏插入（引用不存在的父行）成功、删父行后子行成孤儿、CASCADE 不触发
- 历史原因：外键强制是 SQLite **3.6.19(2009）才加入**的；之前十几年的策略是"解析但不执行"（让其他数据库导出的 schema 能直接跑）。默认关闭是为向后兼容——老库里可能已存在违反外键的数据，默认开会让老应用升级后莫名报错
- 三个使用要点：
  - **按连接生效**，不写进数据库文件——每个新连接都要重新执行；用 `sqlite3` CLI 手动排查时别忘了开，否则看到的行为和 app 不一致
  - 不能在事务内设置（`BEGIN` 之后执行静默无效）
  - ORM/驱动通常有自动开启的钩子；裸用 sqlite3 模块要在建连接处自己执行
- 实例（DeepTutor):`_connect()` 里每次建连接都 `PRAGMA foreign_keys = ON`(`sqlite_store.py:456`)，保证所有连接路径上级联删除都真实生效

# 相关链接

- [SQLite 与 MySQL 方言差异清单](./sqlite-mysql-dialect-differences.md) —— 外键默认开关的方言对比视角（MySQL InnoDB 默认启用）
