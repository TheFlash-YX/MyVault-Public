---
type: interview
scope: topic
area: agents
topics:
  - memory
  - long-term-memory
  - context-management
created: 2026-08-27
updated: 2026-09-02
---

# Agent Memory

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。
- DeepTutor Memory 可参考：[DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)。
- 速记：`L1 = Evidence`，`L2 = Facts`，`L3 = Long-term Profile`；`LLM = Candidate Fact Generator`，`Code = Consistency / Validation Controller`。

## 为什么 Agent 需要 Memory？

<!-- canonical_key: agents.memory.wei-shen-me-agent-xu-yao-memory -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- Agent 需要 Memory 是因为单次 Session History 只能解决当前会话连续性，无法稳定承载跨会话的用户背景、学习状态和长期偏好。
- 如果把所有历史消息都塞进 Prompt，Context 会膨胀，低价值历史会挤占当前任务空间。
- 长期 Memory 的目标不是保存全部对话，而是把高价值用户事实沉淀成可控、可追溯、可纠错的数据。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## Context 和 Memory 有什么区别？

<!-- canonical_key: agents.memory.context-he-memory-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- Context 是本次 Turn 运行时组装出来的输入快照，包含当前消息、历史摘要、工具、知识库、Memory 等信息。
- Memory 是跨会话保存的长期用户状态，例如近期目标、背景、学习偏好和薄弱项。
- 简单说：Context 是“这次模型能看到什么”，Memory 是“系统长期记住了什么”。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## Short-term Memory 和 Long-term Memory 有什么区别？

<!-- canonical_key: agents.memory.short-term-memory-he-long-term-memory-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- Short-term Memory 更接近当前 Session/Branch 的 Conversation History，解决刚才聊了什么。
- Long-term Memory 是跨 Session、跨 Surface 的用户状态，解决用户长期是谁、最近在做什么、稳定偏好是什么。
- 长期记忆不能简单等于聊天记录，需要经过抽取、压缩、引用校验和用户可控维护。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 为什么你们设计成 L1、L2、L3 三层？

<!-- canonical_key: agents.memory.wei-shen-me-ni-men-she-ji-cheng-l1-l2-l3-san-ceng -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 长期记忆同时要解决证据完整、信息压缩和推理时个性化三个问题。
- L1 保存按 Surface 划分的原始 Trace，保证真实性；L2 把单个 Surface 的新增 Trace 整理成 Facts；L3 再跨 Surface 聚合成长期画像。
- 这让高层结论既能进入 Agent Context，又能沿 `L3 -> L2 -> L1` 追溯到底层证据。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## L1 Trace 保存什么？

<!-- canonical_key: agents.memory.l1-trace-bao-cun-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- L1 保存原始事件和行为证据，例如用户在 chat 中说的目标、quiz 中连续做错的题、course 中正在学习的主题。
- L1 按 Surface 拆分，例如 `chat`、`quiz`、`course`、`notebook`、`reading`。
- 关键是先保留证据，不要把一次瞬时行为直接推断成长期人格判断。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 为什么不能直接把聊天记录全部当记忆？

<!-- canonical_key: agents.memory.wei-shen-me-bu-neng-zhi-jie-ba-liao-tian-ji-lu-quan-bu-dang-ji-yi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 全部聊天记录价值密度不均，长期使用后会造成 Prompt 过长和噪声过多。
- 对话里也有大量临时状态、玩笑、反悔和一次性任务，直接当长期记忆会污染用户画像。
- 更合理的链路是 Conversation History 先进入 Trace，再通过 Consolidator 抽取高价值事实，最后进入长期 Memory。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## L2 是如何从 L1 提取的？

<!-- canonical_key: agents.memory.l2-shi-ru-he-cong-l1-ti-qu-de -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- L2 通过 Consolidator 从新增 L1 Trace 中增量提取 Surface-specific Facts。
- 流程是 `Meta Diff -> sort -> Chunk -> LLM Fact Extraction -> Reference Validation -> AddOp -> Append to L2`。
- Meta 里的 seen refs 类似 Consumer Offset，只处理新增输入，避免每次全文重算带来的成本和 Memory Drift。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## Memory Consolidator 是模型还是规则？

<!-- canonical_key: agents.memory.memory-consolidator-shi-mo-xing-hai-shi-gui-ze -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 它是模型和规则的组合，不是纯 LLM。
- LLM 负责从 Chunk 中生成 Candidate Facts；代码负责 diff、chunk、引用校验、AddOp 写入、Document invariant 和 meta watermark。
- 一句话：LLM 负责语义判断，系统代码负责一致性和证据合法性。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## Memory Consolidator 的 Prompt 怎么设计？

