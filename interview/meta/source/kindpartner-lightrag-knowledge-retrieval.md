# KindPartner / DeepTutor 面试复习

- 归档说明：保留原复习稿正文。项目配置、贡献与指标来自学习材料，本次未核验项目源码和测试报告；示例容量不作为项目实测数据。
- 整理后的知识与勘误见 [RAG Provider 与 LightRAG 笔记](../../../notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md)，包括项目与上游默认值、Recall@5 / Hit@5、容量单位和来源追溯边界。
- 面试入口：[RAG Provider](../../agents/rag-frameworks-and-providers.md)、[RAG 评测](../../agents/rag.md)、[KindPartner 项目答辩](../../resume/kindpartner.md)、[指标证据](../../resume/resume-metrics.md)。

# 第 4 点：LightRAG 知识检索与可追溯问答

> 用于面试复习。统一口径：项目原本已有 RAG Provider 框架，我主要负责 **LightRAG Provider 的接入与适配**，包括索引、查询模式、统一 LLM / Embedding 适配、Source 回填、效果评测和本地存储方案的工程取舍。

---

## 一、推荐简历表述

**LightRAG 知识检索**：针对私有课件难以被模型访问、传统向量检索对跨知识点关系召回不足的问题，在现有 RAG Provider 框架中接入 LightRAG，实现文档解析、实体/关系抽取、图谱与向量联合检索及 Source 回填，并复用统一 LLM / Embedding 配置；支持 local、global、hybrid 等检索模式，实现本地课件可追溯问答。基于大学课程课件及中小学近十年期末考试真题构建测试集，平均 F1 **78%**，Recall@5 **91%**。

---

## 二、30 秒面试回答

项目本身已经有统一的 RAG Provider 框架，我主要负责把 LightRAG 接进现有 `RAGPipeline / RAGService` 体系。

文档先经过项目统一的 Parse Layer，之后进入 LightRAG，按 Chunk 做实体和关系抽取，构建 Knowledge Graph，同时维护 Chunk、Entity、Relationship 三类向量索引。查询阶段支持 naive、local、global、hybrid、mix 等模式，我们项目默认使用 hybrid，通过 Entity-level 和 Relationship-level Retrieval 获取结构化上下文，再回溯原始 Chunk。

不同 Provider 最终返回的正文和来源会被归一化，`rag` Tool 把正文放入 `ToolResult.content`，把 Retrieval Provenance 放进 `ToolResult.sources`，所以 Agent 能基于本地课件回答，同时前端仍然能够追溯到原始文件、Chunk 或页面。

评测上，我把 Retrieval 和 Generation 分开：Recall@5 看 Ground Truth Evidence 是否进入前 5 条检索结果，F1 看生成答案和标准答案的覆盖程度。在自建教育 QA 测试集上，平均 Recall@5 为 91%，F1 为 78%。

---

## 三、为什么需要 LightRAG

传统 Vector RAG：

```text
Document
   ↓
Chunk
   ↓
Embedding
   ↓
Vector Search
   ↓
Top-K Chunk
```

LightRAG：

```text
Document
   ↓
Chunk
   ↓
Entity Extraction
   ↓
Relationship Extraction
   ↓
Knowledge Graph
       +
Vector Index
   ↓
Graph-enhanced Retrieval
```

可以简单理解为：

> **LightRAG = Knowledge Graph + Vector Retrieval**

它不是用 Graph 替代 Vector，而是增加了一层 Knowledge Structure。

---

## 四、为什么教育课件适合 LightRAG

例如《计算机网络》：

```text
TCP
│
├── 三次握手
├── 可靠传输
│    ├── ACK
│    └── Retransmission
└── 拥塞控制
     ├── Slow Start
     ├── cwnd
     ├── ssthresh
     └── Congestion Avoidance
```

用户问：

> 为什么 cwnd 达到 ssthresh 后，增长方式从指数增长变成线性增长？

