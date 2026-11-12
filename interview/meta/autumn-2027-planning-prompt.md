你现在需要帮我规划并改造我的个人知识库 MyVault，使其能够更好地承载 **AI 应用开发工程师 / AI Agent 研发工程师秋招面试准备**，但不要把秋招面试资料单独拆成第二套知识库。

请严格遵循当前项目中的 `CLAUDE.md` 规则，并在现有知识库结构上做增量规划，不要破坏已有目录、链接和工作流。

## 背景

当前 MyVault 是我的长期个人学习知识库，使用 OKF 思路组织 Java、后端、前端、Agent、算法、Python 等知识，并已经存在：

- `notes/`：长期知识沉淀
- `daily/`：学习日志
- `pitfalls/`：踩坑
- `resources/`：资源
- `interview/`：按主题组织的面试题集
- `templates/`
- `index.md`

其中 `interview/` 已经用于保存主题面试题，例如：

```text
interview/
├── java/
├── backend/
├── frontend/
└── agents/
```

我现在有一份较大的 **AI 应用开发 / AI Agent 秋招面试问题库**，问题覆盖：

- Agent Runtime
- Agent Loop
- ask_user / Human-in-the-loop
- SSE / StreamEvent
- Function Calling
- Tool System
- MCP
- Memory
- Partner / Workspace / 多租户隔离
- RAG
- LlamaIndex / LightRAG
- Agent Evaluation
- Prompt Injection / Agent 安全
- FastAPI / Python / asyncio
- MySQL
- Spring Boot / Java
- Redis / Kafka
- Docker / Production
- AI Coding / TDD / SDD
- Agent 系统设计
- 故障排查
- KindPartner 简历追问
- KindVideo 简历追问
- 简历数据真实性追问
- 行为面
- 求职动机
- AI Agent 开放题

这份题库不要简单作为一个几百题的大 Markdown 文件长期维护，而应该拆解后融入 MyVault。

---

# 一、总体设计原则

请按照下面的三层结构规划：

```text
知识层
notes/
↓
解决“这个知识我是否真正理解”

题库层
interview/
↓
解决“面试官会如何问，我应该如何表达”

秋招准备层
interview-plan/
↓
解决“当前应该准备什么、哪些是薄弱项、哪些需要复习”
```

核心原则：

1. **不要新建第二个独立知识库。**
2. **不要复制两份相同知识。**
3. `notes/` 是 Single Source of Truth，保存完整知识。
4. `interview/` 保存“问题 + 面试表达 + 关联知识”，而不是重新写一篇完整教程。
5. 秋招进度、薄弱项、刷题计划等短期状态放在 `interview-plan/`。
6. 秋招结束后，即使 `interview-plan/` 被归档，`notes/` 和 `interview/` 中的长期知识仍然有价值。

---

# 二、希望新增/调整的目录

请评估并规划以下结构：

```text
MyVault/
├── notes/
│   ├── agents/
│   ├── backend/
│   ├── frontend/
│   ├── algorithms/
│   └── python/
│
├── interview/
│   ├── java/
│   ├── backend/
│   ├── frontend/
│   ├── agents/
│   └── resume/              # 建议新增
│
├── interview-plan/          # 建议新增
│   ├── autumn-2027.md
│   ├── high-priority.md
│   └── mock-interview-log.md
│
├── daily/
├── pitfalls/
├── resources/
├── templates/
└── index.md
```

请重点判断：

- `interview/resume/` 是否应该新增；
- `interview-plan/` 是否应该新增；
- 是否还需要其他目录；
- 哪些内容应该长期存在，哪些内容只属于秋招阶段。

---

# 三、面试题如何拆分

不要创建：

```text
interview/ai-agent-725-questions.md
```

这种超大文件。

请把原始大题库作为“待导入数据源”，按主题拆分。

建议优先规划为：

```text
interview/agents/
├── agent-runtime.md
├── human-in-the-loop.md
├── streaming-sse.md
├── function-calling.md
├── mcp.md
├── memory.md
├── rag.md
├── rag-evaluation.md
├── agent-evaluation.md
├── agent-security.md
├── agent-system-design.md
├── agent-troubleshooting.md
└── agent-open-questions.md
```

后端相关：

```text
interview/backend/
├── mysql.md
├── transactions-locking.md
├── spring.md
├── redis.md
├── kafka.md
├── docker-production.md
└── backend-system-design.md
```

Python 相关如果当前 `interview/` 没有 python 域，请你判断：

方案 A：

```text
interview/python/
├── asyncio.md
├── fastapi.md
└── python-concurrency.md
```

方案 B：

归入 backend。

请分析后选择更合理的一种，而不是机械照搬。

---

# 四、简历追问题单独管理

以下题目和我的个人简历强绑定，不应该混入普通 Agent 八股：