<!-- canonical_key: agents.memory.memory-consolidator-de-prompt-zen-me-she-ji -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- Prompt 不只给新增 Trace，还要给 Existing Memory、当前 Chunk、Allowed References 和 Surface Focus。
- Existing Memory 让模型知道已经记过什么，减少重复抽取。
- Allowed References 限制候选事实只能引用当前允许的证据池，后续再由代码做 Reference Validation。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 哪些内容应该进入长期记忆？

<!-- canonical_key: agents.memory.na-xie-nei-rong-ying-gai-jin-ru-zhang-qi-ji-yi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- L3 从新增 L2 Entries 中增量形成跨 Surface 长期记忆，常见 slot 包括 `recent`、`profile`、`scope`。
- 它复用类似 L2 的链路：diff、chunk、extract、validate、AddOp、checkpoint、advance meta。
- `preferences` 是特殊 slot，不走普通自动 Consolidation，而通过更明确的写入路径降低过度推断风险。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 哪些内容绝对不应该进入长期记忆？

<!-- canonical_key: agents.memory.na-xie-nei-rong-jue-dui-bu-ying-gai-jin-ru-zhang-qi-ji-yi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- L3 是高层长期画像，直接引用所有 L1 会让它过重，也会混淆场景边界。
- L2 先把每个 Surface 的 Trace 压缩成事实，L3 再引用 L2，形成更清晰的数据血缘。
- 这样既能让 L3 适合进入 Prompt，又能通过 `L3 -> L2 -> L1` 追溯原始证据。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## L3 画像是如何形成的？

<!-- canonical_key: agents.memory.l3-hua-xiang-shi-ru-he-xing-cheng-de -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 引用链提供 Provenance / Data Lineage，让高层 Memory 能解释“这个结论从哪里来”。
- 它支持审计和纠错：L3 错了可以看 L2，L2 可疑可以追到 L1 Trace。
- 它也支持删除和重建：源数据错误时可以删除 L1，再 Reset/Rebuild 相关 L2/L3。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 为什么 L3 要引用 L2，L2 再引用 L1？

<!-- canonical_key: agents.memory.wei-shen-me-l3-yao-yin-yong-l2-l2-zai-yin-yong-l1 -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 如果只是高层总结错了，可以直接编辑或删除对应 L3 Entry。
- 如果 L2 事实错了，应修正 L2 后重新生成或审计相关 L3。
- 如果 L1 Trace 本身错了，要删除源 Trace，并清理对应 meta 后 Reset/Rebuild，避免 seen watermark 阻止重新处理。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 引用链有什么价值？

<!-- canonical_key: agents.memory.yin-yong-lian-you-shen-me-jia-zhi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 不应该盲目全量重写，而是根据变更影响范围做增量重算、审计或显式 Reset。
- 如果 L3 通过引用链依赖被修改的 L2 Entry，就需要标记相关 L3 为待审计或重新 Consolidate。
- 关键是 meta watermark 要和实际文档状态一致，否则可能漏处理或重复处理。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 如果 L3 画像是错误的怎么办？

<!-- canonical_key: agents.memory.ru-guo-l3-hua-xiang-shi-cuo-wu-de-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 会，尤其是用户状态变化、不同 Surface 信息不一致、短期情绪被误写成长期偏好时。
- 不应该直接用新事实覆盖旧事实，而要保留时间、来源和置信边界。
- 冲突处理可以通过 Audit/Merge、人为编辑、时间衰减和更高层的 L3 重整来完成。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 用户修改 L2 以后 L3 是否重新生成？

<!-- canonical_key: agents.memory.yong-hu-xiu-gai-l2-yi-hou-l3-shi-fou-zhong-xin-sheng-cheng -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先不要立刻推断“用户不喜欢数学”，因为后一句可能只是短期状态。
- L1 保留原话，L2 可以描述为某次学习中数学意愿下降，L3 只有在更多跨场景证据支持时才更新稳定偏好。
- 这正是 L1/L2/L3 的缓冲价值：原始行为和长期画像之间多一层证据过滤。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## Memory 是否会出现冲突？

<!-- canonical_key: agents.memory.memory-shi-fou-hui-chu-xian-chong-tu -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 用户之前说“喜欢数学”，后来又说“不喜欢数学”，怎么处理？

<!-- canonical_key: agents.memory.yong-hu-zhi-qian-shuo-xi-huan-shu-xue-hou-lai-you-shuo-bu-xi-huan-shu-xue-zen-me-chu-li -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Memory 是否需要时间衰减？

