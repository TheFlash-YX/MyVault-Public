# MyVault 秋招面试知识库改造——最终确定版实施方案

## 0. 文档状态

本方案为 **最终确定版实施规范**，用于指导 MyVault 中 AI 应用开发工程师 / AI Agent 研发工程师秋招面试题库的迁移、拆分、索引、去重与长期维护。

本方案已经锁定：

* 目录结构
* interview frontmatter schema
* canonical question 标识规范
* source_id 与 canonical_key 的映射关系
* migration ledger 数据结构
* 跨模块去重规则
* notes / interview / resume / interview-plan 的职责边界
* CLAUDE.md 修改内容
* 模板修改规则
* consolidate skill 同步规则
* 迁移阶段
* Git 安全约束
* 验收规则

在用户明确授权以前：

> **禁止修改 MyVault 的任何目录结构、CLAUDE.md、模板、index、consolidate skill 或正式 interview 题集。**

---

# 1. 当前仓库事实基线

## 1.1 interview 当前状态

当前实际存在：

```text
interview/
├── agents/
│   └── .gitkeep
├── backend/
│   └── .gitkeep
├── frontend/
│   └── .gitkeep
└── java/
    └── .gitkeep
```

当前不存在任何正式 `.md` 面试题集。

因此：

* 当前迁移不存在与旧面试题集正文冲突的问题；
* 仍然必须执行全局查重规则；
* 后续新增题集时无需进行旧题答案合并。

---

## 1.2 当前可直接复用的 notes

已确认存在以下相关知识笔记：

```text
notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md

notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md

notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md

notes/python/asyncio-coroutines-and-generators.md

notes/backend/database/optimistic-locking.md

notes/backend/architecture/annotation-probe-quality-loop.md

notes/backend/architecture/cache-and-async-task-alternatives.md
```

这些文件后续允许和相关 interview 题集建立双向链接。

---

## 1.3 当前缺失的长期知识主题

当前没有发现以下主题对应的现成知识笔记：

* MCP
* Agent Memory
* 通用 SSE
* Agent Evaluation
* MySQL 系统知识
* Agent Security
* Tool Calling 系统化知识
* FastAPI 系统化知识
* Redis 系统化知识
* Kafka 系统化知识
* Docker / Production 系统化知识

处理原则：

> **迁移面试题时，不为了建立知识图谱而自动创建缺失 notes。**

若某题没有对应长期知识笔记：

* interview 中保留题目；
* “关联笔记”章节不创建悬空链接；
* 后续真正学习该主题时，再通过主题笔记流创建对应 notes；
* 新 notes 建立后，再补双向链接。

---

# 2. 原始题库事实基线

原始题库（归档于 `interview/meta/source/`）：

```text
interview/meta/source/autumn-2027-question-bank.md
```

实际状态：

```text
822 行
27 个章节
Q1 ～ Q725
编号连续
共 725 道原始问题
```

该文件目前只包含：

* 面试问题
* 模块划分
* 少量优先级信息

不包含：

* 完整参考答案
* 项目实现证据
* 指标统计证据
* 技术设计原始代码
* 可直接认定为真实的项目结论

因此迁移阶段：

> **禁止根据题目反向“编造”项目答案、实现细节、数据口径或指标证据。**

原始题库只作为：

```text
source dataset
```

以及后续全量迁移验收的：

```text
coverage baseline
```

---

# 3. 最终知识库分层模型

MyVault 采用以下四层结构：

```text
长期知识层
notes/
│
│ 解决：这个知识是什么、为什么、如何实现
│
↓
通用面试题层
interview/{technical-area}/
│
│ 解决：面试官会如何问这个技术
│
↓
个人简历答辩层
interview/resume/
│
│ 解决：我的项目为什么这么做、我具体做了什么
│
↓
秋招状态层
interview-plan/
│
│ 解决：我现在该准备什么、哪里薄弱
│
↓
面试执行
```

核心约束：

> `notes` 管知识，`interview` 管表达，`resume` 管个人项目真实性，`interview-plan` 管短期状态。

不得在四层之间复制同一份完整内容。