这个问题不只是找一个包含 `cwnd` 的 Chunk，还需要关联：

```text
cwnd
↓
Slow Start
↓
ssthresh
↓
Congestion Avoidance
```

因此，对跨 Chunk、概念关系明显、需要关联多个知识点的问题，Graph-enhanced Retrieval 更有价值。

---

## 五、统一 RAG Provider 架构

```text
Agent
 ↓
rag Tool
 ↓
RAGService
 ↓
RAGPipeline
 ↓
LightRagPipeline
 ↓
LightRAG
```

核心职责：

```text
RAGPipeline
= 统一协议

RAGService
= 路由与结果归一化

LightRagPipeline
= LightRAG 具体实现

rag Tool
= Agent-facing Capability
```

Provider 按 Knowledge Base 绑定，而不是由 LLM 每次根据 Query 动态决定。

---

## 六、LightRAG 文档索引流程

```text
PDF / PPT / Word
       ↓
DeepTutor ParseService
       ↓
Parsed Document
       ↓
LightRAG Staging
       ↓
Chunk
       ↓
Entity / Relation Extraction
       ↓
Knowledge Graph + Vector Index
```

项目中 LightRAG 复用统一 Parse Layer，而不是自己再重做一遍 PDF/PPT 解析。

---

## 七、LightRAG 如何建图

课件原文：

```text
TCP 拥塞控制使用拥塞窗口 cwnd 和慢启动阈值 ssthresh 控制发送速率。

连接开始时进入 Slow Start，cwnd 指数增长。
当 cwnd 达到 ssthresh 后，TCP 进入 Congestion Avoidance，
此时 cwnd 改为线性增长。
```

可能抽取出 Entity：

```text
TCP Congestion Control
cwnd
ssthresh
Slow Start
Congestion Avoidance
```

再抽 Relationship：

```text
Slow Start
    ↓ increases
cwnd

cwnd
    ↓ compared with
ssthresh

ssthresh
    ↓ determines transition
Congestion Avoidance
```

最终形成 Graph：

```text
       TCP Congestion Control
                │
       ┌────────┴─────────┐
       ↓                  ↓
  Slow Start       Congestion Avoidance
       │                  ↑
       ↓                  │
      cwnd ── reaches ─→ ssthresh
```

图是 LLM 自动抽取构建的，因此 Graph Extraction Quality 会直接影响 Retrieval。

---

## 八、LightRAG 存哪些数据

| 数据 | 作用 |
|---|---|
| Document | 原始文件和 Metadata |
| Chunk | 原始 Evidence |
| Entity | 具体知识对象 |
| Relationship | 知识对象之间的关系 |
| Vector | 自然语言 Query 的语义入口 |
| Graph | 结构化关系扩展 |
| Source Mapping | 从 Entity / Relation 回到原始 Chunk |

最重要的三句话：

```text
Vector = 找谁
Graph  = 找关系
Chunk  = 给证据
```

---

## 九、三类 Vector

LightRAG 主要维护：

```text
chunks_vdb
entities_vdb
relationships_vdb
```

### Chunk Vector

```text
Chunk
↓ Embedding
chunks_vdb
```

用于原始文本语义检索。

### Entity Vector

```text
Entity Name + Description
↓ Embedding
entities_vdb
```

用于把自然语言 Query 映射到 Graph 中最相关的 Entity。

### Relationship Vector

```text
Relationship Description / Keywords
↓ Embedding
relationships_vdb
```

用于查找跨概念关系。

一句话：

> **Vector 是 Graph 的语义入口，Graph 是 Vector 命中后的结构扩展器。**

---

## 十、为什么最后还要回 Chunk

Entity 和 Relationship 是压缩后的知识结构，最终 Answer 还是应该尽量基于原始 Evidence。

因此需要：

```text
Entity / Relation
      ↓
Source Mapping
      ↓
Chunk
      ↓
Original Courseware
```

这样 Graph Retrieval 最后才能回到真实课件文本。

