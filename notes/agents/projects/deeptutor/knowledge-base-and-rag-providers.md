---
type: Note
domain: agents
title: DeepTutor：知识库与 RAG Provider 架构
aliases: [DeepTutor KB, RAG Provider, Knowledge Base, RAG Pipeline, LightRAG 知识检索, NanoVectorDB]
description: 梳理 DeepTutor 的 RAG Provider 绑定与检索策略，以及 LightRAG 建图、模型适配、来源追溯、存储和教育 QA 评测边界
tags: [deeptutor, knowledge-base, rag, rag-provider, retrieval, embedding, lightrag, evaluation]
status: learning
created: 2026-08-23
updated: 2026-09-23
---

# 概述

- KB 是 Knowledge Base，即教材、论文、课件和内部文档等外部资料的知识库；它为回答提供事实依据，不等同于会话历史或用户长期 Memory。
- DeepTutor 采用“离线索引 + Turn 级绑定 + 自动预检索 + Agent 按需追加检索”的处理策略，不把完整知识库直接塞入模型上下文。
- RAG Provider 不是 LLM 供应商，而是知识库索引和检索引擎；DeepTutor 用统一 `RAGService` 门面和 `RAGPipeline` 协议屏蔽不同引擎的实现差异。
- 一个 KB 创建时绑定一个 Provider，后续增量索引和查询继续走同一 Pipeline，不由 LLM 根据每次问题动态更换 Provider。

# 详细内容

## KB、History 与 Memory 的区别

- KB 保存教材、论文和业务文档等外部资料，主要解决“依据哪些资料回答”。
- Conversation History 保存当前 Session 的用户与助手消息，主要解决“对话如何保持连贯”。
- Memory 保存用户偏好、长期事实和学习状态，主要解决“跨会话如何个性化”。
- Workspace Files 是 Agent 可以直接读取或生成的文件，主要解决任务产物和文件操作，不天然等同于已经建立检索索引的 KB。

## 离线索引与版本管理

- 默认 LlamaIndex 流程为“文档解析 -> SentenceSplitter 分块 -> Embedding -> 向量索引 -> BM25 关键词索引 -> 持久化”。
- 原始文件保存在 `<kb>/raw/`，新索引写入平铺的 `version-N/` 目录；旧版本继续保留，避免重建失败破坏当前可用索引。
- LlamaIndex 版本记录 Embedding Binding、模型、维度、Base URL 和 API Version，并计算稳定的 Embedding Signature。
- 查询时只读取与当前 Embedding Signature 匹配的版本；找不到匹配索引时返回 `needs_reindex`，避免用不同向量空间强行检索。
- BM25 依赖不可用时可以降级为纯向量检索，不让可选的关键词索引拖垮整个查询链路。

## Turn 级绑定与工具挂载

- 当前选择的 KB 名称通过 `UnifiedContext.knowledge_bases` 进入 Turn，同一个 Session 的不同 Turn 可以绑定不同知识库组合。
- 当 `has_kb=true` 时，工具组合规则自动挂载 `rag` 和 `kb_files`；没有 KB 时模型看不到这两个工具，也无法伪造调用。
- `rag` 用于按问题检索相关内容，`kb_files` 用于枚举知识库中的文件；资料清单问题应使用文件 Manifest 或 `kb_files`，而不是让相似度检索猜测库存。
- 系统提示词只放轻量 KB Manifest，不放完整文档；这能控制 Token、降低无关内容干扰，并让大文件继续按需读取。
- 对传统 RAG KB，Chat 在主循环前会使用用户问题自动预检索，最多并行处理前 3 个 KB，每个 KB 最多注入约 4000 字符的结果。
- 预检索结果追加到当前 Turn 的尾部用户上下文，便于复用稳定的系统提示词前缀；进入 Agent Loop 后，模型仍可以改写查询并继续调用 `rag`。
- PageIndex 和 Capability 自有知识库不参加传统 RAG 预检索，它们在 Agent Loop 内通过专属工具逐步读取。

## RAGService、Pipeline 与 Provider Binding

