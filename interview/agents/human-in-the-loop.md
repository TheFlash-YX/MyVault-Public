---
type: interview
scope: topic
area: agents
topics:
  - human-in-the-loop
  - pause-resume
  - asyncio
  - distributed-state
created: 2026-08-27
updated: 2026-09-02
---

# Human-in-the-loop 与暂停恢复

- 原始题库只提供问题；回答框架只定义组织方式，不据此推断项目事实或指标证据。
- 分布式队列与中间件取舍可参考：[不引入 Redis/MQ 的架构取舍](../../notes/backend/architecture/cache-and-async-task-alternatives.md)。
- 速记主链：`LLM -> ask_user -> ToolResult.pause_for_user -> DispatchOutcome.pause -> AgentLoop -> Runtime waiter -> asyncio.Queue -> 用户回复 -> tool_call_id 回填 role=tool -> continue`。
- 边界口径：同 Turn pause/resume 保留 live coroutine、messages 和 Tool Calling 协议链；它适合在线同进程交互，不等价于 Temporal 一类 Durable Workflow。

## 普通 Agent 遇到信息不足时一般怎么处理？

<!-- canonical_key: agents.hitl.pu-tong-agent-yu-dao-xin-xi-bu-zu-shi-yi-ban-zen-me-chu-li -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 普通聊天式处理是直接追问并结束当前 Turn，用户下一条消息再触发新 Turn；这种做法主要依赖自然语言历史恢复上下文。
- Agent 场景更适合把澄清建模成 Human-in-the-loop pause/resume：Tool 层声明需要人输入，Runtime 暂停执行，用户回复后继续原任务。
- 回答时按“为什么要问 -> 怎么暂停 -> 怎么等待 -> 怎么恢复 -> 有什么边界”展开。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 什么叫 Turn？

<!-- canonical_key: agents.hitl.shen-me-jiao-turn -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- Turn 是一次用户提交对应的执行实例，从启动开始，到完成、失败、取消或暂停/恢复后完成为止。
- 一个 Turn 内可以包含多轮 LLM 调用、多个 Tool Call 和多批 Tool Result；不要把 Turn 等同于一次模型调用。
- 同 Turn 澄清的关键是用户补充信息回到原执行实例，而不是开启一个新的普通对话轮次。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 为什么重新创建 Turn 会丢失调用上下文？

<!-- canonical_key: agents.hitl.wei-shen-me-zhong-xin-chuang-jian-turn-hui-diu-shi-diao-yong-shang-xia-wen -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 新 Turn 可以重放聊天文本，但原来的 live AgentLoop state、工具调用批次、Capability 临时状态和 `tool_call_id` 关联都需要重新恢复。
- 如果 pause 前已经执行过带副作用的 Tool，新 Turn 还要判断哪些 Tool 已执行、哪些状态已落库、下一步原计划是什么。
- 所以问题会从简单的 pause/resume 升级成 Workflow Recovery 或 Checkpoint Recovery，复杂度明显更高。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 是阻塞线程吗？

<!-- canonical_key: agents.hitl.shi-zu-sai-xian-cheng-ma -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 不是阻塞线程，也不是轮询数据库；Agent coroutine 在 `await queue.get()` 处挂起。
- 等待期间 Event Loop 可以继续服务其它连接，系统不会 CPU 空转，也不会继续调用 LLM 消耗 token。
- 需要补一句边界：被挂起的是当前 Turn 的执行协程和相关内存状态，所以资源释放、超时和取消仍要由 Runtime 管。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 为什么使用异步队列？

<!-- canonical_key: agents.hitl.wei-shen-me-shi-yong-yi-bu-dui-lie -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 这个场景本质是生产者-消费者模型：AgentLoop/TurnRuntime 是消费者，用户通过 WebSocket 或 API 提交回复是生产者。
- `asyncio.Queue` 让消费者用 `await queue.get()` 挂起，生产者 `queue.put(reply)` 后唤醒原 coroutine。
- 它适合在线同进程、低延迟的 Human-in-the-loop；跨进程、多实例或重启恢复则需要额外持久化状态或外部队列。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## Turn 级异步队列的数据结构是什么？