---

# 4. 最终目录树

最终使用以下目录结构：

```text
MyVault/
├── CLAUDE.md
├── AGENTS.md
├── index.md
│
├── notes/
│   ├── agents/
│   ├── backend/
│   ├── frontend/
│   ├── algorithms/
│   ├── python/
│   └── ...
│
├── interview/
│   ├── index.md
│   │
│   ├── meta/
│   │   └── autumn-2027-migration-map.md
│   │
│   ├── agents/
│   │   ├── agent-runtime.md
│   │   ├── human-in-the-loop.md
│   │   ├── streaming-sse.md
│   │   ├── function-calling-and-tools.md
│   │   ├── mcp.md
│   │   ├── memory.md
│   │   ├── agent-multi-tenancy.md
│   │   ├── rag.md
│   │   ├── rag-frameworks-and-providers.md
│   │   ├── prompt-and-agent-security.md
│   │   ├── agent-evaluation.md
│   │   ├── ai-coding-engineering.md
│   │   ├── agent-system-design.md
│   │   ├── agent-troubleshooting.md
│   │   └── agent-architecture-and-tradeoffs.md
│   │
│   ├── backend/
│   │   ├── mysql.md
│   │   ├── transactions-locking-and-idempotency.md
│   │   ├── spring-boot.md
│   │   ├── redis.md
│   │   ├── kafka-and-messaging.md
│   │   └── docker-and-production.md
│   │
│   ├── java/
│   │   └── collections-and-concurrency.md
│   │
│   ├── python/
│   │   ├── fastapi.md
│   │   └── asyncio-and-concurrency.md
│   │
│   ├── frontend/
│   │
│   └── resume/
│       ├── kindpartner.md
│       ├── kindvideo.md
│       ├── resume-metrics.md
│       ├── behavioral.md
│       └── motivation.md
│
├── interview-plan/
│   ├── autumn-2027.md
│   ├── high-priority.md
│   └── mock-interview-log.md
│
├── daily/
├── pitfalls/
├── resources/
│
└── templates/
    ├── note.md
    ├── daily.md
    ├── pitfall.md
    ├── resource.md
    ├── interview.md
    ├── interview-plan.md
    └── mock-interview-entry.md
```

---

# 5. 各目录最终职责

## 5.1 notes

`notes/` 是长期知识的 Single Source of Truth。

包含：

* 原理
* 完整技术实现
* 设计取舍
* 边界条件
* 示例
* 踩坑后的稳定结论

禁止因为一道面试题存在就机械创建 notes。

---

## 5.2 interview/agents、backend、java、python

负责通用面试题。

每道题重点保存：

* 面试问题
* 考察点
* 60～90 秒回答框架
* 易追问
* 已存在的相关知识笔记

不复制完整知识教程。

---

## 5.3 interview/resume

只保存和个人经历强绑定的问题。

例如：

```text
为什么设计 ask_user？
UnifiedContext 为什么这么设计？
98% 如何统计？
7000+ 探针如何产生？
为什么使用乐观锁？
```

这里允许引用通用 interview 题集。

但不得把通用知识重新复制一遍。

---

## 5.4 interview/meta

只用于面试题迁移和治理。

不属于用户日常刷题入口。

当前唯一正式文件：

```text
interview/meta/autumn-2027-migration-map.md
```

---

## 5.5 interview-plan

只保存：

* 当前准备阶段
* 模块掌握度
* 薄弱项
* 高频复习链接
* 模拟面试记录
* 下一步动作

禁止长期保存：

* 完整面试答案
* 完整技术原理
* 项目答辩正文

---

# 6. interview frontmatter 最终 Schema

不使用 `domain / domains` 管理 interview 文件。

`domain` 继续属于 notes 知识体系。

interview 采用独立 schema。

---

## 6.1 通用技术题集

统一格式：