- 为什么设计 ask_user？
- 98% 同 Turn 恢复成功率如何统计？
- 重复执行率 0% 如何证明？
- UnifiedContext 为什么这么设计？
- Memory L1/L2/L3 怎么实现？
- RAG Provider 为什么这样抽象？
- 7000+ 探针从哪里来？
- 困难样本周期缩短 35% 怎么统计？
- 乐观锁具体在哪个表？
- 1000+ 视频是什么口径？
- F1 78%、Recall@5 91% 怎么计算？
- 跨作用域泄漏 0% 如何测试？

建议：

```text
interview/resume/
├── kindpartner.md
├── kindvideo.md
├── resume-metrics.md
├── behavioral.md
└── motivation.md
```

请你评估这个拆法是否合理。

其中：

### `kindpartner.md`

重点管理：

- Agent Runtime
- ask_user
- StreamEvent
- Memory
- Partner
- UnifiedContext
- RAG Provider

对应的“项目级追问”。

### `kindvideo.md`

重点管理：

- 探针池
- 策略模式
- 状态机
- 快照
- 幂等
- 乐观锁
- 数据一致性
- BEV 标注任务

### `resume-metrics.md`

集中管理所有可能被面试官质疑真实性的数据：

- 7000+
- 100+
- 35%
- 1000+
- 98%
- 0%
- 100%
- 78%
- 91%

每个指标应该至少能够回答：

```text
指标是什么
统计口径
样本量
baseline
统计周期
计算公式
数据来源
是否线上数据
我的个人贡献
可能存在的偏差
```

---

# 五、interview 文件的定位

请保持以下原则：

`notes/agents/mcp.md`

回答的是：

> MCP 到底是什么？

而：

`interview/agents/mcp.md`

回答的是：

> 面试官会怎么问 MCP？

例如 interview 文件建议：

```markdown
---
title: MCP 面试题
domain: agents
created:
updated:
tags:
  - interview
  - agent
  - mcp
---

# MCP 面试题

## Q1：MCP 是什么？

### 考察点

- MCP 的定位
- Host / Client / Server
- Tools / Resources / Prompts
- 和 Function Calling 的关系

### 回答框架

1. 一句话定义
2. 解决的问题
3. 核心角色
4. 和 Function Calling 区别
5. 实际项目使用场景

### 易追问

- MCP 和 REST API 有什么区别？
- MCP 和 Function Calling 有什么区别？
- MCP Server 如何发现？
- 如何保证第三方 MCP Server 安全？

### 关联笔记

- [MCP](../../interview/agents/mcp.md)
```

注意：

**不要在 interview 文件里把 notes 内容完整复制一遍。**

---

# 六、题目状态管理

我希望在秋招阶段快速知道：

- 哪些题完全不会；
- 哪些题能讲但是讲不深；
- 哪些题已经准备充分。

请帮我设计一个简单、低维护成本的状态系统。

可以考虑：

```text
🔴 未掌握
🟡 能回答但不够深入
🟢 已掌握
⭐ 高频重点
```

例如：

```markdown
## Q3：为什么 ask_user 要保持同一个 Turn？

状态：🔴 ⭐

### 考察点

...
```

但请你判断：

- 是否真的需要每道题写状态；
- 会不会造成维护成本过高；
- 有没有更合理的方式，例如只在 `interview-plan/` 中维护薄弱题。

原则是：

> 不要为了“知识库看起来很完整”而增加大量维护成本。

---

# 七、秋招准备层 interview-plan

这个目录不是知识库主体，而是短期的“控制面板”。

建议：

## `interview-plan/autumn-2027.md`

作为秋招主 Dashboard：

```markdown
# 2027 秋招 AI Agent 面试准备

## 当前目标岗位

- AI 应用开发工程师
- AI Agent 研发工程师

## 当前准备阶段

-

## 本周重点

- [ ] ask_user
- [ ] Agent Runtime
- [ ] Memory
- [ ] RAG
- [ ] asyncio
- [ ] MySQL / 乐观锁

## 当前薄弱模块

| 模块 | 掌握程度 |
|---|---|
| ask_user | 🔴 |
| Agent Runtime | 🟡 |
| Memory | 🟡 |
| RAG | 🟢 |
| MCP | 🟡 |

## 高频薄弱题

-

## 本周模拟面试

-

## 下一步

-
```

## `high-priority.md`

只管理真正需要近期复习的问题。

不要复制题目答案，可以链接：

```markdown
# 高频重点

## 🔴 ask_user

- [为什么同一个 Turn 恢复？](../../interview/resume/kindpartner.md)
- [多 worker 下 Queue 怎么处理？](../../interview/python/asyncio-and-concurrency.md)

## 🟡 Agent Runtime

-
```

## `mock-interview-log.md`

用于记录模拟面试或真实面试暴露的问题，例如：