- `RAGPipeline` 统一约定 `initialize()`、`add_documents()`、`search()` 和 `delete()`，不同引擎返回统一的 `query`、`content/answer`、`sources` 和 `provider`。
- `RAGService` 根据 `kb_name` 解析绑定 Provider，再从 Factory 获取对应 Pipeline；上层 Agent 不需要判断底层是向量索引、知识图谱还是远程服务。
- Provider 解析优先级为 `kb_config.json`、旧版 `metadata.json`、默认 `llamaindex`；`kb_config.json` 是当前权威运行配置。
- Provider 和 KB 基础目录共同组成 Pipeline 缓存键，同类 KB 可以复用 Pipeline 实例，但实际查询仍按各自 KB 配置执行。
- 对支持查询模式的引擎，模式通常按“本次显式参数 -> KB 自身配置 -> Provider 全局默认 -> 引擎内置默认”解析。

## 七种 Provider

- **LlamaIndex**：本质上是组装 RAG 应用的通用框架，提供文档加载、切分、索引、Retriever 和 Query Engine 等组件；在 DeepTutor 中被封装为默认本地 Provider，以向量语义召回结合 BM25 关键词召回，适合教材、论文和普通文档。
- **PageIndex Cloud**：托管的无向量、无传统分块引擎，Agent 先检查文档结构，再读取相关页面，即 Reasoning as Retrieval；适合长篇结构化文档，但依赖 API Key、外部服务和模型 Tool Calling。
- **PageIndex OSS**：在本地 Library 中提供与 Cloud 相同的结构化阅读工具，当前主要支持 PDF；一次请求最多选择一个 OSS KB，避免多个本地 PageIndex 上下文相互冲突。
- **GraphRAG**：索引阶段用 LLM 提取实体、关系、社区和社区摘要，支持 `local`、`global`、`drift`、`basic`；适合跨文档关系和全局总结，但建图成本高。
- **LightRAG**：结合知识图谱、文本 Chunk 和向量索引，可与多模态解析层集成；具体解析入口由项目适配决定，不能把上游解析能力等同于 DeepTutor 实际使用的链路。支持多种事实与关系检索模式，详见下方 LightRAG 专节。
- **LightRAG Server**：DeepTutor 本地只保存外部服务连接信息，通过 HTTP 查询远程 LightRAG；索引构建和文档管理由外部服务负责，删除本地 KB 不会删除远程数据。
- **Tencent IMA**：将 DeepTutor KB 连接到 IMA 中已有的知识库，通过 OpenAPI 检索且不复制远端索引；搜索后会并行补全过短的来源正文，一个文档失败不会丢弃其他结果。

## LlamaIndex、PageIndex、GraphRAG 与 LightRAG 的区别

- **定位不同**：LlamaIndex 是可用于搭建多种 RAG 链路的开发框架；PageIndex、GraphRAG 和 LightRAG 更侧重具体的知识组织与检索策略。DeepTutor 将四者都封装成 Provider，是项目层面的统一抽象，不代表它们原生处于完全相同的技术层级。
- **LlamaIndex——通用工具箱**：典型链路是“文档切分 -> Embedding -> 向量与关键词索引 -> 召回和重排 -> LLM 生成”；优势是组件齐全、扩展方便，适合快速构建通用课件问答，效果主要受分块、Embedding 和重排策略影响。
- **PageIndex——按目录翻文档**：保留章节、标题和页码等原始层级，将长文档组织成树；查询时由 LLM 逐层判断应进入哪个分支并读取相关页面，适合教材、论文、规范和长 PDF，检索路径更容易解释，但查询需要额外的树上推理。
- **GraphRAG——先建知识关系网**：从文本中提取实体、关系和事实，再执行社区划分并生成多层摘要；局部检索适合具体实体关系，全局检索适合跨文档主题归纳，优势是关联与全局分析能力强，代价是索引阶段需要较多 LLM 抽取和总结。
- **LightRAG——图谱与向量结合**：同时保留实体关系图、文本块和向量索引，面向事实问答与跨知识点关系检索；收益取决于实体关系抽取质量，仍需与纯向量检索比较效果和成本。

## 四种方案的联系与选型