```yaml
---
type: interview
scope: topic
area: agents
topics:
  - agent-runtime
  - function-calling
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

字段定义：

### type

固定：

```yaml
type: interview
```

### scope

通用技术题固定：

```yaml
scope: topic
```

### area

必须与物理目录一致。

合法值：

```text
agents
backend
java
python
frontend
resume
```

例如：

```text
interview/agents/mcp.md
```

必须：

```yaml
area: agents
```

### topics

用于表达文件关联的知识主题。

要求：

* kebab-case
* 1～8 个
* 只用于标签与跨主题检索
* 不参与文件物理路由

---

## 6.2 简历项目题集

格式：

```yaml
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
  - rag
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

### project

合法值当前为：

```text
kindpartner
kindvideo
```

非项目型简历文件不写 `project`。

---

## 6.3 行为面

例如：

```yaml
---
type: interview
scope: resume
area: resume
category: behavioral
topics:
  - internship
  - collaboration
  - problem-solving
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

---

## 6.4 求职动机

```yaml
---
type: interview
scope: resume
area: resume
category: motivation
topics:
  - career
  - ai-agent
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

---

## 6.5 resume_version

只出现在与当前简历内容强绑定的题集。

当前版本：

```yaml
resume_version: 2026-08
```

以后简历发生实质变化时更新。

历史问题不因为简历版本变化自动删除。

---

# 7. canonical_key 最终规范

每一道迁移后的正式面试题必须有一个永久稳定的 `canonical_key`。

canonical_key 不使用题目编号。

---

## 7.1 格式

统一格式：

```text
{namespace}.{topic}.{slug}
```

例如：

```text
agents.runtime.loop-termination
agents.runtime.prevent-infinite-loop

agents.hitl.same-turn-resume
agents.hitl.async-queue

agents.streaming.sse-vs-websocket

agents.mcp.mcp-vs-function-calling

python.asyncio.multi-worker-state

backend.locking.optimistic-vs-pessimistic

resume.kindpartner.ask-user-design
resume.kindpartner.same-turn-recovery
resume.kindvideo.probe-idempotency
resume.metrics.kindpartner-resume-success-rate
```

---

## 7.2 namespace

合法 namespace：

```text
agents
backend
java
python
frontend
resume
```

---

## 7.3 topic

使用稳定的主题短名。

例如：

```text
runtime
hitl
streaming
tools
mcp
memory
rag
evaluation
security
asyncio
fastapi
mysql
locking
spring
redis
kafka
production
kindpartner
kindvideo
metrics
behavioral
motivation
```

---

## 7.4 slug

规则：

* 全小写
* kebab-case
* 只表达语义
* 不包含 Q 编号
* 不包含日期
* 不包含“question”等无意义词
* 一旦生成原则上不修改

例如：

```text
same-turn-resume
sse-vs-websocket
multi-worker-state
optimistic-lock-version-conflict
```

---

# 8. canonical_key 在文件中的位置

每道题使用：

```markdown
## 为什么选择 SSE 而不是 WebSocket？

<!-- canonical_key: agents.streaming.sse-vs-websocket -->
```

要求：

* `canonical_key` 紧跟题目标题；
* 同一文件内唯一；
* 整个 `interview/` 全局唯一；
* 不放在 YAML frontmatter；
* 不用 Q1/Q2 作为永久标识。

显示层可以按当前顺序编号，但编号只用于阅读。

---

# 9. source_id 与 canonical_key 的关系

## source_id

指原始 725 题中的编号：

```text
Q1
Q2
...
Q725
```

它代表：

> 这道问题来自原始题库的哪个位置。

---

## canonical_key

代表：

> 迁移完成后，这个知识库中的规范面试题是谁。

---

二者关系：

```text
source question
        ↓
source_id
        ↓
migration ledger
        ↓
canonical_key
        ↓
canonical interview question
```

source_id 永远不替代 canonical_key。

canonical_key 也不承担原题追踪职责。

---

# 10. Migration Ledger 最终规范

正式路径：

```text
interview/meta/autumn-2027-migration-map.md
```

该文件是 725 题迁移的唯一审计账本。

---

## 10.1 Ledger 字段

使用 Markdown Table：

```markdown
| source_id | source_module | disposition | canonical_file | canonical_key | merged_with | note |
|---|---|---|---|---|---|---|
```

字段含义：

### source_id