---

## 十一、LightRAG 查询模式

### naive

```text
Query
↓
Chunk Vector Search
```

记忆：

```text
naive = 找原文
```

### local

更 Entity-centric：

```text
Low-level Keywords
↓
Entity Vector Search
↓
Entity
↓
Graph Neighborhood
```

记忆：

```text
local = 找实体
```

### global

更 Relationship-centric：

```text
High-level Keywords
↓
Relationship Vector Search
↓
Global Graph Context
```

记忆：

```text
global = 找关系
```

### hybrid

```text
local + global
```

DeepTutor 当前默认使用：

```text
hybrid
```

### mix

```text
local
+
global
+
naive Chunk Retrieval
```

即：

```text
Graph Retrieval + Raw Chunk Retrieval
```

---

## 十二、模式速记表

| Mode | 主要检索对象 | 更适合 |
|---|---|---|
| naive | Chunk | 原文事实 |
| local | Entity | 具体知识点 |
| global | Relationship | 宏观关系 |
| hybrid | Entity + Relationship | 关系型问答 |
| mix | Entity + Relationship + Chunk | Graph + Vector 双保险 |

口诀：

```text
naive  找原文
local  找实体
global 找关系
hybrid 实体 + 关系
mix    再加原始 Chunk
```

---

## 十三、LightRAG 关键配置

| 参数 | 默认 | 含义 |
|---|---:|---|
| `search_mode` | hybrid | KB 查询模式 |
| `top_k` | 60 | Entity / Relation 候选预算 |
| `response_type` | Multiple Paragraphs | 回答形式 |
| `max_concurrent_files` | 1 | Parser Worker 并发 |
| `llm_model_max_async` | 4 | LightRAG LLM 最大并发 |
| `entity_extract_max_gleaning` | 1 | Entity/Relation 追加抽取次数 |
| `llm_profile_id` | 空 | 可选专用 LLM Profile |
| `llm_model_id` | 空 | 可选专用 Model |

### `top_k=60` 和 Recall@5 不是一回事

```text
LightRAG top_k
= 内部 Entity / Relationship Candidate Budget

Recall@5
= Evaluation 阶段前 5 条 Evidence 的召回指标
```

---

## 十四、`entity_extract_max_gleaning`

建图时一次 Entity / Relationship Extraction 可能有遗漏，因此可以：

```text
第一次 Extraction
↓
再次检查遗漏
↓
Gleaning
```

这个参数平衡：

```text
Extraction Recall
vs
LLM Cost / Latency
```

---

## 十五、`llm_model_max_async`

LightRAG 建图阶段包含大量 LLM 请求：

```text
Chunk
↓
Entity Extraction
↓
Relationship Extraction
↓
Merge / Summary
```

完全串行太慢，无限并发又容易 429 / Rate Limit。

所以：

```text
llm_model_max_async = 4
```

属于：

```text
Indexing Throughput
vs
Provider Stability
```

的工程平衡。

---

## 十六、统一 LLM / Embedding Adapter

```text
DeepTutor Unified Model Config
          │
    ┌─────┴─────┐
    ↓           ↓
LLM Client  Embedding Client
    │           │
    └─────┬─────┘
          ↓
  LightRAG Adapter
          ↓
      LightRAG SDK
```

LightRAG 不再单独维护另一套模型和 Embedding 配置。

如果配置专用：

```text
llm_profile_id
llm_model_id
```

LightRAG 可以单独使用指定模型，同时普通 Chat 的 Active Model 不受影响。

---

## 十七、Embedding Model 不能随便换

```text
Indexing Embedding Model
必须和
Query Embedding Model
保持一致
```

如果更换 Embedding：

```text
Chunk
Entity
Relationship
```

都应该重新 Embedding。

一句话：

> **Embedding Model 是 Index Schema 的一部分。**

---

## 十八、结果归一化与 Source 回填

上层统一希望拿到：

