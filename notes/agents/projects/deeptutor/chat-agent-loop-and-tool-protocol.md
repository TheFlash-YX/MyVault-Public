---
type: Note
domain: agents
title: DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复
aliases: [DeepTutor Agent Loop, Chat AgentLoop, Tool Protocol, ask_user]
description: 梳理 DeepTutor Chat Agent Loop 从工具面组装、消息增长、LLM 流式轮次到并行工具、ask_user 暂停、checkpoint 和强制收尾的完整状态机
tags: [deeptutor, agent-loop, tool-calling, ask-user, context-checkpoint, streaming]
status: learning
created: 2026-08-23
updated: 2026-09-02
---

# 概述

- DeepTutor 的 Chat Agent Loop 在一个 Turn 内维护一份持续增长的 `messages`，重复执行“LLM 决策 -> 工具执行 -> `role=tool` 结果回填 -> 下一轮 LLM”。
- 一个 Turn 是外层执行单元，一个 Agent Loop 是 Turn 内的调度器，一个 Round 是一次 LLM 流式调用；一个 Turn 可以包含多个 Round 和多个工具批次。
- Chat Loop 的核心退出条件是“这一轮没有工具调用且正文完整”：有 `tool_calls` 就继续，没有 `tool_calls` 的完整轮次才是 finish。
- 模型负责决定下一步和生成内容，确定性代码负责工具授权、参数补全、协议配对、并发上限、暂停恢复、预算和退出条件。
- `ask_user` 的本质不是“让模型追问一句”，而是在同一个 Turn 中补齐 pending Tool Result，使真人输入进入 Tool Calling 协议链。
- Chat AgentLoop 与 `core/agentic/loop.py` 的 Label-driven Loop 是两套并列引擎，不能把后一套的 LabelProtocol 规则套到 Chat Loop 上。

# 详细内容

## Loop 启动前

- `AgenticChatPipeline.run()` 先准备 deferred provider tools、知识库 manifests 和 exec policy，得到本 Turn 实际可访问的资源。
- 工具面由用户开关、上下文自动挂载、always-on built-ins、Loop Capability 自有工具和 exclusive knowledge surface 合并，再经过 whitelist、grant 与 owner scope 过滤。
- 模型支持 Function Calling 时构造 OpenAI tools schema，并绑定 Turn 级 `ProviderToolView`；不支持时保留 schema catalog，供 DSML 文本工具协议解析参数。
- Capability 可以执行一次异步 `pre_loop` briefing；其结果与 KB seed、capability seed 一起注入当前 user message，失败时只降级而不击穿主循环。
- 初始消息顺序是 system blocks、压缩摘要/历史、当前用户输入与附件；system prompt 在 Loop 中保持稳定，后续只向尾部追加消息。
- `AgentLoopState` 记录总 Round、探索 Round、settlement Round、工具批次数和累计 sources；`checkpoint_boundary` 记录可折叠工具轨迹的边界。

## 单一增长消息协议

- 工具轮先追加一条带 `tool_calls` 的 assistant message，再为每个调用追加匹配 `tool_call_id` 的 `role=tool` message。
- 即使工具不存在、参数缺失、执行失败或被判定为重复，也必须生成对应 tool message，保持 OpenAI 工具调用协议闭合。
- 下一轮 LLM 读取同一数组中的 system、历史、assistant tool decision 和 tool results，因此不需要另外建立“工具结果通道”。
- Settlement、截断续写和协议修复指令也会写入 messages；若末尾已经是 user role，就合并到该消息，避免连续 user role。

## 一次 LLM Round

- 每轮调用前执行上下文窗口 Guard；超过有效窗口比例时，从较早的 `role=tool` 大结果开始替换为 snip marker，优先保留系统指令和对话语义。
- `_call_llm()` 发送 OpenAI-compatible streaming request，并为这一轮建立 call trace 和最后一次真实请求快照。
- Provider 原生的 `reasoning_content/reasoning` 直接转为 thinking event；正文中的 think 标签由 `InlineThinkFilter` 增量拆分。
- 普通正文实时发出 content event；native tool delta 按 index 累加 id、name 和 arguments，流结束后还会解析可能存在的 DSML 工具标记。
- Native 与 DSML 工具调用同时存在时优先 native，避免同一工具被双重分发；DSML 标记和参数不会进入用户可见正文。
- 每轮结束后记录 usage、finish_reason 和 `call_role`；有工具或输出被截断时是 narration，无工具的完整输出是 finish。
- 工具轮正文默认不进入最终持久化回答；截断输出、DSML 混合输出和 Mastery 教学轮可以通过 `answer_visible` 保留确实面向用户的正文。