- “LlamaIndex + PageIndex + GraphRAG + LightRAG”表示一个系统可以接入多种 RAG 后端，通常不是让一次查询依次经过四套引擎。
- DeepTutor 的调用关系可以概括为“用户问题 -> `RAGService` 根据 KB 绑定选择 Provider -> Provider 返回统一正文与来源 -> Agent 结合检索上下文生成回答”。
- 普通课件、零散文档和快速落地优先考虑 LlamaIndex；目录结构清晰的教材或长 PDF 优先考虑 PageIndex；跨文档实体关系和宏观总结优先考虑 GraphRAG；同时需要向量事实召回与图谱关系推理时优先考虑 LightRAG。
- Provider 选择应在知识库创建时根据资料结构、问题类型、索引成本、查询延迟和部署方式确定，而不是由 LLM 在每个 Turn 中随意切换。

## 检索结果与引用

- 各 Provider 的原始结果会归一化为正文和来源列表，来源通常包含文件名、路径、页码、Chunk ID 和相关性分数。
- RAG 查询产生的来源通过 `SOURCES` StreamEvent 发送给前端，使回答正文与引用展示解耦。
- Provider 错误通常作为带 `error_type`、`needs_reindex` 的结构化结果返回，Runtime 可以显示明确状态而不是把底层异常直接混入答案。
- 多 Provider 可以在同一个 Turn 中共存：传统 KB 走 `rag`，PageIndex 走专属 SDK 工具，最终统一进入 Agent 的上下文与引用协议。

## LightRAG 接入与证据范围

