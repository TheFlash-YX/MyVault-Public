---
type: Note
domain: agents
title: DeepTutor Agent Runtime：Turn 生命周期与事件流
aliases: [DeepTutor Runtime, Agent Runtime, Turn Runtime, ChatOrchestrator, StreamBus, Message Tree, Session Store, Partner 运行时复用与隔离, Synthetic Scope]
description: 以 DeepTutor 为例梳理 Turn 生命周期、事件流，以及 Partner 通过 UnifiedContext、Synthetic Scope 和工具策略复用 Runtime 的隔离边界
tags: [deeptutor, agent-runtime, turn, orchestrator, streambus, session, persistence]
status: learning
created: 2026-08-23
updated: 2026-09-21
---

# 概述

- Agent Runtime 不是大模型本身，而是管理一次 Agent 任务完整生命周期的执行基础设施，负责会话、上下文、模型与工具、异步执行、流式输出、持久化、取消和恢复。
- DeepTutor 的一次完整调用链是 `Payload -> TurnRuntimeManager -> UnifiedContext -> ChatOrchestrator -> Capability -> Agent Loop/Pipeline -> StreamBus -> 持久化与 WebSocket`。
- 一个 Turn 表示从一次用户提交开始，到完成、失败、取消或暂停为止的一次任务实例；一个 Turn 内可以包含多轮 LLM 调用和工具调用。
- DeepTutor 的顶层 Capability 路由是显式规则路由，Capability 内部才由 LLM 根据语义决定直接回答、调用工具还是继续循环。

# 详细内容

## 核心对象边界

- `Session`：一段完整会话，持有标题、历史消息、压缩摘要和会话级偏好，可以包含多个 Turn。
- `Turn`：一次用户请求对应的运行实例，具有独立 `turn_id`、状态、错误信息和事件序列。
- `Message`：持久化的用户或助手消息；一个 Turn 通常产生一条用户消息和一条助手消息，但内部可发生多轮模型交互。
- `StreamEvent`：执行过程中的最小可观察事件，如 `content`、`tool_call`、`tool_result`、`sources`、`error` 和 `done`。
- `Payload`：调用方启动 Turn 时提交的原始请求参数，描述“这次做什么以及使用什么配置”。
- `UnifiedContext`：Runtime 基于 Payload、Session 历史、Memory、Persona、知识库和附件构建出的服务端执行快照。

## Payload 到 Turn 的验证与登记

- Runtime 先解析 `capability`，未指定时默认使用 `chat`，再按 Capability 对应的 Pydantic Schema 校验 `config`；未知字段、非法枚举或越界数值会在创建 Turn 前被拒绝。
- `_regenerate`、`_persist_user_message`、`subagent_consult_budget` 等 Runtime 私有配置会先从公开配置中剥离，避免触发 Capability 的 `extra="forbid"`，校验后再合并回来。
- `ensure_session(session_id)` 在 Session 存在时恢复它，不存在或未传 ID 时创建新 Session，并读取其中的 `preferences`。
- Sticky Preferences 遵循“本次 Payload 显式值优先，缺失时继承 Session 值”的思路；当前后端直接恢复的重点是 `persona`、`llm_selection` 和 `mastery_path_id`。
- `persona=""` 表示显式清空 Persona，而完全不传 `persona` 才表示沿用 Session 中的选择。
- 模型选择先与当前用户的授权范围求交集，再对公共模型目录和个人模型目录做存在性校验；非管理员没有可用模型授权时直接拒绝 Turn。
- `tools` 未传时从用户工具设置补全，显式传空列表则表示关闭用户可选工具；补全后还要与管理员下发的工具白名单求交集。
- Runtime 会扫描当前 Session 中数据库状态仍为 `running` 的 Turn；若本进程已没有对应的内存执行对象，说明服务重启或崩溃留下了孤儿记录，需要标记为 `failed`。
- 完成校验后，将最终有效的 Capability、工具、知识库、语言、模型等配置写回 Session Preferences，再向 `turns` 表插入一条 `status=running` 的新记录。
- Store 创建 Turn 时再次检查 Session 是否存在、是否已有健康的 `running` Turn，从持久化层防止同一 Session 并发启动两个 Turn。
- 新 Turn 会先登记为进程内 `_TurnExecution`，再发布 `SESSION` 事件并通过 `asyncio.create_task()` 启动 `_run_turn()`，避免数据库已有运行记录但本进程尚未声明所有权的竞态窗口。

