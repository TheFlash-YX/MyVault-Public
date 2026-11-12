---
type: Note
domain: backend
title: 不引入 Redis/MQ 的架构取舍：DB 任务表与进程内缓存
aliases: [Redis MQ 替代方案, 进程内缓存, DB 任务队列]
description: 单进程内网服务可以不用 Redis/MQ——异步任务=DB任务表+守护线程轮询、缓存=进程内TTL字典；关键是留端口抽象随时升级
tags: [redis, 消息队列, 缓存, 异步任务, 架构取舍, 依赖倒置]
status: learning
created: 2026-08-17
updated: 2026-08-27
---

# 概述

- 中间件的引入应由部署形态与规模决定，不是简历驱动——单进程内网服务引入 Redis/MQ = 多运维一个组件 + 多一层网络往返，收益为负
- KindVideo 的替代方案：异步任务 = **DB 任务表 + 守护线程轮询**；缓存 = **进程内 TTL 字典**；两者都留了升级扩展点
- 面试口径：答成**取舍而非不会**——"当前规模用最小方案，扩展点已预留"

# 详细内容

## 异步任务：DB 任务表即队列

- 任务持久化在 `download_task` 表，状态机流转：pending → running → waiting_retry / completed
- 应用 startup 起 daemon 线程跑调度循环，每 10 秒一个 tick 拉取 pending 任务，线程池提交 worker 执行
- 存储不可用时任务进 `waiting_retry` 退避，进程重启任务不丢（startup 有恢复逻辑）——**持久化 + 可查询 + 可对账**，这是相对内存队列的优势
- 进度推送走 WebSocket：worker 写进程内进度 dict（模块级 dict + threading.Lock），WS 每秒读一次推前端

## 缓存：进程内 TTL 字典

- 实现：模块级 dict + threading.Lock + TTL（600s）+ 前缀失效
- 缓存对象是昂贵的全表聚合统计（未缓存 27s → 命中后毫秒级）
- key 带业务前缀（如 `trace:global_stats`），写路径主动调失效——缓存一致性靠"写时失效 + 短 TTL"双保险

## 端口抽象预留升级（关键设计）

- `ICachePort`（get/set/invalidate/invalidate_all）定义在 **domain 层**，当前绑定内存实现，DI 注入到用例
- 将来上 Redis：infrastructure 加一个 Redis 实现 + DI 换绑定，use case 一个字符不动——依赖倒置的经典应用
- MQ 同理：任务表轮询换 Celery/真 MQ 时，调度入口收敛在一个 scheduler 模块

## 代价与边界（主动讲）

- 内存缓存在 gunicorn 多 worker 下每 worker 一份——靠短 TTL + 写路径主动失效兜底，非强一致
- DB 轮询有秒级调度延迟、空转 tick；单机并发上限
- 量上去（多实例部署/任务洪峰/需要跨进程失效）就是换 Redis/MQ 的信号

# 相关链接

- [2026-08-17 学习日志](../../../daily/2026-08-17.md)
- [DDD 领域驱动设计：战略与战术两层](./ddd-domain-driven-design.md)
- [文件存储三原则：NAS 存文件、索引在库、直链播放](file-storage-nas-direct-url.md)
- [Human-in-the-loop 与暂停恢复面试题](../../../interview/agents/human-in-the-loop.md)
- [Redis 面试题](../../../interview/backend/redis.md)
- [Kafka 与消息队列面试题](../../../interview/backend/kafka-and-messaging.md)