```text
query
answer
content
sources
provider
```

RAG Tool 最后可理解为：

```text
Normalized RAG Result
        │
 ┌──────┼────────┐
 ↓      ↓        ↓
content sources metadata
 ↓      ↓        ↓
LLM   Citation  Trace
```

### content
给 Agent 继续推理。

### sources
保留 Retrieval Provenance。

### metadata
保留 Provider、Mode、完整 Retrieval Result、错误状态等信息。

---

## 十九、什么叫可追溯问答

例如：

```text
Question:
为什么 cwnd 达到 ssthresh 后进入拥塞避免？
```

LightRAG 可以返回：

```text
Entity:
cwnd
ssthresh
Congestion Avoidance

Relationship:
phase transition

Source:
计算机网络.pdf
Page: xx
Chunk: chunk_xxx
```

最终：

```text
Grounded Context
↓
LLM Answer

+

Sources
↓
UI / Citation
```

当前更准确的说法是：

> **Retrieval-level Provenance**

不要夸大成：

> 每一句生成文本都 100% 精确对应某一个 Chunk。

---

## 二十、测试集口径

### 大学部分

来源：

```text
大学课程课件
```

整理：

```text
Question
Ground Truth Answer
Ground Truth Evidence
```

Evidence 尽量标到：

```text
File
Page
Chunk
```

### 中小学部分

来源：

> **中小学近十年的期末考试真题**

将题目作为 Query，标准答案或人工核验答案作为 Ground Truth Answer，并标注对应知识证据作为 Ground Truth Evidence。

### 注意数据泄漏

测试真题及答案不要同时放进被检索 KB，否则 Retrieval 会直接搜到原题答案，Recall 和 F1 会虚高。

---

## 二十一、Recall@5

Recall@5 评价：

> Retriever 有没有找到正确 Evidence。

例如：

```text
Ground Truth:
chunk_102
```

Top-5：

```text
1. chunk_85
2. chunk_102  ← hit
3. chunk_190
4. chunk_53
5. chunk_211
```

则这条 Query 被正确召回。

当前口径：

```text
Recall@5 = 91%
```

---

## 二十二、F1

F1 评价：

> Generated Answer 和 Ground Truth Answer 的覆盖程度。

公式：

```text
F1
=
2 × Precision × Recall
/
(Precision + Recall)
```

当前：

```text
F1 = 78%
```

F1 不是 Accuracy，还会受同义改写、表达方式和 Tokenization 影响。

---

## 二十三、Recall 91% 但 F1 78% 为什么合理

```text
Retriever 找到正确 Evidence
        ↓
不代表
        ↓
Generator 一定完全答对
```

即使 Evidence 已经在 Top-5 中，仍可能：

- LLM 没利用好 Evidence
- Evidence 不完整
- 多 Chunk 组合不好
- 回答表达与 Ground Truth 不一致
- 出现额外无关内容

因此：

```text
Recall@5
= Retrieval Metric

F1
= Generation Metric
```

---

## 二十四、如何做 Error Attribution

如果某道题答错：

```text
Relevant Evidence in Top-5?
```

### 没有
偏 Retrieval 问题：

```text
Chunking
Embedding
Query Rewrite
Mode
Top-K
Graph Extraction
Reranking
```

### 有，但答案仍错
偏 Generation 问题：

```text
Prompt
Context Formatting
Evidence Selection
LLM Reasoning
Hallucination
```

一句话：

> **不要拿 Generation 优化去掩盖 Retrieval 问题。**

---

## 二十五、当前向量存在哪里

当前本地 LightRAG 默认：

```text
vector_storage = NanoVectorDBStorage
```

三类：

```text
chunks_vdb
entities_vdb
relationships_vdb
```

运行时主要在内存，持久化为：

```text
vdb_<namespace>.json
```

所以：

> **LightRAG Vector 默认不存 SQLite。**

其他默认存储可简化成：