## Context、Session 与 Memory

- Conversation History 是当前 Session 的历史消息，主要保证本次对话连贯；Memory 是跨会话保存的用户偏好、长期事实和学习状态；两者不等价。
- Memory、Conversation History 和 RAG 的边界不同：History 负责当前 Session 的短期上下文，Memory 负责跨会话用户状态，RAG 负责外部领域知识。
- `ContextBuilder` 不会无上限塞入历史消息，而是按上下文窗口预算组合压缩摘要和最近消息，防止历史内容挤占当前回答空间。
- Payload 中的附件、笔记、书籍和历史会话引用会在服务端解析后加入有效用户消息；模型看到的是构建后的 `UnifiedContext`，不是未经处理的原始 Payload。
- DeepTutor 默认会话持久化包含 `sessions`、`messages`、`turns`、`turn_events` 四类核心记录；消息保存对话结果，事件保存执行轨迹，两者用途不同。
- 长期 Memory 使用 L1/L2/L3 分层文件体系，知识沉淀由显式工作流驱动，不是每个 Turn 完成后都自动执行一次记忆合并。
- L3 Memory 可以通过 `memory_context` 直接进入 Prompt，也可以由模型通过 `read_memory` Tool 按需读取；两种方式分别对应预注入和运行时按需检索。

## 消息树与分支上下文

- DeepTutor 在数据库中把消息保存为扁平行，但每行用 `parent_message_id` 指向当前路径上的前一条消息，因此逻辑结构是一棵树。
- 第一条消息的 parent 为 `NULL`；同一个 parent 下出现多个 child 时，这些 child 是编辑旧消息或重新生成形成的候选分支。
- 前端一次只选择并展示一条从根到叶子的路径，不会同时把整棵树铺给用户。
- 构建 LLM 上下文时，后端从当前 leaf 沿 `parent_message_id` 反向回溯，只保留祖先链并排除兄弟分支，避免其它版本的回答、附件和摘要泄漏到当前分支。
- `role=user` 表示消息在 LLM 协议中属于提问方，不表示一个真实账号；树上的多个 user 节点通常是同一登录用户在不同轮次或不同编辑分支中的消息。
- 登录身份与消息角色是两套概念：真实用户隔离发生在 Session/PathService/PocketBase `user_id` 边界，消息行只需要记录 `role`、`content` 和父子关系。
- `ask_user` 的回复是特殊情况：它不会新增普通 `role=user` 树节点，而是替换发起工具调用时对应的 `role=tool` 内容，并在同一 Turn 中恢复 Agent Loop。

## Web 与 CLI 的统一持久化

- Web 入口通过 FastAPI/WebSocket 进入 Runtime；CLI 通过 `DeepTutorApp` Facade 进入 Runtime，两者最终都调用同一个 `TurnRuntimeManager` 和 `SessionStoreProtocol`。
- 因此 Web 与 CLI 共用 Session、Message、Turn、TurnEvent 数据模型，也共用上下文摘要、会话偏好、消息分支和事件重放逻辑；CLI 没有另外把会话保存成 JSON 或终端历史。
- 默认零配置后端是 `SQLiteSessionStore`，核心数据落在当前用户作用域的 `chat_history.db`；配置 PocketBase 后，`get_session_store()` 会整体切换为 `PocketBaseSessionStore`。
- “持久化方式相同”不等于“物理数据一定相同”：两端只有在 `DEEPTUTOR_HOME`、存储后端和用户作用域一致时才会读写同一份会话。
- 无 HTTP 登录上下文的 CLI 默认以 local admin 身份运行；多用户 Web 中的普通账号使用独立用户作用域，因此默认不会与 local-admin CLI 共享物理数据。
- SQLite 通过每用户独立数据库文件隔离；PocketBase 使用共享服务，并在 Session 边界按 `user_id` 过滤。

## Orchestrator 与 Agent Loop