<!-- canonical_key: agents.hitl.turn-ji-yi-bu-dui-lie-de-shu-ju-jie-gou-shi-shen-me -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 可以理解为 Runtime 内部的 `turn_id -> asyncio.Queue` 映射，每个等待用户输入的 live Turn 绑定一个队列。
- 队列里放的是用户对当前暂停点的结构化回复，而不是完整聊天历史。
- 用户回复后，Runtime 先用 `turn_id` 找到队列，再由 AgentLoop 用 `pause_tool_call_id` 找到对应 Tool Call。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 每个 Turn 都维护一个 Queue 吗？

<!-- canonical_key: agents.hitl.mei-ge-turn-dou-wei-hu-yi-ge-queue-ma -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 不是所有历史 Turn 都永久维护 Queue；通常是 Turn 进入等待用户输入阶段时才创建或登记对应队列。
- Turn 完成、失败、取消或收尾时必须移除 Queue，避免迟到回复写进已经结束的执行现场。
- 这个设计依赖 live execution，适合在线等待；如果要跨重启保留等待状态，需要把 pending reply 状态持久化。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## Queue 的生命周期怎么管理？

<!-- canonical_key: agents.hitl.queue-de-sheng-ming-zhou-qi-zen-me-guan-li -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 创建发生在 Runtime 启动/登记 Turn 的生命周期里，进入 pause 时把当前 Turn 的 waiter 暴露给 AgentLoop。
- 消费发生在 AgentLoop 等待恢复的位置，本质是 `await queue.get()`。
- 销毁发生在 Turn 的 `finally` 收尾路径：无论完成、失败还是取消，都清理 queue 和 execution 引用。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 用户回复后，你怎么找到对应的 Turn？

<!-- canonical_key: agents.hitl.yong-hu-hui-fu-hou-ni-zen-me-zhao-dao-dui-ying-de-turn -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 两级定位：`turn_id` 先定位“哪一个正在等待的 Turn”，再用该 Turn 绑定的 Queue 投递回复。
- Turn 内部还要保存 `pause_tool_call_id`，因为同一个 Turn 里可能先后出现多个工具调用。
- 所以 `turn_id` 解决会话/执行实例归属，`tool_call_id` 解决 Tool Calling 协议里的精确归属。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 如果用户同时打开两个对话，会不会回复错 Turn？

<!-- canonical_key: agents.hitl.ru-guo-yong-hu-tong-shi-da-kai-liang-ge-dui-hua-hui-bu-hui-hui-fu-cuo-turn -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 不应该只靠前端当前页面状态判断归属；提交回复时要携带能定位 pending Turn 的标识。
- 后端先用 `turn_id` 找到对应 Queue，再用 `pause_tool_call_id` 保证回复写回当前 Turn 内正确的 `ask_user` Tool Call。
- 如果是同一用户多个会话或多个标签页，Session/Turn 边界和 awaiting 状态校验必须同时成立。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 如果同一个 Turn 中连续调用两次 ask_user 怎么办？

<!-- canonical_key: agents.hitl.ru-guo-tong-yi-ge-turn-zhong-lian-xu-diao-yong-liang-ci-ask-user-zen-me-ban -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 不应该把多个 `ask_user` 当作多个独立 pending 状态并行等待；Runtime 一次只应维护一个明确的待回复问题卡。
- 如果需要问多个澄清点，应合并到一次 `ask_user(questions=[...])` 调用里。
- Dispatcher 可以把同一批 Tool Call 中重复的 `ask_user` 去重或只保留第一个，避免恢复时不知道该补哪个 Tool Result。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 如果用户一直不回复怎么办？

<!-- canonical_key: agents.hitl.ru-guo-yong-hu-yi-zhi-bu-hui-fu-zen-me-ban -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 当前机制面向在线 live Turn：用户不回复时，Agent coroutine 保持等待，既不继续执行也不结束原任务。
- 工程上要由 Runtime 定义 timeout、取消、资源释放和迟到回复策略。
- 面试时要主动说明它不是完整 Durable Workflow，不能把内存 Queue 等同于跨进程可恢复的工作流状态。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 是否有 timeout？