| 数据 | 默认存储 |
|---|---|
| Chunk / Entity / Relation Metadata | JSON KV |
| Chunk Vector | NanoVectorDB / JSON |
| Entity Vector | NanoVectorDB / JSON |
| Relationship Vector | NanoVectorDB / JSON |
| Knowledge Graph | NetworkX / GraphML |
| Doc Status | JSON |

---

## 二十六、为什么当前不用 Milvus / Qdrant / pgvector

当前产品首先是：

```text
Local AI Learning Partner
```

优先考虑：

```text
低部署成本
低运维成本
本地运行
单用户 / 中小规模 KB
```

NanoVectorDB：

```text
无需额外服务
无需网络访问
安装简单
开发调试方便
```

因此第一阶段复杂度最低。

---

## 二十七、NanoVectorDB 的扩展瓶颈

### 1. Memory

例如：

```text
1,000,000 vectors
× 1024 dim
× float32
≈ 4 GB raw vector data
```

还没算 Metadata / Index。

### 2. File Persistence

大 JSON 的 Reload / Serialize / Write 成本会上升。

### 3. Concurrency

当前更偏：

```text
Single Writer
+
Multiple Readers
```

不适合大型多用户在线服务。

---

## 二十八、未来存储演进

```text
Stage 1
Local / Desktop
↓
NanoVectorDB
```

```text
Stage 2
School Server / Medium SaaS
↓
Qdrant
或
PostgreSQL + pgvector
```

```text
Stage 3
Large-scale Vector Service
↓
Milvus
```

简单选型：

| 场景 | 更合适 |
|---|---|
| 本地 Demo / Desktop | NanoVectorDB |
| 中型 RAG Server | Qdrant / pgvector |
| 已大量使用 PostgreSQL | pgvector |
| Vector 服务独立伸缩 | Qdrant |
| 超大规模、高吞吐 | Milvus |

---

## 二十九、换 Vector DB 后 Agent 要不要改

不需要。

```text
Agent
↓
RAG Tool
↓
RAGService
↓
LightRagPipeline
↓
LightRAG
↓
BaseVectorStorage
       │
 ┌─────┼───────────┐
 ↓     ↓           ↓
Nano Qdrant      Milvus
      PGVector
```

所以：

> RAG Provider 抽象隔离 RAG Engine；LightRAG 的 Storage Abstraction 又进一步隔离存储后端。

---

## 三十、换 Vector Store 和换 Embedding 不一样

### 换 Storage

```text
Nano
↓
Qdrant
```

Embedding Model 不变。

### 换 Embedding

```text
Old Embedding
↓
New Embedding
```

Vector Space 变化，需要重新计算 Chunk / Entity / Relationship Vector。

---

## 三十一、RAG Index 是 Derived Data

推荐把：

```text
Original Source Documents
```

视为：

```text
Source of Truth
```

而：

```text
Chunk
Vector
Entity
Relationship
Graph
```

都是：

```text
Derived Data
```

当 Storage、Embedding 或 Index Schema 大幅变化时，可以从 Source Documents 重建索引。

---

# 三十二、具体知识库示例：计算机网络课程 KB

> **以下数字是面试叙事参考量级，不是当前项目的真实实测数据。实际面试时如果你有真实统计，请替换。不要把这些数字描述成线上真实数据。**

假设一个：

```text
Knowledge Base:
Computer Networks
```

资料量：

```text
20~30 份 PDF / PPT
约 1,000~1,500 页课件
```

为了讲量级，可以使用：

```text
Documents
≈ 25

Pages
≈ 1,200

Chunks
≈ 6,000

Merged Entities
≈ 8,000

Relationships
≈ 12,000
```

对应 Vector：

```text
Chunk Vectors
≈ 6,000

Entity Vectors
≈ 8,000

Relationship Vectors
≈ 12,000
```

总 Vector：

```text
≈ 26,000
```

即：

> **一个课程知识库可以是几万级 Vector。**

---