- `ChatOrchestrator` 根据 `context.active_capability` 从注册表查找 Capability；没有指定时使用 `chat`，指定不存在的能力时发送错误事件。
- Orchestrator 本身不调用 LLM 做意图分类，它负责确定性路由、创建 Turn 级 `StreamBus`、调用 `capability.run(context, bus)`、捕获异常并发送 `DONE`。
- Capability 是某类任务能力的实现，如 Chat、Deep Solve、Deep Research；Agent Loop 是 Capability 内部反复执行“LLM -> 工具 -> 工具结果 -> LLM”的控制循环。
- 工具是否可见由规则控制，包括用户开关、管理员白名单、KB/Memory/Skills 是否存在以及 Capability 自有工具；工具挂载后是否调用才由 LLM 结合语义决定。
- 因此 DeepTutor 采用“顶层显式规则路由 + Capability 内部 LLM 决策 + 工程规则护栏”的混合方式。

## StreamBus 与 StreamEvent

- `StreamEvent` 定义“传什么”，包含事件类型、来源、阶段、正文、元数据、Session、Turn、序号和时间戳。
- `StreamBus` 负责“怎么传”，是单个 Turn 内的进程内异步发布订阅通道，维护订阅者队列、事件历史、历史重放和关闭信号。
- Capability、Agent Loop、RAG 和工具执行器是事件生产者；Orchestrator、TurnRuntimeManager、WebSocket、CLI 和 SDK 是事件消费者。
- `StreamBus.emit(event)` 会将事件写入内存历史并广播到所有订阅队列；晚加入的订阅者先重放历史，再接收实时事件。
- Runtime 在消费事件后补充 `session_id`、`turn_id` 和递增 `seq`，再写入 `turn_events` 并推送给前端，因此断线重连依赖的是持久化事件和序号，而不只是 Bus 的内存历史。
- `DONE` 是表示 Turn 终态的业务事件，`close()` 是结束订阅通道的通信动作；正常收尾需要先发 `DONE`，再关闭 Bus。
- StreamBus 是进程内对象，不是 Kafka、Redis Stream 一类跨进程消息中间件；服务重启后的恢复由 Runtime 和 Store 负责。

## Turn 持久化收尾与资源释放

- Agent/Capability 结束后，Runtime 先从事件中聚合真正的 answer content，写入 assistant message，再解析并更新 Turn 的 `completed`、`failed` 或 `cancelled` 状态。
- `DONE` 与此前事件进入同一份可重放日志，并携带持久化后的 user/assistant message ID，供前端把乐观临时 ID 精确替换成数据库 ID。
- Turn 事件优先批量写入 Store，减少慢磁盘或 NAS 上逐事件提交的 fsync 成本；随后镜像到任务工作区的 `events.jsonl`。
- 新会话首次成功后，标题生成发生在 assistant message 和 `DONE` 之后，属于 post-turn metadata；失败或超时时回退为截断后的第一条用户消息。
- `finally` 首先移除 `reply_queue`，使迟到的 `submit_user_reply` 返回失败，而不是向已经结束的 Turn 堆积数据。
- Mastery Turn 根据 `turn_id` 释放实际持有的 path lease，避免中途切换学习路径后释放错租约。
- Runtime 向所有订阅队列写入结束哨兵 `None`，再从 `_executions` 删除内存执行对象，从而释放 task、事件缓冲和订阅者引用。
- `schedule_memory_reclaim()` 不是长期 Memory 的整理任务，而是进程内存回收：先 `await asyncio.sleep(0)` 等当前协程释放局部引用，再通过 `asyncio.to_thread()` 执行 `gc.collect()`。
- Linux/glibc 环境还会尝试 `malloc_trim(0)`，把 Python 已释放但分配器仍保留的空闲堆页归还操作系统；其它平台只执行可移植的 GC。
- 同一 Event Loop 上已有回收任务运行时会复用该任务，避免多个重型 Turn 同时触发重复 GC；整个回收流程是 best-effort，不阻塞用户可见的流式收尾。

## TurnRuntime 的两类 Queue