## 工具分发

- Dispatcher 最多接受同轮 8 个工具调用，先解析 JSON arguments，再由服务端注入 session/turn、source index、sandbox mount、output directory 和 owner identity 等私有参数。
- 必填参数缺失会在 dispatch 前被拒绝并返回修复信息，避免错误深入工具内部后才暴露。
- 普通工具按“名称 + 规范化参数”去重；重复调用只执行第一次，其余返回 stub tool result。
- 同一模型消息中的多个 `ask_user` 即使参数不同也只保留第一次，因为 Runtime 同时只能绑定一个待回复问题卡。
- 普通工具通过 `asyncio.gather()` 并行执行；`ask_user` 属于 pause-last 工具，在本轮其它调用提交后再执行和重新补参。
- pause-last 的价值是让本轮普通工具先完成状态更新或持久化，`ask_user` 展示给用户的问题基于已经落地后的真实状态，而不是模型临时参数。
- 每个工具都有独立的 call ID、running/complete/error sub-trace；单个工具抛错通常转换成失败 ToolResult 反馈模型，不直接让整个 Turn 失败。
- `ToolResult` 同时承载数据结果、执行状态和控制信号；普通工具主要返回 content/sources/metadata，`ask_user` 额外通过 `pause_for_user` 发出暂停意图。
- Dispatcher 最终按原调用顺序生成 tool messages，并汇总 sources、tool metadata、`pause`、`pause_tool_call_id`、`terminate` 和 context checkpoint 信号。

## ask_user 暂停与恢复

- `AskUserTool` 不直接执行终端或网页 I/O，而是返回包含结构化问题的 `ToolResult.pause_for_user`。
- `pause_for_user` 不等于 `terminate_turn`：前者表示任务未完成、等待真人补信息后继续；后者表示工具结果可以直接结束当前 Turn。
- Agent Loop 发现 pause 后调用 `wait_for_user_reply()`，当前协程停在 `await reply_queue.get()`；它不忙等 CPU，也不结束当前 Turn。
- Web 通过 WebSocket、CLI 通过终端渲染同一份问题结构，提交后最终都调用 `submit_user_reply(turn_id)` 向该 Turn 的 `asyncio.Queue` 写入答案。
- 回复到达后，原协程从同一个调用栈恢复，执行 Capability 的 `on_user_resume` hook，并把答案写入匹配 `tool_call_id` 的 tool message。
- 下一轮 LLM 看到的是“assistant 调用 ask_user + tool 返回 User answered”，不是一个新普通 user message，也不会创建新 Turn。
- `on_user_pause/on_user_resume` 让 Mastery 等 Capability 在交互边界上提交自己的持久状态；resume hook 还可以设置 `END_LOOP`，直接接管最终内容而不再调用 LLM。
- ask_user 当前不使用 `StreamBus.wait_for_input()`；真实等待链路是 `pause_for_user + TurnRuntimeManager reply queue + submit_user_reply`。
- `turn_id` 定位哪个 Turn 的 Queue，`pause_tool_call_id` 定位这一 Turn 内哪个 Tool Call；两者分别解决执行实例归属和协议级结果归属。
- Reply queue 是进程内状态，不能跨服务重启恢复；取消 Turn 会取消等待协程，Turn 收尾时移除 queue，使迟到回复失败。
- 这套方案保留的是 live coroutine、同一份 messages、AgentLoop state 和 Capability Turn 内状态；如果结束后新建 Turn，就需要额外恢复已执行工具、副作用和下一步计划。

## Web Chat 与 Partner IM 边界

- Web Chat 的 Runtime 可以保持同一个 live Turn，因此能做到真正的同 Turn pause/resume。
- Partner IM 的消息语义通常是当前 Turn 把澄清问题发出去，用户下一条 IM 再开启新 Turn；它更像异步消息接力，不应笼统说所有 `ask_user` 场景都是同 Turn 恢复。
- 因此面试或设计文档中要区分“Web Runtime 支持同 Turn pause/resume”和“外部 IM 渠道按消息边界重新进入系统”。