## 三十三、为什么 1,200 页会产生约 26,000 个 Vector

不是：

```text
1 Page = 1 Vector
```

而是：

```text
1,200 Pages
↓
约 6,000 Chunks
↓
Entity Extraction
↓
约 8,000 Merged Entities
↓
Relationship Extraction
↓
约 12,000 Relations
```

最终：

```text
6,000
+
8,000
+
12,000
=
26,000 vectors
```

LightRAG 相比纯 Chunk RAG，会因为 Entity 和 Relationship 两层索引进一步扩大存储量。

---

## 三十四、这个量级为什么 NanoVectorDB 仍合理

假设 Embedding：

```text
1024 dim
float32
```

单个 Vector 原始大小约：

```text
1024 × 4 bytes
≈ 4 KB
```

26,000 个 Vector：

```text
26,000 × 4 KB
≈ 104 MB
```

这里只是非常粗略的 Raw Vector 大小估算，还没有包含：

```text
Metadata
JSON Serialization
Index
Python / Numpy Overhead
```

但从数量级看：

```text
几万级 Vector
```

仍然适合本地 NanoVectorDB。

---

## 三十五、面试里怎么讲这个知识库例子

推荐：

> 以一个计算机网络课程知识库为例，如果有二三十份 PDF/PPT，大概一千多页课件，解析后可能形成几千个 Chunk。LightRAG 不只对 Chunk 做 Embedding，还会抽取并合并 Entity 和 Relationship，所以一个几千 Chunk 的 KB 最终可能是几万级 Vector，比如 Chunk、Entity 和 Relationship 加起来大概两三万这个数量级。
>
> 这个规模下，本地 NanoVectorDB 的部署成本最低，所以当前没有必要直接引入 Milvus。真正到了学校服务器或者 SaaS 场景，一个实例承载大量课程、多用户同时建库，Vector 进入百万级甚至更高，再考虑 Qdrant、pgvector 或 Milvus。

注意使用：

```text
“可能”
“数量级”
“例如”
```

而不是：

```text
“我们线上就是 26,000”
```

除非你真的统计过。

---

## 三十六、知识库 QA 示例

以计算机网络 KB 为例：

```text
Question:
为什么 cwnd 达到 ssthresh 后进入 Congestion Avoidance？
```

Ground Truth：

```text
因为 Slow Start 阶段 cwnd 指数增长，
当达到慢启动阈值后，
TCP 切换到 Congestion Avoidance，
以更保守的速度继续增加窗口。
```

Ground Truth Evidence：

```text
计算机网络.pdf
Chapter: TCP Congestion Control
Page: xx
Chunk: chunk_xxx
```

运行：

```text
Question
↓
LightRAG hybrid
↓
Entity:
cwnd
ssthresh
Congestion Avoidance

Relationship:
phase transition
↓
Source Chunk
↓
Answer
```

评测：

```text
Top-5 Evidence
vs
Ground Truth Evidence
↓
Recall@5

Generated Answer
vs
Ground Truth Answer
↓
F1
```

---

## 三十七、中小学考试题示例

初中物理：

```text
一个质量为 2kg 的物体受到 10N 合力，
求物体加速度。
```

Query：

```text
题目本身
```

知识 Evidence：

```text
Newton's Second Law
F = ma
```

Graph：

```text
Force
       Newton's Second Law
    /
   /
Mass
 │
 ↓
Acceleration
```

Retriever 找：

```text
Entity:
Force
Mass
Acceleration

Relationship:
F = ma
```

最后回到 Chunk：

```text
牛顿第二定律：
物体的加速度与所受合外力成正比，
与质量成反比，F=ma。
```

LLM：

```text
a = F / m
  = 10 / 2
  = 5 m/s²
```

这里：

```text
Recall@5
```

主要评价牛顿第二定律 Evidence 是否找回来；

```text
F1
```

还会受到 LLM 计算和推理能力影响。

---

## 三十八、完整架构图

