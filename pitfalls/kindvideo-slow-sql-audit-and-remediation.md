---
type: Pitfall
domain: backend
title: KindVideo 慢 SQL 审计：日志噪音、索引失效、回表与重复轮询
description: 2026-08-27 生产慢 SQL 审计的五个问题点，区分真实慢查询、慢日志噪音、已合入整改与待决策项
tags: [kindvideo, mysql, slow-query, index, covering-index, scheduler]
created: 2026-09-27
updated: 2026-11-12
---

# 现象

- 2026-08-12 至 08-27 的生产审计归纳出 **5 个问题点**；其中第 1 项是高频但本身不慢的查询刷满慢日志，不能说成“5 条 SQL 都很慢”。以下数据与整改状态来自 2026-08-27 的项目原稿，本笔记未重新连接生产库验证。
- **慢日志噪音**：`download_settings` 单例表查询在 15 天内留下约 25.7 万条记录，占慢日志 96%，遮蔽真正的慢查询。
- **趋势查询全表扫描**：安防大屏按天过滤 `abnormal_behavior.captured_at`，原稿记录 EXPLAIN cost 约 304K。
- **采样反连接耗时**：标注采样的 `NOT EXISTS` 查询单次 4–9 秒。
- **批量取数过度回捞**：`batch_get_item_payloads` 偶发秒级延迟，将任务下不需要的大 JSON payload 也取回应用层。
- **调度器轮询放大**：`download_task` 轮询 15 天约 670 万次，按状态过滤全表扫描；另观察到进度更新偶发行锁等待。

# 排查过程

- 原稿使用 `performance_schema` digest、慢日志聚合、`EXPLAIN` / `EXPLAIN ANALYZE` 和代码溯源；审计分析阶段对生产库只读，运维变更另列为 P0。
- 查慢日志配置时发现多个 `.cnf` 对 `log_queries_not_using_indexes` 设置冲突：后加载的配置覆盖了预期的 OFF。单例表 `SELECT ... FROM download_settings LIMIT 1` 没有条件与排序，因此被该开关高频记录；**只有一行的表并非性能瓶颈本身**。
- 查趋势查询时发现 `WHERE DATE(captured_at) >= ...` 在索引列上套函数，该查询计划无法用 `captured_at` 做范围定位；表约 9.2 万行，单行宽约 51 KB。
- 查采样反连接时发现 `annotation_task_item` 只有 `item_ref` 单列索引，命中后还需回表取得 `task_id`；大 JSON 使行均宽约 54 KB。原稿记录 buffer pool 1 GB、库体积 18 GB，冷读进一步放大代价。
- 查批量查询时发现接口以 `(task_id, item_ref)` 为键请求 payload，SQL 却只过滤 `task_id IN (...)`，多余行在 Python 端才被丢弃。
- 查调度器时发现 `download_task.status` 无索引；多个 gunicorn worker 各自启动 scheduler 线程，重复执行相同轮询。进度更新的行锁等待与多调度器竞争**可能相关，但原稿未将因果关系验证为定论**。

# 根因

1. **日志配置与查询形态叠加**：无索引日志开关被另一份配置重新打开，裸查单例表造成日志洪水；日志条数不等于慢 SQL 条数。
2. **过滤条件不可用于索引范围定位**：在 `WHERE` 中对 `captured_at` 做 `DATE()` 转换，破坏该查询的可索引范围条件；`GROUP BY` 或输出列中的 `DATE()` 不属于这次 WHERE 改写范围。
3. **反连接探测未覆盖**：索引仅有 `item_ref`，读取 `task_id` 需要回表；大行宽与偏小的 buffer pool 让回表成本显著增加。
4. **复合业务键只下推一半**：仅按 `task_id` 过滤，数据库先返回大量不需要的 payload。
5. **轮询缺索引且启动实例过多**：按 `status` 筛选、`created_at` 排序的查询缺少合适索引；每个 worker 一个 scheduler 放大请求次数。

# 解决方案

- **问题 1，P0 已在 2026-08-27 按原稿于生产生效**：关闭 `log_queries_not_using_indexes` 并修正冲突配置。原稿记录慢日志由每 4–5 秒一条变为 10 分钟内零新增。**P1 代码已合入 main**：单例查询改为 `ORDER BY id LIMIT 1`，使主键顺序明确；不能把关闭日志后的验证归功于这项代码修改。
- **问题 2，P1 代码已合入 main**：把按天过滤改为 `captured_at >= 起始日 00:00` 且 `captured_at < 截止日次日 00:00` 的半开区间；保留非法日期输入的旧行为回退。原稿记录 EXPLAIN cost 从 304K 降至 2213（约 137 倍），这是**估算计划成本变化，不是请求耗时提升 137 倍**；边界测试覆盖当天 23:59:59 与次日 00:00:00。
- **问题 3，P1 索引已合入 main**：增加覆盖索引 `(item_ref, task_id)`，使反连接探测不必为 `task_id` 回表。**P0 运维已在生产生效**：buffer pool 从 1 GB 扩至 3 GB。原稿未给出改造后的单查耗时，不写成“4–9 秒已降到某值”。
- **问题 4，P1 代码已合入 main**：再把 `item_ref IN (...)` 下推到 SQL，与 `task_id IN (...)` 一起缩小候选行。两个独立 `IN` 条件并不等于对 `(task_id, item_ref)` 元组做精确配对；这一点是由 SQL 语义作出的推断，不把它表述为只返回请求的键。
- **问题 5，P1 索引已合入 main**：新增 `(status, created_at)` 复合索引。**P2 尚待决策**：调度器收敛为单实例，以及减少进度状态的数据库写入；不能写成已解决多 worker 重复轮询或行锁问题。
- **状态边界**：原稿称 P1 代码与索引迁移随提交 `59516ef5` 合入 main，新增 9 个专项回归用例与受影响面 189 个用例通过；上线仍需执行迁移 `21ecacc874ef`。这证明代码合入与当时测试结果，**不证明 P1 已在生产部署或真实延迟已经下降**。

# 相关链接

- [MySQL 联合索引、回表与 EXPLAIN](../notes/backend/database/mysql-index-and-explain.md)
- [文件存储三原则：NAS 存文件、索引在库、直链播放](../notes/backend/architecture/file-storage-nas-direct-url.md)
