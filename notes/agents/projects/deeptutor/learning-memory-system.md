---
type: Note
domain: agents
title: DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory
aliases: [DeepTutor Memory, 三级可控学习记忆, L1 L2 L3 Memory, Memory Consolidator]
description: 梳理 DeepTutor 长期学习记忆从原始 Trace、增量 Consolidator、引用校验、AddOp 写入到 L3 注入 Agent Context 的完整生命周期
tags: [deeptutor, memory, long-term-memory, consolidator, provenance, agent-context]
status: learning
created: 2026-09-02
updated: 2026-09-21
---

# 概述

- DeepTutor 没有把长期记忆做成黑盒向量库，而是拆成 L1、L2、L3 三层可审计记忆。
- L1 是原始活动证据层，包括追加式 Trace 与业务数据的 Snapshot Entity 投影；L2 从新增 Entity 中抽取场景事实；L3 聚合多个 Surface 的 L2 信息，形成 `recent`、`profile`、`scope` 等跨场景长期记忆。
- Consolidator 不是每次全文总结，而是通过 meta 记录已处理的 Entity/Entry ID，只消费新增输入，按 Chunk 抽取候选事实，经过 Reference Validation 后用 AddOp 追加到 Memory Document。
- 这套设计强调可控：用户可以 Inspect、Edit、Delete、Reset、Audit、Dedup、Merge 和 Undo；追溯粒度需要区分 L2 的实体引用与当前 L3 的场景引用。
- 运行时 L3 可以作为 `memory_context` 注入 Agent Prompt，也可以通过 `read_memory` Tool 按需读取。

# 详细内容

## 为什么需要长期记忆

- Conversation History 解决“当前会话刚才说了什么”，主要是 Session-local 的短期上下文。
- Memory 解决“这个用户长期是谁”，覆盖跨会话、跨场景的长期用户状态。
- 如果每轮都把所有历史对话塞进 Prompt，历史越长，Context Window 越容易膨胀，真正有价值的用户信息占比也会下降。
- 长期记忆的目标不是保存全部对话，而是把历史中的高价值用户事实、学习状态和偏好沉淀出来。

## 为什么不直接使用 Vector DB

- Vector DB 适合大规模语义检索，但长期用户画像还需要可查看、可编辑、可删除和可追溯。
- 如果用户画像完全依赖黑盒向量召回，用户很难知道系统记住了什么，错误事实也不容易定位和纠正。
- DeepTutor Memory 首先是可管理、可检查的数据系统，其次才是 Retrieval 问题。
- 简单区分：Memory 主要保存 User Knowledge，RAG 主要提供 Domain Knowledge。

## L1：Evidence

- L1 先保存原始行为和事件，不急着生成高层画像。
- 当前 Surface 为 `chat`、`notebook`、`quiz`、`kb`、`book`、`partner`、`cowriter`，避免把不同产品场景的信息混在一起。
- 追加式 Trace 存于 `trace/<surface>/<YYYY-MM-DD>.jsonl`；事件包含 `id`、`ts`、`surface`、`kind`、`payload`、`session_id`、`turn_id`。
- Update 的直接输入是 `snapshot.read_snapshot(surface)` 返回的 Entity，字段包括 `id`、`label`、`ts`、`content`、`metadata`、`fingerprint`。这是业务数据的统一投影，不表示所有聊天正文都复制在 Trace JSONL 中。

| Surface | Snapshot Entity 的粒度与主要内容 |
|---|---|
| `chat` | 一个聊天 Session，包含用户与助手消息 |
| `notebook` | 一条笔记记录，含标题、查询、摘要与输出 |
| `quiz` | 一次作答，含题目、用户答案、正确性与解释 |
| `kb` | 一个已注册知识库的配置；查询行为 Trace 是另一类事件证据 |
| `book` | 一本书，含标题、描述、页面标题与元数据 |
| `partner` | 一个 Partner 会话 JSONL，可来自 Partner 级或按用户隔离的会话目录 |
| `cowriter` | 一份共写文档 |

- 设计原则是先保留证据，不要把一次瞬时行为过早推断成长期人格判断。

## L2：Surface-specific Facts

- L2 写入 `L2/<surface>.md`，从该 Surface 的原始 Entity 中抽取事实，而不是把所有原文直接拼接。
- L2 解决 L1 太原始、太多、不适合直接进入 Agent Context 的问题。
- L2 引用格式是 `surface:entity_id`，每条事实的证据必须落在当前 Chunk 允许的实体范围内。
- 提取重点由场景 Prompt 配置：Chat 的误解/掌握/话题、Quiz 的错误模式/强项/弱项、KB 的兴趣/高频查询/文献库空白、Book 的节奏/卡点/标注主题、Partner 的主题/频道、Cowriter 的风格/反复修改/话题。
- L2 让“场景事实抽取”和“跨场景长期画像”分开，降低单次错误事件直接污染全局画像的风险。