```text
                    Private Courseware
                           │
                           ↓
                     ParseService
                           │
                           ↓
                         Chunk
                           │
               ┌───────────┴───────────┐
               ↓                       ↓
        Entity Extraction      Relationship Extraction
               │                       │
               ↓                       ↓
            Entities               Relations
               │                       │
               └───────────┬───────────┘
                           ↓
                    Knowledge Graph

同时：

Chunk ───────────────→ Chunk Vector
Entity ──────────────→ Entity Vector
Relationship ────────→ Relation Vector

                           │
                           ↓
                     LightRAG Query
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
           local         global        naive
           Entity       Relation       Chunk
             └─────────────┬─────────────┘
                           ↓
                  hybrid / mix
                           ↓
                 Grounded Context
                           +
                       Sources
                           ↓
                      RAG Tool
                           ↓
                         Agent
```

---

## 三十九、完整面试回答

> 项目原本已经有统一的 RAG Provider 框架，我主要负责把 LightRAG 接入现有 `RAGPipeline / RAGService`。文档首先走统一 Parse Layer，解析结果再进入 LightRAG；LightRAG 对 Chunk 做 Entity 和 Relationship Extraction，构建 Knowledge Graph，同时维护 Chunk、Entity、Relationship 三类 Vector Index。
>
> 查询时，local 更偏 Entity-centric Retrieval，global 更偏 Relationship-centric，hybrid 会把两者结合；另外还保留 naive Chunk Retrieval，mix 模式可以同时使用 Graph 和原始 Chunk Vector Search。我的理解是 Vector 主要负责从自然语言 Query 找到图中的语义入口，Graph 负责结构关系扩展，最后再通过 Source Mapping 回到原始 Chunk，把真实课件作为 Grounded Evidence。
>
> 工程接入上，我们没有让 LightRAG 再维护独立的 LLM 和 Embedding 配置，而是通过 Adapter 复用项目统一 Model Client。Provider 最终返回的正文、Sources 和 Metadata 会再做归一化，正文进入 `ToolResult.content`，Retrieval Provenance 放在 `ToolResult.sources`，所以回答能够回溯到本地课件。
>
> 评测集来自大学课程课件和中小学近十年的期末考试真题。我把 Retrieval 和 Generation 分开评：Recall@5 看正确 Evidence 是否进入前五，F1 看最终 Answer 和 Ground Truth Answer 的覆盖程度，当前测试集平均 Recall@5 91%，F1 78%。
>
> 存储方面，我们当前本地版使用 LightRAG 默认的 NanoVectorDB，分别保存 Chunk、Entity、Relationship Vector。以一个计算机网络课程 KB 为例，二三十份课件、一千多页资料，经过 Chunk 和图谱抽取后可能形成两三万级 Vector，这个量级下本地 NanoVectorDB 的复杂度最低。以后如果演进到学校服务器或 SaaS，多用户、百万级以上 Vector 和高并发场景下，再考虑 Qdrant、PGVector 或 Milvus，而不需要修改 Agent 和上层 RAG Tool。

---

## 四十、最终速记

```text
LightRAG
= Knowledge Graph + Vector Retrieval
```

```text
Vector = 找谁
Graph  = 找关系
Chunk  = 给证据
```

```text
naive  = Chunk
local  = Entity
global = Relationship
hybrid = Entity + Relationship
mix    = Graph + Chunk
```

```text
Recall@5
= 找没找到正确证据

F1
= 最终答案和标准答案有多接近
```

```text
Current Storage
= NanoVectorDB

Scale-out
= Qdrant / PGVector / Milvus
```

面试最核心总结：

> **我主要做的不是“调用一下 LightRAG”，而是把 LightRAG 作为一个可替换的 RAG Provider 接入统一 Agent Runtime，解决文档解析、模型与 Embedding 适配、Graph + Vector Retrieval、Source Provenance、评测与后续存储扩展之间的工程问题。**