```text
Q1 ～ Q725
```

必须每个 source_id 恰好出现一次。

---

### source_module

原题库中的原章节名称。

---

### disposition

合法值只有：

```text
moved
merged
split
excluded
```

不得增加其他值。

---

### canonical_file

目标正式题集路径。

例如：

```text
interview/agents/human-in-the-loop.md
```

若 disposition 为 `excluded`，为空。

---

### canonical_key

最终规范题的 canonical_key。

如果一个原题拆为多道规范题：

```text
key-a; key-b
```

---

### merged_with

只有 disposition=`merged` 时使用。

填写主 source_id。

例如：

```text
Q105
```

意味着当前题被合并进 Q105 所对应的规范问题。

---

### note

只用于记录：

* 合并原因
* 拆分原因
* 排除原因
* 特殊迁移说明

不写答案。

---

# 11. 四种 disposition 最终语义

## 11.1 moved

原题直接迁移为一条规范问题。

例如：

```text
Q102
→ agents.streaming.sse-vs-websocket
```

---

## 11.2 merged

原题和另一个原题语义基本相同。

当前 source 不创建新的规范题。

例如：

```text
Q102 为什么用 SSE 而不用 WebSocket？
Q135 SSE 和 WebSocket 怎么选？
```

处理：

```text
Q102 → moved
Q135 → merged → Q102
```

二者最终都指向：

```text
agents.streaming.sse-vs-websocket
```

---

## 11.3 split

一个原题实际上包含多个独立考察点。

例如：

```text
Tool 如何做 timeout、retry 和 circuit breaker？
```

如果迁移后确定需要形成多个独立规范问题，则：

```text
disposition: split
canonical_key:
agents.tools.tool-timeout;
agents.tools.tool-retry;
agents.tools.circuit-breaker
```

split 应谨慎使用。

---

## 11.4 excluded

原题不进入正式题集。

只允许以下情况：

* 明确超出当前岗位范围；
* 无实际面试价值；
* 完全依赖已经删除的简历内容；
* 问题本身有明显错误且不值得保留；
* 无法形成独立问题且也不应 merge。

必须在 `note` 写排除原因。

---

# 12. 三种重复问题的最终处理规则

## 12.1 Exact Duplicate

定义：

> 两个问题实际考察的是同一个知识点，只有措辞差异。

例如：

```text
为什么 SSE 不用 WebSocket？
SSE 和 WebSocket 为什么选 SSE？
```

处理：

* 只生成一个 canonical question；
* 第一条 source 使用 `moved`；
* 其余使用 `merged`；
* merged source 指向同一个 canonical_key。

---

## 12.2 Semantic Overlap

定义：

> 使用相同知识，但面试考察语境不同。

例如：

```text
SSE 和 WebSocket 有什么区别？
```

与：

```text
你的 ask_user 为什么使用 SSE 而不是 WebSocket？
```

处理：

两题都保留。

通用题：

```text
interview/agents/streaming-sse.md
```

项目题：

```text
interview/resume/kindpartner.md
```

项目题通过 Markdown 链接关联通用题。

不得因为知识重叠而合并。

---

## 12.3 Contextual Duplicate

定义：

> 同一底层知识在不同系统场景中被调用。

例如：

```text
FastAPI 多 worker 后内存是否共享？
```

和：

```text
你的 ask_user 如果部署多个 worker 会发生什么？
```

处理：

保留两题。

底层规范题：

```text
interview/python/asyncio-and-concurrency.md
```

项目应用题：

```text
interview/resume/kindpartner.md
```

项目题不复制 Python 原理答案。

---

# 13. 一个问题只能有一个 canonical topic 归属

通用题必须有唯一正式归属文件。

例如：

```text
多 worker 下 asyncio.Queue 为什么有问题？
```

规范归属：

```text
interview/python/asyncio-and-concurrency.md
```

不能同时完整存在于：

```text
agent-runtime.md
human-in-the-loop.md
fastapi.md
```

其他题集需要提及时，只保留：

* 链接
* 场景化问题
* 易追问

---

# 14. resume-metrics.md 最终职责

