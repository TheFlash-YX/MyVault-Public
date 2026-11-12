---
type: interview
scope: resume
area: resume
project: kindpartner
resume_version: 2026-08
topics:
  - agent-runtime
  - human-in-the-loop
  - asyncio
  - memory
  - rag
created: 2026-08-27
updated: 2026-09-07
---

# KindPartner 项目答辩

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。
- 关联知识与题集：
- [Agent Runtime 与 Agent Loop](../agents/agent-runtime.md)
- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [Agent Memory](../agents/memory.md)
- [RAG Frameworks 与 Provider](../agents/rag-frameworks-and-providers.md)
- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)
- [DeepTutor：知识库与 RAG Provider 架构](../../notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)
- ask_user 项目口径：Web Chat Runtime 支持同 Turn pause/resume；Partner IM 更接近“当前 Turn 发出澄清问题，用户下一条 IM 进入新 Turn”，不要把两种渠道混说。
- Memory 项目口径：长期记忆按 L1/L2/L3 分层，强调可查看、可编辑、可删除、可追溯和可回滚；不要把它说成单纯 Vector DB 召回。

## 介绍一下 KindPartner 整体的 Agent 架构。

<!-- canonical_key: resume.kindpartner.jie-shao-yi-xia-kindpartner-zheng-ti-de-agent-jia-gou -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 一个用户请求进入系统之后，完整执行链路是什么？

<!-- canonical_key: resume.kindpartner.yi-ge-yong-hu-qing-qiu-jin-ru-xi-tong-zhi-hou-wan-zheng-zhi-xing-lian-lu-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 你的 Agent Loop 是怎么实现的？

<!-- canonical_key: resume.kindpartner.ni-de-agent-loop-shi-zen-me-shi-xian-de -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## Agent Runtime 和业务层之间如何解耦？

<!-- canonical_key: resume.kindpartner.agent-runtime-he-ye-wu-ceng-zhi-jian-ru-he-jie-ou -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么设计 UnifiedContext？

<!-- canonical_key: resume.kindpartner.wei-shen-me-she-ji-unifiedcontext -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## UnifiedContext 里面包含哪些信息？

<!-- canonical_key: resume.kindpartner.unifiedcontext-li-mian-bao-han-na-xie-xin-xi -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## Agent 的 System Prompt 是怎么组织的？

<!-- canonical_key: resume.kindpartner.agent-de-system-prompt-shi-zen-me-zu-zhi-de -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 不同 Partner 如何共享 Runtime 又保证行为不同？

<!-- canonical_key: resume.kindpartner.bu-tong-partner-ru-he-gong-xiang-runtime-you-bao-zheng-xing-wei-bu-tong -->

### 考察点

- 运行时复用的项目取舍、个人贡献与实现证据

### 回答框架