## L3：Long-term Profile

- L3 聚合多个 Surface 的 L2 事实，形成跨场景长期记忆。
- 文件是 `L3/<slot>.md`；聚合依据是 Slot 的语义目标和已有记忆，由 LLM 生成候选事实，并非固定“每 N 条记录合一条”的统计规则。
- 常见 Slot 包括 `recent`、`profile`、`scope` 和 `preferences`。
- `recent` 描述近期正在做什么，`profile` 描述相对稳定的个人背景，`scope` 描述长期目标或活动范围，`preferences` 描述长期偏好。
- `preferences` 不走普通自动 Consolidation，而通过更明确的 `write_memory` 路径写入，避免从一次偶发行为中过度推断长期偏好。

## 增量 Consolidator

- Consolidator 的 Update Mode 更接近 Chunk-based Incremental Fact Extraction，不是全文总结。
- L2 更新消费新增 L1 Entity；L3 更新消费新增 L2 Entry。
- Meta 中的 `seen_entity_refs` 或 `seen_l2_entry_ids` 类似 Consumer Offset / Watermark，用来记录已经处理过的输入。
- L2 先按 `(ts, id)` 排序 Entity，再用 `seen_entity_refs` 过滤已消费引用，渲染、分 Chunk、调用 LLM、校验引用、通过 AddOp 追加条目，最后保存 meta。
- L3 读取各 Surface 的 L2 文档，按 `seen_l2_entry_ids` 过滤、按条目 ID 排序，再按 Surface 渲染文本、分 Chunk、综合抽取、写入目标 Slot 并保存 meta。
- Entity 中有 `fingerprint` 不等于这条 Update 路径会凭内容变化自动重算；这里的增量判断以已见引用/条目 ID 为依据，不能把它理解成全文差异检测。
- 每个 Chunk 连同 Existing Memory、Allowed References 与提取目标一起交给 LLM；已有记忆用于减少重复，引用校验用于约束证据来源。
- Dedup、Merge、Audit 是独立维护模式；Update 也可根据 `dedup.auto_after_update` / `merge.auto_after_update` 设置接续去重与合并。

## Validation 与 AddOp

- LLM 只负责生成 Candidate Facts，不是 Memory 数据库的最终权威。
- 系统代码会执行 Reference Validation，检查候选事实引用是否属于当前允许的证据池；非法引用直接丢弃。
- 写入时不让 LLM 直接覆盖文件，而是通过 AddOp 进入 Memory Operation Layer。
- AddOp 统一处理 Entry ID 分配、数据校验、Document Invariant、Reference / Footnote rebuild 等一致性工作。
- 这体现了分工：LLM 负责语义抽取，代码负责一致性和证据约束。

## Checkpoint 与 Meta

- Chunk 处理过程中可以阶段性写入有效事实，但 seen meta 应该在整批新增输入真正完成后才推进。
- 如果 Chunk 1 和 Chunk 2 成功，Chunk 3 失败，却提前把所有输入标记为 seen，失败输入以后可能永远不会再被处理。
- 原则是先完成数据写入，再推进 Watermark。

## Provenance

- 当前 L2 使用 `surface:entity_id` 精确引用源 Entity；L3 使用 `chat`、`quiz` 等裸 Surface 名引用 L2 文档，不能宣称 L3 每条结论都精确引用某个 L2 Entry。
- `seen_l2_entry_ids` 是增量消费账本，不等于 L3 输出事实的引用字段；输入去重粒度与输出来源粒度不同。
- `render_l2_entries_for_concat()` 给模型的是 Surface 标题与事实正文，不携带原 L2 的逐条 ID/脚注；`refs_in_span_l3()` 按 Chunk 覆盖的 Surface 构造允许引用集合。
- 2026-09-21 本地代码仍有“L3 无 refs”等旧注释，以及按 L2 ID 查找的审计辅助逻辑；记录以当前 Update 校验与 Prompt 为准，不据此推断端到端精确追溯已经完整实现。

## 可控维护

- Inspect：L2/L3 是显式 Markdown 文档，用户能直接查看系统记住了什么。
- Edit：用户可以直接修正错误事实，模型生成 Memory，但用户保留最终编辑权。
- Delete：支持删除单条 Entry，避免只能整份清空。
- Reset：不能只删 `.md`，还要清理对应 meta watermark，否则 Update 会误以为旧 L1 已处理。
- Audit / Dedup / Merge：分别负责检查、去重和整理，不和增量抽取混在同一个职责里。
- Undo：Consolidator Run 可能写入多条 Memory，Run-level Undo 让长期记忆变更可逆。