路径：

```text
interview/resume/resume-metrics.md
```

是所有简历量化数据的唯一口径文件。

包括：

* 7000+
* 100+
* 35%
* 1000+
* 98%
* 0%
* 100%
* 78%
* 91%

每个指标模板：

```markdown
## 指标名称

<!-- canonical_key: resume.metrics.xxx -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-
```

如果目前只有问题，没有真实统计证据：

> 保留字段为空，不补造答案。

其他 resume 文件只链接这里。

---

# 15. 题集正文最终格式

通用 interview 题：

```markdown
## 为什么选择 SSE 而不是 WebSocket？

<!-- canonical_key: agents.streaming.sse-vs-websocket -->

### 考察点

- SSE 与 WebSocket 的通信模型
- LLM Streaming 场景
- 工程复杂度
- 断线恢复

### 回答框架

1. 先说明两者核心区别
2. 再结合当前业务通信方向
3. 解释工程成本
4. 补充不适用 SSE 的场景

### 易追问

- SSE 如何重连？
- 多实例如何恢复事件？
- 什么情况下应该使用 WebSocket？

### 关联笔记

-
```

---

# 16. 不维护逐题掌握状态

正式决定：

> **interview 题集不维护 🔴 / 🟡 / 🟢 掌握状态。**

原因：

* 状态是短期信息；
* 面试题是长期内容；
* 725 题逐题状态维护成本过高；
* 状态会快速过期。

---

## 16.1 autumn-2027.md

管理模块级状态：

```text
🔴 不熟
🟡 能回答但不稳
🟢 可以稳定回答
```

---

## 16.2 high-priority.md

只存：

```text
问题短标题 + 链接 + 当前复习原因
```

不得复制答案。

---

## 16.3 interview 内的高频标记

不使用用户学习状态标记。

只有极少数结构性核心题允许：

```text
priority: core
```

但初次迁移阶段不引入该字段。

---

# 17. interview/index 与根 index 最终结构

## 17.1 根 index.md

只新增两个入口：

```markdown
## 面试准备

- [面试题库](../../interview/index.md)
- [2027 秋招 Dashboard](../../interview-plan/autumn-2027.md)
```

不得在根 index 列出所有题集。

---

## 17.2 interview/index.md

完整管理：

```text
Agents
Backend
Java
Python
Frontend
Resume
```

每个正式题集只在这里出现一次。

`interview/meta/` 不作为普通刷题入口。

---

# 18. 现存 notes 回链规则

以下已确认 notes 在迁移时可建立关系。

---

## 18.1 Agent Runtime

```text
notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md
```

关联：

```text
interview/agents/agent-runtime.md
interview/agents/human-in-the-loop.md
interview/resume/kindpartner.md
```

---

## 18.2 Agent Loop / Tool Protocol

```text
notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md
```

关联：

```text
interview/agents/agent-runtime.md
interview/agents/function-calling-and-tools.md
interview/agents/human-in-the-loop.md
interview/resume/kindpartner.md
```

---

## 18.3 RAG Provider

```text
notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md
```

关联：

```text
interview/agents/rag.md
interview/agents/rag-frameworks-and-providers.md
interview/resume/kindpartner.md
```

---

## 18.4 asyncio

```text
notes/python/asyncio-coroutines-and-generators.md
```

关联：

```text
interview/python/asyncio-and-concurrency.md
interview/agents/human-in-the-loop.md
interview/resume/kindpartner.md
```

---

## 18.5 乐观锁

```text
notes/backend/database/optimistic-locking.md
```

关联：

```text
interview/backend/transactions-locking-and-idempotency.md
interview/resume/kindvideo.md
```

---

## 18.6 标注探针

```text
notes/backend/architecture/annotation-probe-quality-loop.md
```

关联：

```text
interview/resume/kindvideo.md
```

---

## 18.7 Redis / MQ 替代方案

```text
notes/backend/architecture/cache-and-async-task-alternatives.md
```

关联：

```text
interview/backend/redis.md
interview/backend/kafka-and-messaging.md
interview/agents/human-in-the-loop.md
```

---