- 按 [归档复习稿](../meta/source/deeptutor-partner-runtime-reuse-and-isolation.md) 组织项目表达：Partner 复用产品 Chat 的执行机制，差异由 UnifiedContext、Synthetic Scope 和独立 Workspace 注入。
- 用具体配置说明差异：Soul、KB、Skill、Notebook 与每 Turn 工具策略；记忆分为只读 Shared L3 和 Partner × User 可写关系记忆。
- 通用隔离原理见 [多租户题集](../agents/agent-multi-tenancy.md#如何保证-partner-a-读不到-partner-b-的数据)，工具策略见 [工具权限题](../agents/function-calling-and-tools.md#如何防止-partner-调用没有权限的工具)。
- 当前只有复习材料，需补对应提交、入口与个人负责模块；材料中的 TurnEngine 与旧笔记中的 TurnRuntimeManager 尚未核对版本关系。

### 易追问

- 共享 Runtime 时每个 Turn 的可变状态怎样隔离？Partner IM 与 Web 的 ask_user 行为是否一致？

### 关联笔记

- [DeepTutor Runtime 与 Partner 隔离](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor 学习记忆与 Split Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 为什么不直接为每个 Partner 单独创建一套 Agent？

<!-- canonical_key: resume.kindpartner.wei-shen-me-bu-zhi-jie-wei-mei-ge-partner-dan-du-chuang-jian-yi-tao-agent -->

### 考察点

- 运行时复用的项目取舍、个人贡献与实现证据

### 回答框架

- 材料中的取舍是复用 Agent Loop、工具调用和流式基础设施，将 Partner 差异放入 Context 与资源策略，减少多套引擎同步维护的成本。
- 区分“每个 Partner 有独立状态”与“复制一套执行引擎”；可复用代码仍需隔离工作区、权限和关系记忆。
- 结合实际改动解释自己参与的边界与复用点；未提供代码或测试报告前，不声称完成了特定规模的扩展验证。
- 通用组件边界见 [UnifiedContext 装配](../agents/agent-multi-tenancy.md#unifiedcontext-如何装配这些组件)，项目链路见 [Runtime 笔记](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md#partner-运行时复用与资源隔离)。

### 易追问

- 哪些变化只需配置，哪些必须新增 Capability？共享执行引擎与独立进程部署是否冲突？

### 关联笔记

- [DeepTutor Runtime 与 Partner 隔离](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor 学习记忆与 Split Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 如果未来有几百种 Partner，当前架构能扩展吗？

<!-- canonical_key: resume.kindpartner.ru-guo-wei-lai-you-ji-bai-zhong-partner-dang-qian-jia-gou-neng-kuo-zhan-ma -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 你的 Runtime 和 LangGraph 这类框架有什么区别？

<!-- canonical_key: resume.kindpartner.ni-de-runtime-he-langgraph-zhe-lei-kuang-jia-you-shen-me-qu-bie -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么没有直接使用 LangGraph？

<!-- canonical_key: resume.kindpartner.wei-shen-me-mei-you-zhi-jie-shi-yong-langgraph -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 如果现在让你重构，你会不会换成 LangGraph / OpenAI Agents SDK / Claude Agent SDK 一类框架？

<!-- canonical_key: resume.kindpartner.ru-guo-xian-zai-rang-ni-zhong-gou-ni-hui-bu-hui-huan-cheng-langgraph-openai-agents-sdk-claude-agent-sdk-yi-lei-kuang-jia -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么需要设计 ask_user 工具？

<!-- canonical_key: resume.kindpartner.wei-shen-me-xu-yao-she-ji-ask-user-gong-ju -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 背景：复杂学习计划、练习或检索任务中，Agent 经常缺少目标、时间、范围等关键约束，直接猜会降低任务质量。
- 选择：在 Web Chat Runtime 里把 `ask_user` 做成 pause/resume Tool，而不是让模型简单输出一句追问后结束 Turn。
- 实现：`AskUserTool` 只返回 `ToolResult.pause_for_user`，Dispatcher 汇总成 `DispatchOutcome.pause`，Runtime 通过 Turn 级 waiter 等用户回复。
- 边界：这个回答只覆盖已验证的 Web Runtime 机制；Partner IM 的消息语义不一定保持同 Turn。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 你这里的 ask_user 和“模型直接回复用户一个问题”有什么区别？

<!-- canonical_key: resume.kindpartner.ni-zhe-li-de-ask-user-he-mo-xing-zhi-jie-hui-fu-yong-hu-yi-ge-wen-ti-you-shen-me-qu-bie -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 模型直接回复问题，本质是普通聊天；用户下一条消息主要靠自然语言语义和历史上下文重新关联。
- `ask_user` 是 Tool Calling 协议的一部分：assistant 先产生 `ask_user` tool call，用户回复再作为对应 `tool_call_id` 的 `role=tool` 结果回填。
- 工程价值是关联更精确，AgentLoop state、Tool Result 和 Capability Turn 内状态不用靠新 Turn 重新恢复。
- 边界：它提升 Runtime 连续性，不保证模型一定语义理解正确。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 为什么一定要保持在同一个 Turn 中？

<!-- canonical_key: resume.kindpartner.wei-shen-me-yi-ding-yao-bao-chi-zai-tong-yi-ge-turn-zhong -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 因为新 Turn 虽然能重放聊天文本，但会丢失或需要重建原来的 live Agent execution state。
- 同 Turn 方案保留三件事：同一个 AgentLoop coroutine、同一份 messages、同一条 `assistant.tool_calls -> role=tool` 协议链。
- 用户补充信息只是在原 `tool_call_id` 上补齐 pending Tool Result，然后继续下一轮 LLM。
- 如果改成新 Turn，就要处理已执行 Tool、副作用、状态落库和下一步计划的恢复问题。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)

## ask_user 被调用之后，Agent Loop 发生了什么？

<!-- canonical_key: resume.kindpartner.ask-user-bei-diao-yong-zhi-hou-agent-loop-fa-sheng-le-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 执行链路可以按 `LLM -> ask_user -> ToolResult.pause_for_user -> DispatchOutcome.pause -> AgentLoop -> Runtime waiter -> asyncio.Queue -> 用户回复 -> role=tool 回填 -> continue` 讲。
- `AskUserTool` 不直接等人，它只声明暂停；真正等待由 Runtime 注入到 `UnifiedContext` 的 waiter 完成。
- 用户回复后，Runtime 通过 `turn_id` 找到等待中的 Turn，通过 `pause_tool_call_id` 找到原来的 Tool Call。
- 恢复后继续同一个 AgentLoop，而不是重新创建一次 Agent 执行。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## “挂起 Agent Loop”具体是怎么实现的？

<!-- canonical_key: resume.kindpartner.gua-qi-agent-loop-ju-ti-shi-zen-me-shi-xian-de -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- Runtime 为当前 Turn 准备 reply queue 和 waiter，AgentLoop 在发现 pause 后调用 waiter。
- waiter 内部等待 `asyncio.Queue.get()`；这是挂起当前 coroutine，不是阻塞线程，也不是轮询数据库。
- 用户通过 WebSocket 或 API 提交回复后，Runtime 把回复 `put` 到对应 Queue，原 coroutine 被唤醒。
- Turn 收尾时清理 Queue，迟到回复不能再恢复已经结束的执行现场。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 这对你的 ask_user 机制有什么影响？

<!-- canonical_key: resume.kindpartner.zhe-dui-ni-de-ask-user-ji-zhi-you-shen-me-ying-xiang -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 影响是需要明确区分“Web Chat 的同 Turn Runtime”和“Partner IM 的消息驱动入口”。
- Web Chat 可以把 `ask_user` 作为 live pause/resume，用户回答回填到原 Tool Call。
- Partner IM 里用户下一条消息天然是新的外部消息，更适合新 Turn 处理，再通过会话历史和业务状态衔接。
- 因此简历答辩里不要说所有入口都做到同 Turn 恢复，只说 Web Runtime 支持该机制。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## UnifiedContext 是你自己设计的吗？

<!-- canonical_key: resume.kindpartner.unifiedcontext-shi-ni-zi-ji-she-ji-de-ma -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么叫 UnifiedContext？

<!-- canonical_key: resume.kindpartner.wei-shen-me-jiao-unifiedcontext -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 代码里它具体是 class、dataclass 还是 dict？

<!-- canonical_key: resume.kindpartner.dai-ma-li-ta-ju-ti-shi-class-dataclass-hai-shi-dict -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 有哪些字段？

<!-- canonical_key: resume.kindpartner.you-na-xie-zi-duan -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 谁负责构造？

<!-- canonical_key: resume.kindpartner.shei-fu-ze-gou-zao -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 生命周期是什么？

<!-- canonical_key: resume.kindpartner.sheng-ming-zhou-qi-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 有没有序列化？

<!-- canonical_key: resume.kindpartner.you-mei-you-xu-lie-hua -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 有没有持久化？

<!-- canonical_key: resume.kindpartner.you-mei-you-chi-jiu-hua -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 多 Turn 之间是否复用？

<!-- canonical_key: resume.kindpartner.duo-turn-zhi-jian-shi-fou-fu-yong -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 并发请求是否线程安全？

<!-- canonical_key: resume.kindpartner.bing-fa-qing-qiu-shi-fou-xian-cheng-an-quan -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么不直接用 request context？

<!-- canonical_key: resume.kindpartner.wei-shen-me-bu-zhi-jie-yong-request-context -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 有没有循环依赖问题？

<!-- canonical_key: resume.kindpartner.you-mei-you-xun-huan-yi-lai-wen-ti -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 单元测试怎么写？

<!-- canonical_key: resume.kindpartner.dan-yuan-ce-shi-zen-me-xie -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 如果现在让我画类图，你能画出来吗？

<!-- canonical_key: resume.kindpartner.ru-guo-xian-zai-rang-wo-hua-lei-tu-ni-neng-hua-chu-lai-ma -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## ask_user 的 Queue 存在哪里？

<!-- canonical_key: resume.kindpartner.ask-user-de-queue-cun-zai-na-li -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- Queue 是 TurnRuntime 管理的进程内状态，可理解为 `turn_id -> asyncio.Queue`。
- 它不是数据库表，也不是 Redis/Kafka 这类外部持久化队列。
- 这个选择适合同进程在线 pause/resume；多实例或服务重启恢复需要额外的共享状态或 durable workflow 设计。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## ask_user 的 key 是什么？

<!-- canonical_key: resume.kindpartner.ask-user-de-key-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 外层 key 是 `turn_id`，用来找到哪一个 live Turn 的 Queue。
- 协议内 key 是 `tool_call_id`，更准确地说是 pause 时保存的 `pause_tool_call_id`，用来把用户回复写回哪个 `ask_user` Tool Result。
- 简洁口径：`turn_id` 定位 Turn，`tool_call_id` 定位 Turn 内具体 Tool Call。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 谁创建 Queue？

<!-- canonical_key: resume.kindpartner.shei-chuang-jian-queue -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- TurnRuntime 创建或登记 Queue，并把 waiter 通过 `UnifiedContext.metadata` 注入给 Agent Pipeline。
- 这样 Agent Pipeline 不需要直接依赖 WebSocket、Session 或 Runtime 私有对象。
- 这属于轻量依赖注入：Runtime 提供等待能力，AgentLoop 只消费 pause/resume 抽象。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)

## 谁消费 Queue？

<!-- canonical_key: resume.kindpartner.shei-xiao-fei-queue -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 消费者是等待恢复的 AgentLoop/Agent Pipeline coroutine。
- 它在 Runtime 注入的 waiter 中执行 `await queue.get()`，用户没回复时挂起，回复到达后从同一调用栈继续。
- 因为恢复的是原 coroutine，所以不用重放前面已经执行过的 Tool。

### 易追问

-

### 关联笔记

- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)
- [Python asyncio、协程与异步生成器](../../notes/python/asyncio-coroutines-and-generators.md)

## 谁销毁 Queue？

<!-- canonical_key: resume.kindpartner.shei-xiao-hui-queue -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- TurnRuntime 在 Turn 的完成、失败、取消或 finally 收尾路径中销毁 Queue。
- 销毁后迟到的 `submit_user_reply` 应失败或走普通新消息路径，不能继续唤醒已结束的 Turn。
- 这个清理动作也防止长时间等待导致内存中的 execution、task 和订阅引用泄漏。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [Human-in-the-loop 与暂停恢复](../agents/human-in-the-loop.md)

## timeout 谁控制？

<!-- canonical_key: resume.kindpartner.timeout-shei-kong-zhi -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## cancel 怎么处理？

<!-- canonical_key: resume.kindpartner.cancel-zen-me-chu-li -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## SSE disconnect 怎么处理？

<!-- canonical_key: resume.kindpartner.sse-disconnect-zen-me-chu-li -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## server restart 怎么处理？

<!-- canonical_key: resume.kindpartner.server-restart-zen-me-chu-li -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## multiple worker 怎么处理？

<!-- canonical_key: resume.kindpartner.multiple-worker-zen-me-chu-li -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么不用 Redis？

<!-- canonical_key: resume.kindpartner.wei-shen-me-bu-yong-redis -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 线上真的部署过多 worker 吗？

<!-- canonical_key: resume.kindpartner.xian-shang-zhen-de-bu-shu-guo-duo-worker-ma -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## RAGProvider 是 interface、protocol 还是抽象类？

<!-- canonical_key: resume.kindpartner.ragprovider-shi-interface-protocol-hai-shi-chou-xiang-lei -->

### 考察点

- 既有框架与个人接入工作的边界

### 回答框架

- 按本次复习材料，项目已有 RAGPipeline / RAGService，个人工作集中在 LightRAG Provider 接入，不能将既有框架描述成自己从零设计。
- RAGPipeline 的 Python 声明究竟是 Protocol、ABC 还是普通基类，材料未提供源码；面试前需定位真实声明和调用方。
- 抽象职责见 [Provider 题集](../agents/rag-frameworks-and-providers.md)，项目证据应补具体提交、类定义和本人修改范围。

### 易追问

- 协议约定与实际继承关系是否相同？哪些组件在接入前已存在？

### 关联笔记

- [DeepTutor RAG Provider 与 LightRAG](../../notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md)

## 新增一个 Provider 具体需要实现哪些方法？

<!-- canonical_key: resume.kindpartner.xin-zeng-yi-ge-provider-ju-ti-xu-yao-shi-xian-na-xie-fang-fa -->

### 考察点

- LightRAG 接入职责与实现证据

### 回答框架

- 按 [复习稿](../meta/source/kindpartner-lightrag-knowledge-retrieval.md)，LightRagPipeline 复用统一 Parse Layer，适配索引、查询模式、LLM / Embedding 客户端，并将 Source 回填到工具结果。
- 既有笔记将协议方法记录为 initialize、add_documents、search、delete；以对应版本签名为准，逐项说明自己实现或复用的部分。
- 材料记录项目 hybrid 模式和本地 NanoVectorDB，实际配置需核对源码；格式适配原理见 [通用题集](../agents/rag-frameworks-and-providers.md#不同-rag-provider-返回格式不一致怎么处理)。
- 效果与数据集口径见 [简历指标](./resume-metrics.md#recall5-91-对应多少测试样本)；原稿容量示例不是实测值，个人贡献需代码或报告支持。

### 易追问

- 模型适配具体改了哪里？换 Embedding 与换向量库各需迁移什么？

### 关联笔记

- [DeepTutor RAG Provider 与 LightRAG](../../notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md)