```markdown
# 模拟面试记录

## 2026-08-XX

### 没答好的问题

- SSE 为什么不用 WebSocket
- asyncio.Queue 多 worker 问题
- RAG Recall@5 的统计口径

### 暴露的问题

- 对工程异常场景回答不够深入
- 指标口径记忆不清

### 后续动作

- 更新 xxx
```

---

# 八、notes 与 interview 的关联策略

请充分利用已有双向链接规则。

例如：

```text
notes/agents/agent-memory.md
             ↑ ↓
interview/agents/memory.md
             ↑
interview/resume/kindpartner.md
```

其中：

- `notes` 保存知识；
- `interview/agents` 保存通用面试表达；
- `interview/resume` 保存“我的项目是怎么做的”。

希望最终形成：

```text
知识
  ↓
通用面试题
  ↓
项目追问
  ↓
秋招薄弱项
```

而不是四套相互独立的内容。

---

# 九、对 CLAUDE.md 的调整

请检查当前 `CLAUDE.md`。

如果上述规划与现有规则存在冲突，请提出修改建议。

重点检查：

1. `interview` 领域当前只有：

```text
java
backend
frontend
agents
```

是否需要：

```text
resume
python
```

2. 「记题流」是否需要支持 `resume` 类型。
3. 领域映射是否需要区分：

```text
知识领域
vs
简历项目追问
```

4. `interview-plan/` 是否应该加入项目结构探测。
5. 是否应该建立新的 template：

```text
templates/interview-plan.md
templates/mock-interview.md
```

6. `index.md` 如何展示：

- 普通面试题
- 简历追问
- 秋招准备入口

7. Obsidian 图谱中：

`interview-plan/`

建议继续保持基础设施灰色，不新增颜色。

---

# 十、迁移现有秋招题库

如果当前项目中能访问我的 AI Agent 秋招题库，请：

### 第一步：分析

先不要直接创建几十个文件。

先输出：

```text
原题库
↓
去重
↓
领域分类
↓
主题分类
↓
预计生成哪些文件
```

例如：

```text
Agent Runtime → interview/agents/agent-runtime.md
ask_user → interview/agents/human-in-the-loop.md
MCP → interview/agents/mcp.md
KindPartner 项目追问 → interview/resume/kindpartner.md
MySQL → interview/backend/mysql.md
...
```

### 第二步：检查已有知识库

扫描现有：

```text
notes/
interview/
```

判断哪些主题已经存在。

严格遵守：

> 宁可更新已有文件，不制造重复文件。

### 第三步：制定迁移计划

输出：

```text
新增文件
更新文件
无需处理
需要人工确认
```

### 第四步：再执行迁移

只有完成上述分析之后，再开始写文件。

---

# 十一、需要特别避免的问题

不要做以下事情：

### 1. 不要生成一个 700+ 题超级文件

错误：

```text
interview/ai-agent-725.md
```

### 2. 不要把同一知识复制到 notes 和 interview

错误：

```text
notes/agents/rag.md       # 5000 字
interview/agents/rag.md   # 又复制 5000 字
```

### 3. 不要为了每一道题创建一个文件

错误：

```text
interview/agents/what-is-mcp.md
interview/agents/mcp-vs-function-calling.md
...
```

应该：

```text
interview/agents/mcp.md
```

内部多个 Q&A。

### 4. 不要让 interview-plan 成为第二套知识库

里面只放：

- 状态
- 计划
- 薄弱项
- 链接
- 面试记录

不要放完整技术知识。

### 5. 不要随意修改现有知识结构

所有改动应尽可能兼容当前 CLAUDE.md 的：

- 查重机制
- Markdown 相对链接
- index
- frontmatter
- Obsidian 图谱
- 目录命名规则

---

# 十二、最终请先给我一份规划方案

这一次先不要直接大规模写文件。

请先完成以下输出：

## 1. 当前知识库结构评估

分析当前 MyVault 是否适合承载秋招题库。

## 2. 推荐目录结构

给出最终 tree。

## 3. 文件职责

说明：

```text
notes
interview/agents
interview/backend
interview/resume
interview-plan
```

分别负责什么。

## 4. AI Agent 秋招题库拆分映射

把现有题库模块映射到具体文件。

## 5. CLAUDE.md 修改建议

明确指出需要新增、删除、修改哪些规则。

## 6. 迁移步骤

给出一个低风险顺序：

```text
阶段 1：改规则
阶段 2：创建目录/template
阶段 3：拆题库
阶段 4：关联 notes
阶段 5：建立秋招 Dashboard
阶段 6：检查重复和死链
```

## 7. 潜在问题

重点考虑：

- 文件数量是否过多；
- 题目状态维护成本；
- notes/interview 内容重复；
- 简历版本变化；
- 面试题长期过时；
- 大规模导入后 index 是否变得臃肿；
- Agent 自动维护是否容易制造重复文件。

最后给出你的推荐方案，并说明为什么。

在我确认规划之前，不要自动执行 `git commit`，也不要进行不可逆的大规模重构。