# 19. 缺失 notes 的最终处理

MCP、Memory、SSE、Agent Evaluation、MySQL 等当前没有对应 notes。

迁移时：

```text
interview 题正常创建
```

“关联笔记”保持：

```markdown
### 关联笔记

-
```

或不写链接。

禁止：

```text
为了面试题自动创建 notes/agents/mcp.md
```

以后真正学习 MCP 时：

```text
主题笔记流
↓
查重
↓
新建 notes
↓
再补 interview ↔ notes 双向链接
```

---

# 20. CLAUDE.md 最终修改内容

正式实施时必须修改以下规则。

---

## 20.1 项目结构

新增：

```text
interview/python/
interview/resume/
interview/meta/
interview-plan/
templates/interview-plan.md
templates/mock-interview-entry.md
```

---

## 20.2 面试题领域规则

原 interview 领域路由升级为：

```text
agents
backend
java
python
frontend
resume
```

---

## 20.3 scope 路由

记题流第一步先判断：

```text
scope: topic
或
scope: resume
```

规则：

```text
通用知识/八股/系统设计
→ scope: topic

“你的项目”“你简历”“你这个指标”
→ scope: resume
```

---

## 20.4 frontmatter

明确规定 interview 使用：

```text
type
scope
area
topics
```

resume 可额外使用：

```text
project
category
resume_version
```

禁止使用：

```text
domain
domains
```

作为 interview 路由字段。

---

## 20.5 记题流查重

修改为：

```text
第一步：全局 semantic duplicate check
第二步：判断 Exact / Semantic / Contextual
第三步：确定唯一 canonical file
第四步：检查 canonical_key 是否已存在
第五步：再新增或更新
```

不得只在目标目录内查重。

---

## 20.6 canonical_key

加入全局规则：

> 所有正式 interview question 必须拥有全局唯一 canonical_key。

---

## 20.7 原题库批量迁移规则

加入：

> 批量导入必须维护 migration ledger，任何 source_id 必须记录 disposition。

---

## 20.8 interview-plan

不加入基础 MyVault 强制结构探测。

只有：

```text
秋招
面试计划
模拟面试
高频复习
```

相关工作流才检查该目录。

---

## 20.9 链接规则

补充：

```text
interview/resume → interview/agents/backend/python
interview-plan → interview
interview-plan → notes
notes → interview/resume
```

全部继续使用标准 Markdown 相对链接。

---

## 20.10 Graph

明确：

```text
interview/
interview-plan/
```

都属于基础设施节点。

保持灰色。

不新增 graph color group。

---

# 21. templates/interview.md 最终修改

模板必须调整为：

```markdown
---
type: interview
scope:
area:
topics: []
created:
updated:
---

# 标题

## 问题

<!-- canonical_key: -->

### 考察点

-

### 回答框架

-

### 易追问

-

### 关联笔记

-
```

不再默认要求“完整参考答案”。

---

# 22. templates/interview-plan.md

固定结构：

```markdown
---
type: interview-plan
created:
updated:
---

# 秋招面试准备

## 当前目标岗位

-

## 当前阶段

-

## 本周重点

-

## 模块掌握度

-

## 高频薄弱题

-

## 下一步

-
```

---

# 23. templates/mock-interview-entry.md

固定结构：

```markdown
## YYYY-MM-DD

### 面试类型

-

### 没答好的问题

-

### 暴露的问题

-

### 后续动作

-
```

---

# 24. consolidate skill 最终修改

`/consolidate` 必须同步 CLAUDE.md 的以下规则：

* interview 新增 python/resume
* scope: topic/resume
* interview frontmatter schema
* 全局查重
* canonical_key
* 不自动创建缺失 notes
* resume 与 topic 的区别
* interview-plan 的职责
* 标准 Markdown 相对链接
* 不自动 commit

要求：

> CLAUDE.md 和 consolidate skill 的路由规则必须一致。

若两者不一致：

> 以 CLAUDE.md 为准，同时停止自动记题并报告规则漂移。

---

# 25. index.md 最终修改

根 index 只新增：