<!-- canonical_key: agents.memory.memory-shi-fou-xu-yao-shi-jian-shuai-jian -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Memory 是否需要置信度？

<!-- canonical_key: agents.memory.memory-shi-fou-xu-yao-zhi-xin-du -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Memory 如何做去重？

<!-- canonical_key: agents.memory.memory-ru-he-zuo-qu-zhong -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Memory 如何做合并？

<!-- canonical_key: agents.memory.memory-ru-he-zuo-he-bing -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Memory 怎么检索？

<!-- canonical_key: agents.memory.memory-zen-me-jian-suo -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 运行时有两类消费方式：一种是把相关 L3 形成 `memory_context` 直接注入 Prompt，另一种是通过 `read_memory` Tool 按需读取。
- L3 信息密度高，适合作为长期背景进入 Agent Context；L1 更适合审计，L2 更适合整理。
- 不应该把所有 Memory 无脑塞进每轮 Prompt，需要根据任务相关性、slot 和上下文预算筛选。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 是语义检索还是结构化查询？

<!-- canonical_key: agents.memory.shi-yu-yi-jian-suo-hai-shi-jie-gou-hua-cha-xun -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- Memory 不必被简化成纯语义检索；L3 slot、Surface、Entry ID、引用关系都属于结构化信号。
- 小规模高价值 Memory 可以优先结构化选择；规模变大后可以叠加 embedding/top-k。
- 关键是 Retrieval 不能绕过用户作用域、引用链和可控编辑语义。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 每轮对话需要把所有 Memory 放进 Prompt 吗？

<!-- canonical_key: agents.memory.mei-lun-dui-hua-xu-yao-ba-suo-you-memory-fang-jin-prompt-ma -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 不需要，也不应该。
- Memory 是长期状态，但当前 Turn 未必都相关；全量注入会浪费上下文，还可能把无关偏好带入模型输出。
- 更合理的方式是相关 L3 预注入，剩余 Memory 通过 `read_memory` Tool 按需读取。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 如何选择当前 Turn 相关的 Memory？

<!-- canonical_key: agents.memory.ru-he-xuan-ze-dang-qian-turn-xiang-guan-de-memory -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先看 Turn 的任务类型、Capability、用户当前问题和可用上下文预算。
- 明确相关的 L3 slot 可以预注入，例如近期学习目标、稳定背景和当前 scope。
- 不确定或更细的记忆可以交给模型通过 `read_memory` Tool 按需读取，避免第一轮 Prompt 过重。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## Memory 很多以后，检索性能怎么处理？

<!-- canonical_key: agents.memory.memory-hen-duo-yi-hou-jian-suo-xing-neng-zen-me-chu-li -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Memory 会不会污染模型输出？

<!-- canonical_key: agents.memory.memory-hui-bu-hui-wu-ran-mo-xing-shu-chu -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 会，错误长期 Memory 的风险比普通错误回答更大，因为它会进入未来多个 Session。
- 典型链路是错误 Memory 进入 Prompt，影响回答，再产生更多错误 Trace，继续污染 Memory。
- 所以 Memory 必须可 Inspect、Edit、Delete、Audit、Dedup、Merge 和 Undo，并保留引用链。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 你们为什么强调“后台静默沉淀率 0%”？

<!-- canonical_key: agents.memory.ni-men-wei-shen-me-qiang-diao-hou-tai-jing-mo-chen-dian-lu-0 -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 长期 Memory 影响未来回答，所以不能悄悄把用户的每句话都沉淀成长期画像。
- 强调 0% 的核心是用户可控：写入、编辑、删除、Reset 和 Audit 都应有明确入口和可见结果。
- 这不是说系统不能辅助抽取，而是 LLM 不能在用户不可见、不可回滚的情况下静默改变长期状态。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)
- [简历指标与证据口径](../resume/resume-metrics.md)

## 为什么让用户主动决定是否写入 Memory？

<!-- canonical_key: agents.memory.wei-shen-me-rang-yong-hu-zhu-dong-jue-ding-shi-fou-xie-ru-memory -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 用户长期记忆包含身份、目标、偏好和学习状态，属于高影响个人数据。
- 主动决定写入可以减少误记、过度推断和隐私风险，也让用户知道系统到底记住了什么。
- Agent 可以建议候选 Memory，但最终应允许用户确认、编辑或拒绝。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 这种设计相比自动 Memory 有什么优劣？

<!-- canonical_key: agents.memory.zhe-zhong-she-ji-xiang-bi-zi-dong-memory-you-shen-me-you-lie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 优点是可控、可审计、可纠错，长期画像更可信。
- 缺点是交互成本更高，用户不确认时系统可能少记一些有用信息。
- 取舍上，长期 Memory 影响未来多轮任务，宁愿少记，也不要把错误事实悄悄写进用户画像。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 隐私方面怎么处理？