<!-- canonical_key: agents.hitl.shi-fou-you-timeout -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- timeout 属于 Runtime 生命周期策略，不应该写进 `AskUserTool` 本身；Tool 只声明 `pause_for_user`。
- 超时后可把 Turn 标记为 cancelled/failed，清理 Queue，并向前端发终态事件。
- 如果用户超时后才回复，Runtime 应拒绝或转成新的普通消息，不能再写入已结束 Turn 的 Queue。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## timeout 以后 Agent 状态怎么处理？

<!-- canonical_key: agents.hitl.timeout-yi-hou-agent-zhuang-tai-zen-me-chu-li -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 先说明何时需要人工介入，再沿暂停、等待、恢复、终态链路回答，并覆盖并发、重启和幂等边界。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 用户关闭网页后怎么办？

<!-- canonical_key: agents.hitl.yong-hu-guan-bi-wang-ye-hou-zen-me-ban -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 如果只是页面关闭但服务端进程和 Turn 仍在，等待协程可以继续挂起；前端重连后依赖事件日志或 Turn 状态恢复 UI。
- 如果关闭导致用户不再回复，则按 timeout/取消策略处理。
- 关键区分前端连接状态和 Runtime 执行状态：SSE/WebSocket 断开不必然等于 Turn 已结束。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 服务重启以后挂起的 Agent 怎么恢复？

<!-- canonical_key: agents.hitl.fu-wu-zhong-qi-yi-hou-gua-qi-de-agent-zen-me-hui-fu -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 如果实现只依赖 live coroutine 和内存 Queue，服务重启后原 `await queue.get()` 执行现场不能自动恢复。
- 可以把持久化里的 running/paused Turn 标记为 failed/cancelled，或者设计 checkpoint + pending tool call 重建机制。
- 面试回答要说清：当前在线方案不是 Durable Workflow；跨重启恢复需要额外工程。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 当前实现是内存队列还是持久化队列？

<!-- canonical_key: agents.hitl.dang-qian-shi-xian-shi-nei-cun-dui-lie-hai-shi-chi-jiu-hua-dui-lie -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 当前 `asyncio.Queue` 是进程内队列，用来唤醒同进程内挂起的 Agent coroutine。
- 持久化日志可以记录 Turn、事件和消息，但不等价于把等待协程和 Queue 本身持久化。
- 选型口径是：单进程在线交互用内存队列简单直接；多实例和跨重启恢复需要外部协调或 Durable Workflow。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 如果是内存队列，多实例部署会不会出问题？

<!-- canonical_key: agents.hitl.ru-guo-shi-nei-cun-dui-lie-duo-shi-li-bu-shu-hui-bu-hui-chu-wen-ti -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 会有路由问题：用户回复如果打到另一个进程，该进程内没有原 Turn 的 Queue，无法唤醒原 coroutine。
- 短期可以用粘性会话或让同一 Turn 的启动、订阅和回复落到同一实例。
- 长期应引入共享状态、外部队列或 durable execution，把 pending ask_user 状态从单进程内存中抽出来。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 如果部署三个 FastAPI 实例，用户请求到了不同实例怎么办？

<!-- canonical_key: agents.hitl.ru-guo-bu-shu-san-ge-fastapi-shi-li-yong-hu-qing-qiu-dao-le-bu-tong-shi-li-zen-me-ban -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 三个实例下，如果没有粘性路由或共享执行状态，`submit_user_reply(turn_id)` 可能落到没有该 Queue 的实例。
- 现象通常是回复提交失败、等待中的 Turn 不被唤醒，或者前端误以为已经恢复。
- 解决方向是 sticky session、按 `turn_id` 路由、共享 pending 状态，或者改造成可重放的 durable workflow。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 如果升级成分布式系统，你准备怎么改？

<!-- canonical_key: agents.hitl.ru-guo-sheng-ji-cheng-fen-bu-shi-xi-tong-ni-zhun-bei-zen-me-gai -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 第一层先持久化 `paused` Turn、`pause_payload`、`pause_tool_call_id` 和必要的 messages/checkpoint。
- 第二层让用户回复写入共享存储或外部队列，再由持有执行权的 worker 消费；多实例下需要租约、幂等键和状态机校验。
- 如果要求跨重启继续原工作流，需要能从 checkpoint 重建 AgentLoop，而不只是重新开一个普通 Turn。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## Redis Stream、Kafka、RabbitMQ 和 asyncio.Queue，你会怎么选？