## Context Checkpoint

- 工具可以在 metadata 中返回 `_context_checkpoint.summary`，用于说明一个阶段已经完成并提供后续所需摘要。
- Loop 取本批最后一个有效 checkpoint，保留 `checkpoint_boundary` 之前的稳定前缀，再追加 `[Context checkpoint]` system message。
- Checkpoint 之后的 assistant/tool 细节会被删除，新 system summary 成为下一阶段的稳定边界。
- Solve 使用该机制折叠已经完成的步骤，避免多轮工具链不断膨胀上下文；它不同于 Session 级历史摘要，作用域只在当前 Agent Loop。

## 退出、预算与容错

- 正常 finish：某轮没有 tool calls、正文非空且没有被 token 上限截断，清理 thinking 后作为 `final_text`。
- Terminator tool：工具返回 `terminate_turn=true`，工具 content 直接成为最终产物；当前内置 Chat 工具通常不使用该路径。
- 未恢复 pause：没有 waiter、回复为 `None` 或 Capability 设置 `END_LOOP` 时停止 Chat Loop，不伪造下一轮模型回答。
- Exploration Round 达到配置上限后进入 settlement，最多保留 3 轮带工具收尾；仍未结束时增加一次禁用工具的 forced finish，因此理论上限为 exploration + 4。
- `finish_reason=length/max_tokens` 不视为正常完成：保留可见前缀，将 raw assistant 输出与续写指令放回 messages，再进入下一轮。
- 模型只产生 thinking 而无工具或正文时，Loop 最多提醒一次“执行计划或直接回答”；最终仍为空则发本地 fallback 文案。
- Provider 流在任何输出前中断时按 0.5 秒、1.5 秒短重试；一旦 content、thinking 或 tool delta 已流出，就不盲目重放，避免重复文本和工具副作用。
- Provider 不支持 `stream_options` 时移除该参数；拒绝 native tools schema 时退化到 DSML；不支持图片且允许降级时去掉 image parts 重试。
- 中途 LLM 失败但此前 Round 已收集有效材料时，Loop 尝试 forced finish；首轮失败或已流出部分内容后的 transport 中断上抛 Runtime 处理。

## 输出与持久化边界

- Loop 结束后统一发送累计 sources，再通过 RESULT 输出 response、completed、engine、rounds、settlement_rounds、tool_steps、usage/cost 和 context budget。
- Agent Loop 只负责产生 StreamEvent 和最终 Outcome；`TurnRuntimeManager` 才负责从事件中提取 answer content、写 assistant message、更新 Turn 状态并持久化事件。
- 这层边界让 Web、CLI 和 SDK 共用同一 Agent Loop 与持久化语义，只替换输入输出适配器。

## 与 Label-driven Loop 的区别

- Chat AgentLoop 以“是否有 tool calls”作为核心控制信号；通用 Label-driven Loop 要求模型首行输出声明式 label。
- `LabelProtocol` 定义 terminal、intermediate、final 和 tool label，并校验缺失/重复 label、label 与工具调用不一致等协议错误。
- 通用循环通过 `LoopHost` 回调注入 trace、工具分发、pause、terminal validation 和 force finalize，主要被 deep_question、deep_research 等多阶段能力复用。
- 两套引擎共享 StreamBus、工具 Dispatcher 和部分消息协议，但不是继承关系，也不应把通用 Label 修复规则描述成 Chat AgentLoop 的退出条件。

# 相关链接

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](./agent-runtime-and-turn-lifecycle.md)
- [DeepTutor：知识库与 RAG Provider 架构](./knowledge-base-and-rag-providers.md)
- [Python asyncio、协程与异步生成器](../../../python/asyncio-coroutines-and-generators.md)
- [Agent Runtime 与 Agent Loop 面试题](../../../../interview/agents/agent-runtime.md)
- [Human-in-the-loop 与暂停恢复面试题](../../../../interview/agents/human-in-the-loop.md)
- [KindPartner 项目答辩](../../../../interview/resume/kindpartner.md)
- [2026-08-23 学习日志](../../../../daily/2026-08-23.md)