<!-- canonical_key: agents.memory.yin-si-fang-mian-zen-me-chu-li -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何支持用户删除记忆？

<!-- canonical_key: agents.memory.ru-he-zhi-chi-yong-hu-shan-chu-ji-yi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 删除要支持 Entry-level Correction，用户可以删除单条 Memory，而不是只能清空整份文档。
- 如果删除的是 L3/L2 派生事实，要维护文档结构、引用和后续审计状态。
- 如果删除 L1 源 Trace，还要考虑由它派生出的 L2/L3 是否需要 Reset/Rebuild。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 删除 L1 后，L2/L3 怎么办？

<!-- canonical_key: agents.memory.shan-chu-l1-hou-l2-l3-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- L1 是证据源，删除后依赖它的 L2/L3 事实需要被标记、审计或重建。
- 如果选择 Reset，不能只删 Memory 文档，还要清理对应 meta watermark。
- 否则 Consolidator 会以为旧 L1 已经 seen，导致重建时漏掉应重新处理的输入。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 删除 L2 后引用它的 L3 怎么办？

<!-- canonical_key: agents.memory.shan-chu-l2-hou-yin-yong-ta-de-l3-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- L3 引用 L2，因此删除 L2 后引用它的 L3 Entry 不能继续当作可信结论。
- 可以把相关 L3 标记为失效、删除，或从剩余 L2 重新 Consolidate。
- 核心是引用链不能悬空，否则 Provenance 和 Audit 都会失效。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## Memory 是否应该做软删除？

<!-- canonical_key: agents.memory.memory-shi-fou-ying-gai-zuo-ruan-shan-chu -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 软删除适合保留审计和 Undo 能力，但运行时 Retrieval 必须默认排除软删除 Entry。
- 对隐私或合规要求更高的数据，用户要求彻底删除时不能只做软删除。
- 面试时可以按“普通纠错保留审计，隐私删除支持硬删除”区分。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## Memory 数据是否需要加密？

<!-- canonical_key: agents.memory.memory-shu-ju-shi-fou-xu-yao-jia-mi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 你写“引用覆盖率 100%”，怎么定义这个指标？

<!-- canonical_key: agents.memory.ni-xie-yin-yong-fu-gai-lu-100-zen-me-ding-yi-zhe-ge-zhi-biao -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 技术口径可以定义为：所有非手动自由文本的 L2/L3 Memory Entry 都带有合法引用，并且引用能解析到允许的 L1 或 L2 来源。
- L2 看是否引用合法 L1 Trace；L3 看是否引用合法 L2 Entry，必要时再追溯到 L1。
- 这只是引用结构完整性指标，不等于 Memory 内容 100% 正确。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)
- [简历指标与证据口径](../resume/resume-metrics.md)

## 记忆系统如何做测试？

<!-- canonical_key: agents.memory.ji-yi-xi-tong-ru-he-zuo-ce-shi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 单测覆盖 diff/watermark、chunk 边界、Reference Validation、AddOp invariant、删除/Reset/Undo。
- 集成测试覆盖 L1 -> L2 -> L3 -> memory_context/read_memory 的完整链路。
- 评测样本要区分 Runtime 连续性、引用完整性、语义正确性和隐私隔离，不要用单一准确率概括全部。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 如何判断某条 Memory 已经失效？

<!-- canonical_key: agents.memory.ru-he-pan-duan-mou-tiao-memory-yi-jing-shi-xiao -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 看时间、来源、是否有后续冲突事实、用户是否手动修正，以及该 Memory 是否仍被当前任务使用。
- `recent` 类记忆更容易失效，`profile` 类更稳定，但也不能永久假设正确。
- 失效处理可以是降权、标记待审计、合并新事实或删除旧 Entry。

### 易追问

-

### 关联笔记

- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 不同 Partner 是否应该共享 Memory？

<!-- canonical_key: agents.memory.bu-tong-partner-shi-fou-ying-gai-gong-xiang-memory -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何防止 Memory Retrieval 把其他用户的信息召回？

<!-- canonical_key: agents.memory.ru-he-fang-zhi-memory-retrieval-ba-qi-ta-yong-hu-de-xin-xi-zhao-hui -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果 Memory 数量达到百万级，你会如何设计存储和检索？

<!-- canonical_key: agents.memory.ru-guo-memory-shu-liang-da-dao-bai-wan-ji-ni-hui-ru-he-she-ji-cun-chu-he-jian-suo -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-