- 本节根据 [LightRAG 知识检索复习稿](../../../../interview/meta/source/kindpartner-lightrag-knowledge-retrieval.md) 整理；项目具体配置、类名与个人贡献来自该材料，本次未读取项目源码或执行其评测。
- 材料明确区分既有基础设施与个人工作：项目已有 `RAGPipeline / RAGService` 框架，新增工作是 `LightRagPipeline` 的索引、查询、模型适配及 Source 回填；个人贡献口径见 [KindPartner 项目答辩](../../../../interview/resume/kindpartner.md#新增一个-provider-具体需要实现哪些方法)。
- 材料中的索引链路是 `PDF/PPT/Word -> ParseService -> Parsed Document -> LightRAG Staging -> Chunk -> 实体/关系抽取 -> 图与向量索引`，复用统一 Parse Layer，不重新实现一套文件解析器。
- 查询通过 `Agent -> rag Tool -> RAGService -> KB 绑定的 LightRagPipeline -> LightRAG`；Provider 的绑定职责与每次查询的 mode 选择是两层配置。

## LightRAG 建图与三类向量

- 从 Chunk 抽取实体及关系，合并后形成 Knowledge Graph；实体与关系是抽取结果，可能遗漏、重复或出现错误，不天然等同于原文事实。

| 索引 | 语义入口 | 用途 |
|---|---|---|
| `chunks_vdb` | 原始文本块的 Embedding | 检索原文 Evidence |
| `entities_vdb` | 实体名称与描述的 Embedding | 定位相关知识对象 |
| `relationships_vdb` | 关系描述、关键词等的 Embedding | 定位跨对象的关系 |

- Vector 负责找到语义相关的入口，Graph 提供关系上下文，再经 Source Mapping 回到原始 Chunk；图谱摘要不能替代原文证据。
- 课程材料中概念关联明显、问题需要组合多个知识点时，图谱有机会改善检索；是否优于普通 Vector RAG 应由同一测试集验证。
- 原稿的“计算机网络课程 25 份资料、1200 页、6000 Chunk、8000 Entity、12000 Relationship”是容量示例，不是项目实测；它只说明总向量数可能同时包含三个索引，而非“一页一个向量”。

## LightRAG 查询模式与候选预算

| Mode | 主要检索路径 | 记忆要点 |
|---|---|---|
| `naive` | 原始 Chunk 向量召回 | 找原文 |
| `local` | 低层关键词定位 Entity，再组织相关图上下文 | 找具体知识对象 |
| `global` | 高层关键词定位 Relationship，再组织关系上下文 | 找跨知识点关系，并非遍历全部图 |
| `hybrid` | 合并 local 与 global | 结合实体与关系检索 |
| `mix` | 图谱检索结合直接 Chunk 向量召回 | 增加原文召回路径，不保证必然优于其它模式 |

- `local` 适合以明确实体为中心的问题，例如“`cwnd` 是什么？它和 `ssthresh` 有什么关系？”检索从低层关键词和相关 Entity 出发，再扩展相关关系与原文 Chunk。
- `global` 适合较宽主题和关系问题，例如“TCP 拥塞控制如何在提升带宽利用率与避免拥塞之间权衡？”检索从高层主题关键词定位关系上下文，再回溯相关实体与原文 Chunk；“global”不表示把整张图无差别遍历并塞进上下文。
- 查询模式是检索策略，不是模型自动选出来的 Provider。项目的 LightRAG Native、LightRAG Server 和上游 SDK 默认值需要分开说明。
- 2026-09-23 核对本地 DeepTutor 源码：Native LightRAG 的 `DEFAULT_MODE` 为 `hybrid`（`deeptutor/services/rag/pipelines/lightrag/config.py`）；Chat Agent 自动预检索显式传 `mode="hybrid"`（`deeptutor/agents/chat/chat_agent.py`）。
- Native LightRAG 未收到有效显式模式时，按“调用显式 `mode` → 知识库 `search_mode` → 引擎级 `defaults.provider_modes[provider]` → Adapter 内置默认值”解析（`deeptutor/services/rag/pipelines/modes.py`）。显式传入的模式优先，因此 Chat Agent 这条自动预检索路径固定使用 `hybrid`。
- 独立的 `lightrag-server` Provider 内置默认是 `mix`（`deeptutor/services/rag/pipelines/lightrag_server/config.py`）；这与 DeepTutor Native Adapter 的 `hybrid` 不冲突，因为两者是不同调用路径。
- 上游 LightRAG 的 `QueryParam` 默认值曾核对为 `mix`；它不应被当成所有版本和项目适配器的统一默认。[官方 QueryParam](https://github.com/HKUDS/LightRAG/blob/main/lightrag/base.py)
- 上游区分图候选 `top_k` 与文本块候选 `chunk_top_k`；材料的 `top_k=60` 不是评测中的 Top-5。评测必须先确定最终 Evidence 的去重、排序与截断阶段。[官方参数定义](https://github.com/HKUDS/LightRAG/blob/main/lightrag/base.py)

## 项目配置与统一模型适配

| 参数 | 复习稿中的配置 | 作用 |
|---|---|---|
| `search_mode` | `hybrid` | 项目查询模式 |
| `top_k` | `60` | 内部图检索候选预算 |
| `response_type` | `Multiple Paragraphs` | 回答形式 |
| `max_concurrent_files` | `1` | 解析文件并发 |
| `llm_model_max_async` | `4` | LightRAG LLM 请求并发上限 |
| `entity_extract_max_gleaning` | `1` | 追加抽取轮数 |
| `llm_profile_id` / `llm_model_id` | 空，可选专用配置 | 为检索组件选择模型 |

- 上表是材料记录的项目配置，不是本次实测值或最新 SDK 默认值；需要用对应提交的配置定义复核。
- Gleaning 用追加抽取弥补遗漏，会增加 LLM 成本与延迟；LLM 并发上限用于平衡建库吞吐与限流，不能由“4”直接推导吞吐量。
- LLM / Embedding Adapter 复用项目统一模型客户端；材料允许为 LightRAG 指定专用模型配置，而不改变普通 Chat 的 Active Model。
- 索引和查询必须使用兼容的 Embedding 模型、维度与预处理约定；仅维度相同仍不足以保证同一向量空间。
- 更换不兼容的 Embedding 模型时，应重新计算 Chunk、Entity、Relationship 向量；仅更换向量存储后端且保持模型与数据语义一致时，可以迁移已有向量。

## Source 回填与可追溯边界

- Provider 结果归一化后，正文进入 `ToolResult.content`，检索来源进入 `ToolResult.sources`，Provider、Mode、错误及检索详情进入 Metadata，供 Agent、引用展示与 Trace 分别消费。
- 文件、页码与 Chunk ID 必须来自解析和索引保存的 Source Mapping；缺少页码就保留实际可追踪的文件或 Chunk，不补造定位信息。
- 可追溯问答首先是 Retrieval-level Provenance：能追到检索证据来源；不代表每个生成句子都已与某个 Chunk 精确对齐，更不等于所有生成结论都正确。

## 本地存储、容量与迁移

- 复习稿描述的本地组合是 JSON KV、NanoVectorDB、NetworkX / GraphML 与 JSON Doc Status；这是材料中的存储方案，项目实际 backend 仍需查配置。
- NanoVectorDB 适配器在进程内维护向量状态，以 `working_dir/[workspace/]vdb_<namespace>.json` 持久化，并含同步机制；不能把它概括为“向量默认存 SQLite”或“完全没有并发控制”。[官方适配器实现](https://github.com/HKUDS/LightRAG/blob/main/lightrag/kg/nano_vector_db_impl.py)
- 容量估算：26000 × 1024 × 4 = 106496000 bytes，约 106.5 MB / 101.6 MiB；百万条相同维度 float32 向量约 4.096 GB / 3.81 GiB，均不含元数据、图、JSON 与运行时开销。
- 本地存储的取舍是少部署外部服务；限制需要从内存峰值、文件加载与写回耗时、并发读写和恢复行为评估。“本地向量存储”不等于整个流程离线，模型客户端仍可能调用远程服务。
- Qdrant、PostgreSQL + pgvector、Milvus 是材料提出的后续候选，不能把“百万向量”当作必须更换产品的固定阈值；应依据延迟、吞吐、过滤需求、运维条件及实测决定。
- Provider 抽象隔离检索引擎，Storage 抽象隔离后端；保持上层契约时 Agent / RAG Tool 通常可继续复用，但数据迁移、索引构建、连接配置和一致性验证仍需完成。
- 原始文档是 Source of Truth，Chunk、图与向量是可重建的派生数据；保留解析结果、来源映射和版本配置，避免重建后评测 Evidence ID 失配。

## 教育 QA 评测与错误归因

- 每个样本分开保存 `Question`、`Ground Truth Answer`、`Ground Truth Evidence`；Evidence 按固定粒度定位 File / Page / Chunk，并绑定索引版本。
- 材料描述来源为大学课程课件及中小学近十年期末考试真题；具体文件、年份、样本数量与答案复核记录尚未提供，不把来源描述视为已验证的数据清单。
- 设某 Query 的相关证据集合为 G，最终前五条结果为 R5，则 `Recall@5 = |G ∩ R5| / |G|`；按 Query 平均时应说明宏平均规则，G 为空的处理也要预先规定。
- “前五条命中任意一个相关证据”是 Hit@5；每个 Query 恰好只有一个相关证据时，两者才相同。多证据题命中一个，不能直接记作 Recall 为 1。
- 若采用 Answer Token F1，应先明确中文分词、数字/单位/标点归一化及多标准答案策略，再以匹配 Token 计算 Precision、Recall 和 `2PR/(P+R)`；当前材料未提供评分脚本，不能确认其具体实现。
- Retrieval 指标与生成指标衡量不同阶段，不能将两者直接相减来估算生成失败率。分层排查时先检查所需 Evidence 是否充分进入最终 Prompt，再检查证据利用、推理与评分差异。
- 不把评测题目及标准答案直接灌入可检索 KB；课程知识作为 Evidence 可以保留，近重复题、调参集与最终测试集需分离。
- 材料中的 Recall@5 91%、F1 78% 仅作为待验证的项目表述，真实结果、样本量、baseline 与个人贡献统一记录在 [简历指标口径](../../../../interview/resume/resume-metrics.md)。

# 相关链接

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](./agent-runtime-and-turn-lifecycle.md)
- [KindPartner 项目答辩](../../../../interview/resume/kindpartner.md)
- [RAG 与 RAG 评测面试题](../../../../interview/agents/rag.md)
- [RAG Frameworks 与 Provider 面试题](../../../../interview/agents/rag-frameworks-and-providers.md)
- [简历指标与证据口径](../../../../interview/resume/resume-metrics.md)
- [KindPartner LightRAG 知识检索复习稿（归档）](../../../../interview/meta/source/kindpartner-lightrag-knowledge-retrieval.md)
- [2026-08-23 学习日志](../../../../daily/2026-08-23.md)
- [2026-08-26 学习日志](../../../../daily/2026-08-26.md)
- [2026-09-23 学习日志](../../../../daily/2026-09-23.md)
- [LlamaIndex 官方文档](https://docs.llamaindex.ai/)
- [PageIndex 官方文档](https://docs.pageindex.ai/)
- [Microsoft GraphRAG 官方文档](https://microsoft.github.io/graphrag/)
- [LightRAG 官方仓库](https://github.com/HKUDS/LightRAG)
