---
type: Pitfall
domain: backend
title: SQLAlchemy 只 flush 不 commit：SQLite 写锁卡死后续 INSERT
description: SSE 流式回合中写工具经 update_profile 只 flush 不 commit，未提交事务持有 SQLite 写锁，后续持久化 INSERT 报 database is locked，且流中断时档案整个丢失；修法是把 commit 沉到最底层写方法里
tags: [sqlalchemy, sqlite, fastapi, sse, transaction, classinsight]
created: 2026-08-27
updated: 2026-08-27
---

# 现象

- ClassInsight 课堂助手（FastAPI SSE 流式对话）在多工具回合后，持久化助手回复的 INSERT 报 `sqlite3.OperationalError: database is locked`。
- 流被中断时，教师成长档案（跨会话记忆 profile）整个丢失——写工具明明执行成功，库里却没有记录。

# 排查过程

- 梳理请求生命周期：SSE `event_stream` 在流式开始前就退出主 `db_session()` 块（提交用户消息、装载历史），随后 `stream_turn` 跑 ReAct 工具。
- 写档案工具 `memory_write` / `memory_set_goal` 走 `MemoryStore.update_profile`——**只 `flush`、不 `commit`**，未提交事务持续持有 SQLite 写锁。
- 回合结束后新的 `db_session()` 块持久化助手回复，`INSERT assistant_messages` 抢写锁失败。
- 初版修复只在 `memory_set_goal` 后加 `ctx.session.commit()`，不对称：`mock responder` 从不在流中调 `memory_write`，既有测试只暴露 goal 路径，`memory_write` 的同构缺陷漏网，评审时才被发现。

# 根因

- SQLite 单写者模型：任一连接的未提交写事务持有写锁，其他连接的写操作立即报 `database is locked`（默认无等待）。
- SQLAlchemy 的 `flush ≠ commit`：flush 把 SQL 发到连接但事务不结束，锁不释放。
- 跨会话档案的写入发生在「外层会话块已退出、回复持久化块未开启」的窗口里，没有任何一方对这次写负责提交。
- 深层教训：mock 测试通过 ≠ live 路径覆盖——mock 不走真实写工具分支，缺陷只在 live 多工具回合暴露。

# 解决方案

- 把提交集中沉到最底层写方法：`update_profile` 内 `flush` 后立即 `self._session.commit()`，档案即时落盘（流中断也留存），同时释放写锁。
- 所有调用路径统一受益：两个流内写工具、反馈同步 `_sync_feedback_to_profile`、会话收尾提炼 `_finalize_latest_session`；外层 `db_session` 的 commit-on-exit 对已提交状态是无害 no-op。
- 测试固化：`test_update_profile_commits_durably` 单测 + `TestImprovementGoal` 4 个集成用例覆盖持久化路径。
- 通用原则：流式/SSE 场景里「谁拥有事务边界」必须显式设计；写操作需要跨会话块存活时，提交点沉到最底层写方法，而不是靠调用方记得 commit。

# 相关链接

- [ORM 跨语言对照：SQLAlchemy 与 Java 生态](../notes/backend/database/orm-python-vs-java.md)
- [SQLite 存储类型与外键约束](../notes/backend/database/sqlite-types-and-foreign-keys.md)