<!-- canonical_key: agents.hitl.redis-stream-kafka-rabbitmq-he-asyncio-queue-ni-hui-zen-me-xuan -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- `asyncio.Queue` 适合同进程在线等待，延迟低、实现简单，但不能跨进程或重启恢复。
- Redis Stream 更适合轻量持久化通知和消费组；RabbitMQ 更偏任务投递与确认；Kafka 更偏高吞吐事件日志。
- 对 `ask_user` 恢复通知，先看是否需要多实例、重放、顺序、消费确认和运维成本；不要为单机在线场景过早引入重型 MQ。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## Tool Result 是怎么重新插入原调用链的？

<!-- canonical_key: agents.hitl.tool-result-shi-zen-me-zhong-xin-cha-ru-yuan-diao-yong-lian-de -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 模型先产生 `assistant.tool_calls`，其中 `ask_user` 有自己的 `tool_call_id`。
- 用户回复到达后，不新增普通 `role=user`，而是把回复写成对应 `tool_call_id` 的 `role=tool` message。
- 下一轮 LLM 看到的是完整工具协议链：assistant 调用了 `ask_user`，tool 返回了用户补充信息。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 重新恢复之后，模型看到的 messages 是什么结构？

<!-- canonical_key: agents.hitl.zhong-xin-hui-fu-zhi-hou-mo-xing-kan-dao-de-messages-shi-shen-me-jie-gou -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- messages 结构仍是同一个 Turn 内持续增长的数组：前面有 user，随后是带 `tool_calls` 的 assistant message，再跟匹配 `tool_call_id` 的 tool message。
- 用户补充内容作为 tool result 出现，而不是一个新的普通 user message。
- 这样模型不需要靠语义猜测“2 小时”是在回答哪个问题，而是能通过协议结构看到精确关联。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 为什么可以保证不会重复执行之前的 Tool？

<!-- canonical_key: agents.hitl.wei-shen-me-ke-yi-bao-zheng-bu-hui-zhong-fu-zhi-xing-zhi-qian-de-tool -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 同 Turn 方案没有重新运行前面的 Loop，而是原 coroutine 从 `await queue.get()` 之后继续执行。
- 已执行 Tool 的结果已经在 messages 和 Runtime state 中，恢复动作只是补齐 pending 的 `role=tool` 结果。
- 这只能保证 Runtime 不主动重放之前的 Tool；如果恢复后模型再次要求同样工具，还要依赖 Dispatcher 的去重、幂等键或业务侧幂等控制。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## ask_user 有没有 race condition？

<!-- canonical_key: agents.hitl.ask-user-you-mei-you-race-condition -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 有，主要关注“回复早于 pending 状态登记”“重复提交同一回复”“取消/超时与回复同时发生”“多实例路由到错误 worker”。
- 处理思路是先把 pending 状态登记清楚，再向前端暴露问题；恢复时检查 Turn 状态、`pause_tool_call_id` 和幂等键。
- Turn 收尾时移除 Queue，迟到回复应失败或转入新消息路径，不能写入已结束的执行现场。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 用户在 Agent 挂起之前就提交补充信息会怎么样？

<!-- canonical_key: agents.hitl.yong-hu-zai-agent-gua-qi-zhi-qian-jiu-ti-jiao-bu-chong-xin-xi-hui-zen-me-yang -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 先说明何时需要人工介入，再沿暂停、等待、恢复、终态链路回答，并覆盖并发、重启和幂等边界。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 前端和后端如何同步“等待用户输入”的状态？

<!-- canonical_key: agents.hitl.qian-duan-he-hou-duan-ru-he-tong-bu-deng-dai-yong-hu-shu-ru-de-zhuang-tai -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 后端通过 `pause_for_user`/`DispatchOutcome.pause` 进入等待态，并把结构化问题作为事件发给前端。
- 前端展示待回复 UI，提交时携带 `turn_id` 或恢复端点所需标识；后端再由 `turn_id` 找到 pending Queue。
- 刷新页面后应依赖持久化 Turn 状态和事件重放恢复 UI，而不是只靠前端内存状态。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## ask_user 本质上是不是 Human-in-the-loop？