## 错误 Memory 风险

- 普通错误回答通常只影响一轮；错误长期 Memory 会进入未来 Prompt，影响后续多个 Session。
- 错误 Memory 还可能诱导 Agent 产生更多错误 Trace，再继续污染 Memory，形成 Memory Pollution Feedback Loop。
- 因此长期记忆必须支持检查、追踪和纠错，而不是只追求“多记”。

## Runtime 消费

- L3 最适合进入 Prompt：L1 适合审计，L2 适合整理，L3 适合推理时消费。
- 方式一是 Eager Context Injection：Runtime 读取 L3，形成 `context.memory_context`，由 `ChatPromptAssembler` 放入独立的 `## memory` System Prompt Block。
- 方式二是 On-demand Retrieval：模型在执行过程中调用 `read_memory` Tool，Memory 作为 `role=tool` 结果进入下一轮 LLM。
- 不应该每轮无脑全量注入所有 Memory；`memory_context` 需要受相关性和上下文预算控制。

## Partner 的 Split Memory 边界

- 本节最初依据 [Partner 运行时复用与隔离复习稿](../../../../interview/meta/source/deeptutor-partner-runtime-reuse-and-isolation.md) 整理；2026-09-21 补查 `tools/partner_memory.py`、`services/partners/interaction.py` 与 Partner 路径辅助函数，核对以下读写边界，未运行隔离测试。
- 前文普通 Chat 的 `read_memory` / `write_memory` 路径不能直接套用到 Partner：Partner 使用专属工具限制读写范围。
- 用户 Shared L3 是跨场景长期信息，对 Partner 只读；Partner 的关系记忆按 `Partner × User` 隔离，可通过 `partner_memorize` 写入。
- 关系目录可概念化为 `data/partners/<partner_id>/users/<user_id>/workspace/memory/`；需要同时携带 Partner 与真实用户两个维度，不能只按 Partner 分目录。
- 例如“算法题先别给提示”可以属于算法 Partner 与当前用户的关系偏好，不必改变英语 Partner 或全局用户画像。
- 按材料，`partner_read` 提供授权范围内的共享与关系记忆读取；`partner_memorize` 只能修改当前关系记忆，普通 `read_memory` / `write_memory` 从 Partner 工具集合中移除。
- L1/L2/L3 描述信息加工层级，Shared / Relationship 描述所有权与权限边界；关系记忆不是新增的 L4，也不能仅凭目录名推断它采用相同 Consolidator 流程。
- 写入测试应同时检查目标关系记忆、其它 Partner / User 的关系记忆和 Shared L3，验证既能完成合法写入，又没有扩大写入范围。

## 三类 Context

| 类型 | 解决的问题 | 生命周期 |
|---|---|---|
| Conversation History | 当前会话刚才说了什么 | Session / Branch |
| Memory | 用户长期是谁 | Cross-session |
| RAG | 外部知识是什么 | Knowledge Base |

- 例子：“按照我之前的学习习惯，结合《算法导论》，继续讲刚才的动态规划。”
- “继续讲刚才的动态规划”依赖 Conversation History；“按照我之前的学习习惯”依赖 Memory；“结合《算法导论》”依赖 RAG。

# 相关链接

- [2026-09-21 学习日志](../../../../daily/2026-09-21.md)
- 源码核对入口（DeepTutor 仓库内路径）：`services/memory/` 下的 `paths.py`、`snapshot/adapters.py`、`consolidator/modes/update.py`、`consolidator/references.py` 与 `consolidator/prompts/zh/_meta.yaml`，均位于 `deeptutor/` 包内。
- [记忆修改 JSON 被尾随花括号干扰：上游 Bug 与回归测试](../../../../pitfalls/deeptutor-memory-json-trailing-braces.md)
- [DeepTutor Agent Runtime：Turn 生命周期与事件流](./agent-runtime-and-turn-lifecycle.md)
- [DeepTutor：知识库与 RAG Provider 架构](./knowledge-base-and-rag-providers.md)
- [Agent Memory 面试题](../../../../interview/agents/memory.md)
- [KindPartner 项目答辩](../../../../interview/resume/kindpartner.md)
- [Partner、Workspace 与多租户隔离](../../../../interview/agents/agent-multi-tenancy.md)
- [Partner 运行时复用与隔离复习稿（归档）](../../../../interview/meta/source/deeptutor-partner-runtime-reuse-and-isolation.md)
- [2026-09-05 学习日志](../../../../daily/2026-09-05.md)
- [简历指标与证据口径](../../../../interview/resume/resume-metrics.md)
- [DeepTutor 三级可控学习记忆复习稿（归档）](../../../../interview/meta/source/deeptutor-three-level-controllable-learning-memory.md)