```markdown
## 面试准备

- [面试题库](../../interview/index.md)
- [2027 秋招 Dashboard](../../interview-plan/autumn-2027.md)
```

详细题集不写入根 index。

所有面试导航集中到：

```text
interview/index.md
```

---

# 26. 当前 dirty worktree 安全规则

当前工作区存在用户自己的未提交修改。

这些修改与秋招迁移不能混淆。

当前 Phase 0 处于：

> **BLOCKED**

状态。

---

## 26.1 Agent 禁止行为

在当前 dirty worktree 下，Agent 禁止：

* 自动 commit
* 自动 stash
* 自动 reset
* checkout 覆盖用户文件
* 修改已有 dirty 文件
* 开始正式题库迁移
* 修改 CLAUDE.md
* 修改 index.md
* 修改 graph.json
* 批量创建 interview 文件

---

## 26.2 用户处理方式

用户 review 本方案以后，应自行决定如何处理当前工作区。

安全目标不是强制 clean，而是：

> **能够明确区分“迁移前已有变更”和“秋招迁移新增变更”。**

推荐用户自行：

```bash
git status
```

确认当前修改后，选择自己熟悉的方式建立边界，例如：

* 手动 commit 当前工作；
* 或创建明确的 Git baseline；
* 或自行 stash；
* 或创建独立 branch。

Agent 不替用户做决定。

---

## 26.3 Phase 0 开始条件

只有用户明确授权以后，Agent 才可以进入 Phase 0。

Phase 0 仍然首先：

```text
只检查
不修改
```

如果工作区仍无法安全区分旧变更和迁移变更：

> Phase 0 停止。

---

# 27. 最终迁移阶段

## Phase 0：安全基线检查

只执行检查。

检查：

```text
git status
当前 branch
原题库文件存在
CLAUDE.md 存在
notes/interview 当前结构
```

不修改文件。

若 dirty worktree 尚未由用户明确处理：

```text
BLOCK
```

---

## Phase 1：规则与基础设施

修改：

```text
CLAUDE.md
consolidate skill
templates/interview.md
```

新增：

```text
interview/index.md
interview/meta/
interview/python/
interview/resume/
interview-plan/
templates/interview-plan.md
templates/mock-interview-entry.md
```

更新：

```text
index.md
```

此阶段：

> 不导入 725 道题。

---

## Phase 2：建立 migration ledger

解析：

```text
Q1 ～ Q725
```

建立：

```text
interview/meta/autumn-2027-migration-map.md
```

完成：

```text
source_id
→ disposition
→ canonical_file
→ canonical_key
```

此阶段：

> 不生成正式题集内容。

---

## Phase 3：三文件试点

第一批必须且只能先完成：

```text
interview/agents/agent-runtime.md

interview/agents/human-in-the-loop.md

interview/resume/kindpartner.md
```

同时：

* 建立必要的 notes 回链；
* 校验 canonical_key；
* 校验 source_id ledger；
* 校验 topic/resume 去重；
* 校验 index 导航。

然后：

> 停止，等待人工 review。

未通过 review：

> 不进入 Phase 4。

---

## Phase 4：Agent 核心题集

迁移：

```text
streaming-sse
function-calling-and-tools
mcp
memory
agent-multi-tenancy
rag
rag-frameworks-and-providers
prompt-and-agent-security
agent-evaluation
```

---

## Phase 5：后端 / Python / Java / Production

迁移：

```text
mysql
locking/idempotency
spring
redis
kafka
docker
java collections/concurrency
fastapi
asyncio
```

---

## Phase 6：系统设计与故障排查

迁移：

```text
agent-system-design
agent-troubleshooting
agent-architecture-and-tradeoffs
ai-coding-engineering
```

---

## Phase 7：简历题

迁移：

```text
kindvideo
resume-metrics
behavioral
motivation
```

并完成：

```text
kindpartner
```

剩余题目补全。

---

## Phase 8：Dashboard

创建并填写：

```text
interview-plan/autumn-2027.md
interview-plan/high-priority.md
interview-plan/mock-interview-log.md
```

只维护状态和链接。

---

## Phase 9：全量验收