<!-- canonical_key: agents.hitl.ask-user-ben-zhi-shang-shi-bu-shi-human-in-the-loop -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 是，但不是“模型随口问一句”的普通多轮聊天，而是把真人输入纳入 Agent Runtime 控制协议。
- `AskUserTool` 负责声明需要人输入，Dispatcher 归一化成 pause，Runtime 负责真正等待和恢复。
- 价值在于真人交互插入工作流时不丢失 Tool Calling 链、Loop state 和 Capability 的 Turn 内状态。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 和 LangGraph interrupt / resume 机制相比有什么区别？

<!-- canonical_key: agents.hitl.he-langgraph-interrupt-resume-ji-zhi-xiang-bi-you-shen-me-qu-bie -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 先承认语义相近：都是在 Agent 工作流中断点等待真人输入，再恢复后续执行。
- 差异要落在实现层：当前方案是 DeepTutor 自己的 `ToolResult.pause_for_user + DispatchOutcome.pause + Runtime queue` 链路。
- 如果没有 durable checkpoint，它更偏 live in-memory pause/resume；LangGraph 一类框架通常会把 interrupt/resume 和图状态持久化能力结合得更系统。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 如果用户修改之前的问题，而不是回答澄清问题，怎么处理？

<!-- canonical_key: agents.hitl.ru-guo-yong-hu-xiu-gai-zhi-qian-de-wen-ti-er-bu-shi-hui-da-cheng-qing-wen-ti-zen-me-chu-li -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 需要把“回答当前澄清问题”和“编辑/重开原始问题”区分成不同入口。
- 如果是回答澄清问题，就走 `submit_user_reply(turn_id)` 并回填 `role=tool`。
- 如果用户修改原始问题，应取消或结束当前 pending Turn，再创建新 Turn，不能把新需求强行塞进旧 `tool_call_id`。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 挂起期间如何支持用户主动取消当前任务？

<!-- canonical_key: agents.hitl.gua-qi-qi-jian-ru-he-zhi-chi-yong-hu-zhu-dong-qu-xiao-dang-qian-ren-wu -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 取消由 Runtime 控制，不由 `AskUserTool` 自己处理；取消时设置 Turn 状态，取消等待 coroutine，并移除 Queue。
- 前端需要收到明确终态事件，避免仍展示待回复卡片。
- 迟到回复应被拒绝或转成普通新消息，避免恢复一个已经取消的执行现场。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 恢复时如何确认用户回复属于当前 ask_user，而不是普通新消息？

<!-- canonical_key: agents.hitl.hui-fu-shi-ru-he-que-ren-yong-hu-hui-fu-shu-yu-dang-qian-ask-user-er-bu-shi-pu-tong-xin-xiao-xi -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 外层靠 `turn_id` 确认回复投递给哪个 pending Turn，而不是进入普通新消息创建路径。
- 内层靠 `pause_tool_call_id` 把内容写回原 `ask_user` 调用对应的 `role=tool` message。
- 所以它不是靠模型从自然语言判断归属，而是靠 Runtime 状态和 Tool Calling 协议双重确认。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 如果用户重复提交同一条澄清回复，如何保证幂等？

<!-- canonical_key: agents.hitl.ru-guo-yong-hu-zhong-fu-ti-jiao-tong-yi-tiao-cheng-qing-hui-fu-ru-he-bao-zheng-mi-deng -->

### 考察点

- 人工介入协议、暂停恢复语义、并发边界、超时取消与分布式状态

### 回答框架

- 恢复接口应携带幂等键或至少校验 Turn 当前是否仍处于 awaiting_user_reply。
- 第一条有效回复写入 Queue 后，Turn 状态应从等待态切走；重复提交看到的应是已恢复/已结束，而不是再次 `queue.put`。
- 如果已经生成了对应 `role=tool` message，再次提交不能追加第二条同 `tool_call_id` 的结果。

### 易追问

-

### 关联笔记

- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)