- 2026-09-21 对照本地 `deeptutor/services/session/turn_runtime.py`：这里有事件订阅队列与用户回复队列，不能把二者当成同一条通道。
- `subscribe_turn()` 为每个订阅者建立独立队列；`_publish_live_event()` 向各队列 `put_nowait(payload)`，订阅协程 `await queue.get()` 后向外输出事件，收到 `None` 则结束。
- `_run_turn()` 为 Turn 创建 `reply_queue`，登记在 `_reply_queues[turn_id]`；`ask_user` 等待回调通过 `await reply_queue.get()` 接收用户回复，`submit_user_reply()` 负责投递。
- 事件队列方向是“执行过程 → 订阅端”；回复队列方向是“用户输入 → 正在等待的 Turn”。后者让同一 Turn 的 Agent Loop 暂停并恢复，而不是另起一个普通聊天 Turn。
- 队列只负责进程内交接，不承担断线重放或跨进程恢复；已持久化事件的重放由 Session Store 等机制负责。
- 队列的 FIFO、背压和哨兵用法见 [asyncio.Queue](../../../python/asyncio-coroutines-and-generators.md#asyncioqueue协程间的生产者与消费者)。

## Partner 运行时复用与资源隔离

- 来源：本节最初依据 [Partner 运行时复用与隔离复习稿](../../../../interview/meta/source/deeptutor-partner-runtime-reuse-and-isolation.md) 整理；2026-09-21 补查本地 `services/partners/{scope,runtime,workspace}.py` 和 `partners/config/paths.py`，核对 Workspace 与作用域路径，未运行隔离测试。
- 材料把共享执行链写作 `TurnEngine -> AgenticChatPipeline -> AgentLoop`；本文早期记录使用 `TurnRuntimeManager -> ChatOrchestrator -> Capability`。两者的版本对应与调用关系待源码核实，不能直接认定为同名替换或串成一条已验证链路。
- 共享的是 Agent Loop、工具分发、Streaming、Prompt Assembly 和 Context Budget 等执行机制；各 Partner 通过身份、Soul、资源和权限配置产生差异，避免多份引擎分叉维护。
- `UnifiedContext` 承载当前 Turn 的运行上下文；Synthetic User Scope 决定底层资源解析属于哪个 Partner。`partner:<partner_id>` 是理解作用域的概念示例，不是已核实的真实编码格式。
- Partner Workspace 复用普通用户工作区的布局约定，使 RAG、Skill 和 Notebook 等已有工具能够经 PathService / Resource Resolver 定位当前 Partner 的资源。
- Workspace 确实对应文件系统中的独立目录树：默认 Partner 根为 `data/partners/<partner_id>/workspace/`。这里的 `data` 是默认布局，实际位置由管理员 workspace root 决定。
- 它在程序中是 PathService 的资源作用域根；进入 `user_context(partner_user(...))` 后，工具经 `get_current_path_service()` 解析到 Partner 资源，不等于执行一次 `os.chdir()`。
- 根下可承载 `knowledge_bases/`、`user/workspace/SOUL.md`、技能、笔记和设置等用户工作区布局；Memory 路径以 PathService 为准，不能仅凭注释把它拼成 `user/workspace/memory/`。
- Partner 级资源目录与关系目录不同：当前用户的关系工作区是 `data/partners/<partner_id>/users/<user_id>/workspace/`，关系记忆位于其 `memory/` 下；会话文件位于对应的 `sessions/` 目录，而不是必须塞进 workspace。
- Workspace 的独立目录只是隔离机制的一部分；安全边界还要求服务端校验 Scope、资源归属与访问权限，不能由模型提交的路径或身份决定授权。
- `SOUL.md` 描述人格和行为偏好；完整 Partner 还包括 KB、Skill、Notebook、Memory 和 Tool Policy，修改 Persona 不会自动改变资源权限。

## Partner 的每 Turn 工具策略

- 按材料，每次 Turn 在进入 Agent Loop 前由 `UnifiedContext -> ToolMountFlags -> compose_enabled_tools()` 生成最终工具集合，把 Policy 决策集中在组装边界。

| 策略 | 作用 | 边界 |
|---|---|---|
| `builtin_whitelist` | 限制普通 Built-in 自动挂载候选 | 只做筛选，不强制开启；有 KB 且满足挂载条件才会出现 RAG |
| `forced` | 补入 Partner 专属基础工具 | 材料中的 `partner_read`、`partner_memorize`、`partner_search` 不依赖普通 Context Gate 或 Built-in 白名单 |
| `suppressed` | 最终硬拒绝 | 无论工具从哪条路径进入都移除，包括与 `forced` 冲突时 |

- 三者的概念顺序是“普通候选经白名单筛选，再合入 forced，最后减去 suppressed”；这只概括这三项策略，不替代管理员授权、Provider Grant 等其它检查。
- 因此“Whitelist 是最多能有什么”只适用于它负责的普通 Built-in 候选，不能误讲成整个最终工具集合的绝对上限。
- Partner 的普通 `read_memory` / `write_memory` 被 suppressed，改用 Partner 专属工具遵循 [Split Memory 边界](./learning-memory-system.md#partner-的-split-memory-边界)。
- Tool Surface 决定“能不能执行这个操作”，Resource Scope 决定“能对谁的数据执行”；允许 `partner_memorize` 不等于允许写任意用户的 Memory。
- 隐藏工具 Schema 不能单独证明越权调用会被拦截；执行入口也应按本 Turn 的授权集合校验，拒绝模型伪造的工具名，并验证没有发生实际调用或副作用。

## 隔离验证与证据边界

- 按 Partner 和 User 两个维度构造矩阵：A 读 B 的 KB / Skill / Notebook、A × User1 读 A × User2 或 B × User1 的关系记忆；同时保留本 Scope 合法读写的正向用例。
- 为各私有 Scope 放置唯一 Canary Token，覆盖直接查询、搜索、Prompt Injection 和伪造 Tool Call；检查 Tool Result、Runtime Context 和最终回答，非授权数据进入任一层都应判失败。
- 对写操作比较 Memory Before / After，断言目标关系记忆发生预期变化、其它 Scope 和用户 Shared L3 保持不变。
- 工具测试同时检查 Final Enabled Tools 与真实分发入口：覆盖 suppressed 与 forced 冲突、白名单外普通 Built-in、无 KB 时 RAG 不自动挂载、Product Chat 调用 Partner 专属工具。
- 授权读取当前用户 Shared L3 属于正常共享，不算泄漏；模型最终拒答也不能抵消此前 Tool Result 已经泄漏的事实。
- Canary 命中能证明对应测试失败，未命中不能证明所有形式的数据泄漏都不存在；量化结果必须限定在有报告的测试集内。
- 通用测试方法见 [多租户隔离题集](../../../../interview/agents/agent-multi-tenancy.md)，实际样本量、0% / 100% 结果与个人贡献统一在 [简历指标口径](../../../../interview/resume/resume-metrics.md#跨作用域泄漏-0-测了多少场景) 维护。

# 相关链接

- [2026-09-21 学习日志](../../../../daily/2026-09-21.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](./chat-agent-loop-and-tool-protocol.md)
- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](./learning-memory-system.md)
- [DeepTutor：知识库与 RAG Provider 架构](./knowledge-base-and-rag-providers.md)
- [Python asyncio、协程与异步生成器](../../../python/asyncio-coroutines-and-generators.md)
- [Agent Runtime 与 Agent Loop 面试题](../../../../interview/agents/agent-runtime.md)
- [Human-in-the-loop 与暂停恢复面试题](../../../../interview/agents/human-in-the-loop.md)
- [KindPartner 项目答辩](../../../../interview/resume/kindpartner.md)
- [Partner、Workspace 与多租户隔离](../../../../interview/agents/agent-multi-tenancy.md)
- [Function Calling 与 Tool System](../../../../interview/agents/function-calling-and-tools.md)
- [简历指标与证据口径](../../../../interview/resume/resume-metrics.md)
- [Partner 运行时复用与隔离复习稿（归档）](../../../../interview/meta/source/deeptutor-partner-runtime-reuse-and-isolation.md)
- [2026-09-05 学习日志](../../../../daily/2026-09-05.md)
- [2026-08-23 学习日志](../../../../daily/2026-08-23.md)