要求：

```text
Q1 ～ Q725
```

全部在 ledger 中出现且恰好一次。

---

# 28. 验收规则

以下命令和检查项只作为实施规范。

当前禁止执行。

---

## 28.1 Git 检查

```bash
git status --short
git branch --show-current
```

验收目标：

* 可区分迁移前修改与迁移新增内容；
* 没有覆盖用户原有 dirty files。

---

## 28.2 原题连续性

检查：

```text
Q1
...
Q725
```

必须连续。

预期：

```text
725
```

---

## 28.3 Ledger 覆盖

必须满足：

```text
source_id unique count = 725
```

并：

```text
moved + merged + split + excluded
= 725 source rows
```

---

## 28.4 canonical_key 唯一性

扫描：

```text
interview/**/*.md
```

所有：

```text
canonical_key:
```

必须全局唯一。

出现重复：

> 验收失败。

---

## 28.5 Ledger → canonical_key 可达性

所有 disposition 为：

```text
moved
merged
split
```

的 canonical_key：

必须能在正式 interview 文件中找到。

---

## 28.6 链接检查

检查：

```text
Markdown relative links
```

确保：

* 无悬空 notes
* 无不存在的 interview 文件
* 无 wiki link
* 无混合链接

---

## 28.7 Frontmatter 检查

正式 interview 文件必须包含：

```text
type
scope
area
topics
created
updated
```

resume 项目文件额外检查：

```text
project
resume_version
```

---

## 28.8 禁止字段

interview 文件中不得使用：

```text
domain:
domains:
```

作为路由字段。

---

## 28.9 单一规范题检查

检查明显的 Exact Duplicate。

不得出现：

```text
两个不同 canonical_key
表达完全同一个知识问题
```

---

## 28.10 resume-metrics 唯一来源检查

量化指标口径不得在：

```text
kindpartner.md
kindvideo.md
```

重复维护完整版本。

只能链接：

```text
resume-metrics.md
```

---

## 28.11 index 体积检查

根：

```text
index.md
```

不得列全部题集。

必须只保留：

```text
interview/index
autumn-2027 dashboard
```

---

## 28.12 interview-plan 内容检查

不得包含：

* 大段技术答案
* 完整教程
* 项目指标口径副本

---

# 29. 迁移完成定义

只有同时满足以下条件，迁移才算完成：

```text
725 个 source_id 全部登记
+
canonical_key 全局唯一
+
所有正式 key 可定位
+
无 Exact Duplicate
+
无死链
+
frontmatter 合法
+
notes 不被机械生成
+
resume 指标只有一个口径来源
+
interview-plan 不复制知识
+
用户原有 dirty worktree 未被覆盖
```

---

# 30. 禁止事项

整个迁移过程禁止：

1. 自动执行 `git commit`
2. 自动 stash 用户修改
3. 自动 reset
4. 删除用户现有文件
5. 为补图谱自动创建知识笔记
6. 把 725 题保存成长期维护的单一大题集
7. 每道题创建一个独立文件
8. 逐题维护红黄绿状态
9. 在多个文件重复维护同一道通用知识题
10. 根据面试问题虚构项目实现
11. 根据简历数字虚构统计证据
12. 使用 Obsidian wiki link
13. 在根 index 展开所有面试题集
14. 未完成试点 review 就开始全量迁移

---

# 31. 当前实施状态

当前状态：

```text
PLAN: LOCKED
IMPLEMENTATION: NOT AUTHORIZED
PHASE 0: BLOCKED PENDING USER REVIEW
```

在收到明确授权以前：

> **不得修改 MyVault 的任何结构、规则、模板、索引或面试题文件。**

---

# 32. 实施授权口令

用户完成本方案 review，并自行确认当前 dirty worktree 的处理方式后，可发送以下完整口令：

> **授权执行 MyVault 秋招面试知识库最终方案，从 Phase 0 开始；先完成安全基线检查，满足条件后进入 Phase 1，禁止自动 commit，Phase 3 三文件试点完成后停止并等待我 review。**

只有收到该明确授权后，Agent 才能开始执行 Phase 0。
