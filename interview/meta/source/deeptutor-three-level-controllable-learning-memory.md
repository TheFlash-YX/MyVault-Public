# DeepTutor / KindPartner 面试复习

## 第 2 点：三级可控学习记忆 L1 → L2 → L3

> 目标：能在面试中解释为什么要做三级记忆、Consolidator 如何增量更新、如何保证可追溯和可纠错，以及 L3 Memory 最终怎么进入 Agent Context。

---

## 一、30 秒面试回答

我们没有把长期记忆直接做成一个黑盒向量库，而是设计成 L1、L2、L3 三层可审计记忆。

- **L1** 保存按 Surface 划分的原始 Trace，作为底层证据；
- **L2** 通过 Consolidator 从新增 Trace 中增量抽取按场景整理的事实；
- **L3** 再把多个 Surface 的 L2 信息聚合成跨场景长期记忆，比如 `recent`、`profile`、`scope` 等。

Consolidator 不是每次重写整份历史，而是通过 meta 记录已处理的 Entity / Entry ID，只消费新增部分；新增数据会分 Chunk 交给 LLM 抽取候选事实，再经过 Reference Validation，最后通过统一 AddOp 追加到 Memory Document。

这套系统的重点是“可控”：用户可以查看、修改、删除单条 Memory、Reset、Audit、Dedup、Merge 和 Undo；高层 Memory 还能沿引用关系追溯到底层 Trace。运行时 L3 可以作为 `memory_context` 注入 Agent Prompt，也可以通过 `read_memory` Tool 按需读取。

---

# 二、为什么不能只保存 Conversation History

长期使用中可能出现：

```text
Session 1：我准备算法岗面试
Session 2：动态规划是我的薄弱项
Session 3：我喜欢先讲思路，再看代码
Session 4：最近在学习 Agent Runtime
...
```

如果每次都把所有历史对话塞进 Prompt，会带来：

```text
历史越来越长
↓
Context Window 膨胀

同时：
大量历史消息价值很低
真正有用的用户信息比例不高
```

所以需要：

```text
Conversation History
 ↓
压缩 / 整理
 ↓
Long-term Memory
```

长期记忆的目的不是保存全部对话，而是把历史中的高价值用户信息沉淀下来。

---

# 三、为什么不直接做成 Vector DB

典型方案：

```text
用户对话
 ↓
Embedding
 ↓
Vector DB
 ↓
Query 时 Top-K Retrieval
```

这种方案适合大规模语义检索，但如果把“用户长期画像”完全交给黑盒向量召回，会有几个问题：

```text
用户不知道系统记住了什么
错误事实不容易修改
高层结论来源不清楚
用户难以审计为什么 Agent 形成某个判断
```

DeepTutor 的 Memory 更强调：

```text
Readable
Curatable
Auditable
Traceable
```

所以长期记忆首先是：

> **一个可管理、可检查的数据系统。**

---

# 四、三级结构总览

存储结构可以理解为：

```text
trace/<surface>/<YYYY-MM-DD>.jsonl
L2/<surface>.md
L3/<recent|profile|scope|preferences>.md
```

对应：

```text
L1 = Evidence
L2 = Facts
L3 = Long-term Profile
```

整体链路：

```text
                    用户真实行为
                         │
                         ↓
                ┌────────────────┐
                │      L1        │
                │   Raw Trace    │
                │    原始证据     │
                └───────┬────────┘
                        │
                   Consolidator
                        ↓
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
   L2/chat.md       L2/quiz.md      L2/course.md
   Chat Facts       Quiz Facts      Course Facts
        └───────────────┬───────────────┘
                        │
                   Consolidator
                        ↓
                ┌────────────────┐
                │      L3        │
                │ Cross-surface  │
                │ Long-term      │
                └────────────────┘
                 │    │      │
              recent profile scope
```

一句话：

> **L1 保证真实性，L2 负责压缩，L3 负责个性化。**

---

# 五、L1：原始事件层

L1 不急着生成高层画像，而是先保存事实。

例如：

```text
用户说：我最近正在准备算法岗面试
用户说：动态规划是我的薄弱项
用户连续做错两道 DP 题
用户在课程中正在学习动态规划
```

记录为不同 Surface 下的 Trace：

```text
event_001
surface = chat
用户说：动态规划是我的薄弱项

event_002
surface = quiz
用户连续两道 DP 题错误

event_003
surface = course
用户正在学习 DP
```

核心思想：

> **先保留证据，不要过早把一个瞬时行为变成长期人格判断。**

---

# 六、什么是 Surface

不同产品场景会产生不同类型的信息，例如：

```text
Chat
Quiz
Course
Notebook
Reading
...
```

这些可以理解成不同的 `Surface`。

所以 L1 按：

```text
trace/<surface>/<date>.jsonl
```

拆分，而不是把所有事件混到一起。

---

# 七、L2：Surface-specific Facts

L1 太原始、太多，不适合直接被 Agent 消费。

于是：

```text
L1 Trace
 ↓
Consolidator
 ↓
L2 Facts
```

例如：

```text
我下个月参加算法面试
动态规划不熟
我更喜欢先讲思想再看代码
```

可以整理成：

```text
L2/chat.md

- 用户近期准备算法岗位面试
- 用户认为动态规划是薄弱项
- 用户偏好先理解思路，再查看代码
```

因此：

> **L2 是按 Surface 整理后的 Curated Facts。**

---

# 八、为什么不直接 L1 → L3

直接从大量原始 Trace 生成最终 User Profile，会带来：

```text
输入过长
成本越来越高
一次错误事件直接污染全局画像
难以判断某条结论来自哪个场景
```

增加 L2 后：

```text
L1 Chat ─────→ L2 Chat ──┐
L1 Quiz ─────→ L2 Quiz ──┤
L1 Course ───→ L2 Course ├→ L3
L1 Reading ──→ L2 Reading┘
```

职责变清楚：

```text
L1：证据层
L2：场景压缩层
L3：跨场景综合层
```

---

# 九、L3：跨场景长期记忆

L3 更接近真正意义上的 Long-term Memory。

典型 Slot：

```text
recent
profile
scope
preferences
```

简单理解：

- `recent`：近期正在做什么
- `profile`：相对稳定的个人背景
- `scope`：长期目标或活动范围
- `preferences`：长期偏好

注意：

> `preferences` 不走普通自动 Consolidation，而有更明确的写入路径。

---

# 十、为什么三级结构能减少错误 Memory 污染

例如用户某天说：

```text
今天不想学数学了。
```

如果系统直接生成：

```text
用户不喜欢数学。
```

很可能是错误长期结论。

三级结构提供缓冲：

```text
L1：
今天用户说不想学数学

↓

L2：
近期一次学习中数学学习意愿下降

↓

L3：
只有在更多跨场景事实支持时
才应该形成更稳定的长期结论
```

因此：

> **原始行为和长期画像之间多了一层证据缓冲。**

---

# 十一、Consolidator 不是“全文总结”

Update Mode 更接近：

> **Chunk-based Incremental Fact Extraction**

不是：

```text
每次更新
↓
读取全部历史
↓
让 LLM 重新生成完整 Memory
↓
覆盖旧文件
```

而是：

```text
Existing Memory
+
New Input
 ↓
只处理新增部分
 ↓
抽取新 Fact
 ↓
Append 到现有 Memory
```

关键词：

> **Incremental Update**

---

# 十二、怎么知道哪些 L1 已经处理过

依赖 `*.meta.json` 中的：

```text
seen_entity_refs
```

例如：

```text
当前 L1：
entity_001
entity_002
entity_003
entity_004
entity_005

已处理：
entity_001
entity_002
entity_003
```

下一次只处理：

```text
entity_004
entity_005
```

Meta 可以类比为：

> **Consumer Offset / Watermark**

它记录 Consolidator 的消费进度。

---

# 十三、为什么只处理新增输入

如果每次全文重算：

```text
历史越长
↓
Token 成本越高

旧事实反复被模型重写
↓
可能产生 Memory Drift
```

增量方案则是：

```text
新增 Trace
↓
新增 Fact
↓
Append
```

更加稳定。

---

# 十四、为什么要 Chunk

如果一次新增大量 Trace，不会全部塞给一次 LLM。

而是：

```text
新 Entities
 ↓
按时间排序
 ↓
拼接
 ↓
chunk_with_boundary()
 ↓
Chunk 1 / Chunk 2 / Chunk 3 ...
```

然后每个 Chunk 单独做 Fact Extraction。

---

# 十五、每个 Chunk 给 LLM 什么

不只是新增 Trace，还包括：

```text
Existing L2
+
Current Chunk
+
Allowed References
+
Surface Focus
```

LLM 最终只负责生成：

```text
Candidate Fact 1
Candidate Fact 2
...
```

---

# 十六、为什么要提供 Existing L2

为了帮助模型知道：

```text
已经记过什么
```

减少重复抽取。

不过 Update 并不负责所有重复整理，DeepTutor 还把：

```text
update
dedup
merge
audit
```

拆成不同维护模式。

这体现：

> **事实抽取和事实整理职责分离。**

---

# 十七、LLM 抽出的 Fact 不能直接写入

LLM 负责：

```text
Candidate Fact Generation
```

代码还负责：

```text
Reference Validation
```

例如当前 Chunk 合法引用只有：

```text
chat:event_004
chat:event_005
```

如果 LLM 引用了别的事件：

```text
validate_fact_refs(...)
 ↓
invalid
 ↓
drop
```

因此：

> **LLM 不是 Memory 数据库的最终权威。**

LLM 负责语义判断，系统代码负责证据合法性。

---

# 十八、L1 → L2 完整链路

```text
L1 Raw Trace
 │
 ↓
Meta Diff
只取新增数据
 │
 ↓
按时间排序
 │
 ↓
Chunk
 │
 ↓
LLM Fact Extraction
 │
 ↓
Reference Validation
 │
 ├── invalid → drop
 │
 └── valid
       ↓
     AddOp
       ↓
Append to L2
```

---

# 十九、为什么通过 AddOp 写入

不是让 LLM 直接：

```python
file.write(llm_output)
```

而是：

```text
LLM 输出 Candidate Facts
 ↓
AddOp
 ↓
Memory Operation Layer
 ↓
Document
```

这样可以统一维护：

```text
Entry ID 分配
数据校验
Document Invariant
Reference / Footnote rebuild
```

核心思想：

> **LLM 负责语义，代码负责一致性。**

---

# 二十、Checkpoint 和 Meta 为什么分开

处理 Chunk 时可以阶段性写入有效事实。

但 `seen meta` 应该在整个新增输入真正处理完成后才推进。

假设：

```text
Chunk 1 ✅
Chunk 2 ✅
Chunk 3 ❌
```

如果提前把所有 Input 标记 Seen：

```text
Chunk 3
未来可能永远不会再处理
```

所以原则是：

> **先完成工作，再推进 Watermark。**

---

# 二十一、L2 → L3 如何更新

基本复用同样逻辑。

区别是输入不同：

```text
L2 Update：
New L1 Entities
↓
L2 Facts
```

```text
L3 Update：
New L2 Entries
↓
L3 Facts
```

L3 会根据：

```text
seen_l2_entry_ids
```

只处理新增 L2 Entry。

仍然是：

```text
Diff
↓
Chunk
↓
Extract
↓
Validate
↓
AddOp
↓
Checkpoint
↓
Advance Meta
```

---

# 二十二、L3 的 Provenance

L2 更接近：

```text
L2 Fact
↓
具体 L1 Trace
```

L3 的引用链则是：

```text
L3 Fact
 ↓
L2 Surface / Document
 ↓
L1 Raw Trace
```

即：

> **L3 → L2 → L1**

这是 Provenance / Data Lineage。

---

# 二十三、`preferences` 是特殊 Slot

当前普通 L3 Consolidation 不自动更新 `preferences`。

也就是说：

```text
recent
profile
scope
```

可以通过普通 Consolidator 更新。

而：

```text
preferences
```

走更明确的 `write_memory` 路径。

代码事实是：

> preferences 不参与普通自动 Consolidation。

从设计上理解，这有助于减少系统从一次偶发行为中过度推断长期偏好的风险。

---

# 二十四、“可控”具体体现在哪

DeepTutor 的 Memory 可以：

```text
Inspect
Edit
Delete
Reset
Audit
Dedup
Merge
Undo
```

这才是“Controllable Memory”的核心。

---

# 二十五、Inspect：用户能看到 Memory

L2/L3 是显式 Markdown 文档。

用户可以直接查看：

```text
L2/chat.md
L3/profile.md
L3/recent.md
...
```

因此它是：

> **Inspectable Memory**

---

# 二十六、Edit：用户可以修改

如果系统记成：

```text
用户长期使用 Java
```

用户可以直接改成：

```text
用户主要使用 Python
```

所以：

> **模型可以生成 Memory，但用户保留最终编辑权。**

---

# 二十七、Delete：删除单条 Entry

例如：

```text
m_001 用户是计算机相关背景
m_002 用户偏好 Java
m_003 用户正在准备 Agent 面试
```

如果 `m_002` 错了，可以只删除这一条。

这是：

> **Entry-level Correction**

---

# 二十八、Reset：整份重建

Reset 不能只删 `.md`，还要清除对应 Meta。

否则：

```text
seen_entity_refs
```

还在，下一次 Update 会认为原 L1 已经处理过。

所以：

```text
删除 Document
+
清理 Meta Watermark
 ↓
重新从 L1 构建
```

---

# 二十九、L1 Trace 也能删除

不只是高层 Memory 可以修改。

原始 Trace 也可以按：

```text
日期
Surface
```

删除。

所以纠错分两类：

```text
Derived-memory Correction
L3 错
↓
修改 / 删除 L3
```

```text
Source-level Correction
L1 本身错
↓
删除 L1
↓
Reset / Rebuild
```

---

# 三十、Update / Audit / Dedup / Merge

可以简单记：

```text
Update = 增
Dedup = 去重
Merge = 整理
Audit = 检查
```

Consolidator 不只是“自动记忆”，而是一整套维护生命周期。

---

# 三十一、Memory Maintenance 是可管理任务

Consolidator 可能需要多个 Chunk 和多个 LLM Call，因此会作为独立 Run 管理。

支持：

```text
start
status
events
cancel
undo
```

所以：

> **Memory Maintenance 本身也是一个 Workflow。**

---

# 三十二、为什么 Undo 很重要

长期 Memory 一旦写错，可能影响很多后续 Session。

Run-level Undo 让 Memory Mutation 具备：

> **Reversible**

而不是一次写入不可恢复。

---

# 三十三、错误 Memory 为什么危险

普通错误回答：

```text
影响一轮
```

错误长期 Memory：

```text
错误 Memory
 ↓
进入未来 Prompt
 ↓
Agent 基于错误认知回答
 ↓
产生更多错误 Trace
 ↓
继续污染 Memory
```

可能形成：

> **Memory Pollution Feedback Loop**

因此长期记忆一定要支持检查、追踪和纠错。

---

# 三十四、L3 最后如何进入 Agent Context

L3 最终必须被 Agent 消费。

主要有两种方式：

```text
1. memory_context 直接注入
2. read_memory Tool 按需读取
```

---

# 三十五、方式一：`memory_context`

运行时可以读取 L3 并形成：

```python
context.memory_context
```

`ChatPromptAssembler` 再把它加入：

```text
## memory
```

System Prompt Block。

大致：

```text
## general
...

## runtime_context
...

## loop
...

## memory
用户长期记忆：
- ...
- ...

## tools
...
```

因此 Memory 被明确作为长期用户背景，而不是当前用户新输入。

---

# 三十六、不是每轮无脑全量注入

`memory_context` 有上下文选择条件。

不是每一轮都强行把全部 L3 塞进 Prompt。

这也是 Context / Token 控制的一部分。

---

# 三十七、方式二：`read_memory` Tool

Agent 也可以运行时主动：

```text
read_memory()
```

链路：

```text
用户问题
 ↓
LLM 判断需要长期背景
 ↓
read_memory
 ↓
L3 Memory
 ↓
role=tool
 ↓
下一轮 LLM
```

这属于：

> **On-demand Memory Retrieval**

---

# 三十八、为什么既有直接注入又有 Tool

直接注入：

```text
第一轮 LLM 就能看到
不需要额外 Tool Call
```

适合明确需要 Memory 的 Turn。

Tool 读取：

```text
执行过程中
模型自己决定是否需要
```

更 Agentic，也避免不必要的 Context 占用。

---

# 三十九、Memory 和 Conversation History 的区别

## Conversation History

回答：

```text
这次会话刚刚说过什么？
```

特点：

```text
Session-local
短期上下文
```

## Memory

回答：

```text
这个用户长期是谁？
```

特点：

```text
Cross-session
Cross-surface
长期状态
```

一句话：

```text
Conversation History = 当前会话的短期上下文
Memory = 跨会话的长期用户状态
```

---

# 四十、Memory 和 RAG 的区别

Memory 主要提供：

```text
关于用户自己的知识
```

比如：

```text
背景
学习状态
长期偏好
近期目标
```

RAG 主要提供：

```text
外部领域知识
```

比如：

```text
教材
PDF
Knowledge Base
```

最简单记：

```text
Memory = User Knowledge
RAG = Domain Knowledge
```

---

# 四十一、三类 Context 对比

| 类型 | 解决的问题 | 生命周期 |
|---|---|---|
| Conversation History | 当前会话刚才说了什么 | Session / Branch |
| Memory | 用户长期是谁 | Cross-session |
| RAG | 外部知识是什么 | Knowledge Base |

例子：

```text
“按照我之前的学习习惯，
结合《算法导论》，
继续讲刚才的动态规划。”
```

对应：

```text
继续讲刚才的动态规划
↓
Conversation History

按照我之前的学习习惯
↓
Memory

结合《算法导论》
↓
RAG
```

---

# 四十二、为什么 Memory 不等同于 RAG

RAG 更适合：

```text
海量外部文档
↓
Query
↓
Top-K Chunk
```

而 L3 Memory 已经是：

```text
高度压缩
高价值
相对稳定
```

的信息。

同时 Memory 更强调：

```text
可查看
可编辑
可删除
可追溯
```

所以：

> **Memory 首先是可管理的数据产品，其次才是 Retrieval 问题。**

---

# 四十三、为什么 L3 最适合进 Prompt

可以记成：

```text
L1 适合审计
L2 适合整理
L3 适合推理时消费
```

因为 L3 已经经过两轮信息压缩，信息密度高、噪声低，更适合直接进入 Agent Context。

---

# 四十四、完整 Memory Lifecycle

```text
用户行为
 ↓
L1 Raw Trace
 ↓
Incremental Consolidator
 ↓
Reference Validation
 ↓
AddOp
 ↓
L2 Surface Facts
 ↓
Incremental Consolidator
 ↓
L3 Long-term Memory
 ↓
      ┌───────────────────┐
      │                   │
      ↓                   ↓
memory_context        read_memory Tool
      │                   │
      └─────────┬─────────┘
                ↓
            Agent Context
                ↓
               LLM
                ↓
           个性化回答
```

旁边还有控制面：

```text
Inspect
Edit
Delete
Reset
Audit
Dedup
Merge
Undo
```

这不是单纯的 Memory Extraction，而是：

> **Memory Lifecycle Management**

---

# 四十五、完整面试回答

如果面试官问：

> 介绍一下你们的三级长期记忆系统。

可以回答：

> 我们把长期记忆设计成 L1、L2、L3 三层，而不是把所有聊天直接 Embedding 到一个黑盒向量库。L1 保存按 Surface 划分的原始 Trace，保证证据完整；Consolidator 根据 meta 中的 seen id 做增量消费，把新增 Trace 分 Chunk 交给 LLM 抽取候选事实，经过 Reference Validation 后，再通过统一 AddOp 追加到 L2。L2 是各 Surface 的结构化事实，再用类似的增量机制聚合成 L3 的 recent、profile、scope 等跨场景长期记忆。
>
> 这套系统重点是“可控”。L2/L3 都是用户可查看和编辑的 Markdown Document，可以删单条 Entry、Reset、Audit、Dedup、Merge，Consolidator Run 也支持 Undo；高层 Memory 还能沿引用关系追溯到底层 Trace。
>
> 在运行时，L3 可以作为 `memory_context` 进入 Agent 的 System Prompt，也可以由模型通过 `read_memory` Tool 按需读取。它和 Conversation History、RAG 分工不同：History 负责当前 Session 的短期上下文，Memory 负责跨 Session 的长期用户状态，RAG 负责外部领域知识。

---

# 四十六、常见面试追问

## Q1：为什么要做三级，而不是一层 Memory？

> 因为长期记忆同时要解决证据完整、信息压缩和长期个性化三个问题。L1 保留原始 Trace，L2 按 Surface 压缩成事实，L3 再跨 Surface 形成稳定画像。这样高层结论既能直接被 Agent 使用，又能追溯到底层证据。

## Q2：为什么不直接用 Vector DB？

> Vector DB 更适合大规模语义检索，但长期用户画像更强调可解释和可管理。DeepTutor 的 L2/L3 是显式文档，用户能查看、修改、删除和追踪来源。对于长期 Memory，控制权和 Provenance 与召回能力同样重要。

## Q3：Consolidator 每次会重新总结所有历史吗？

> 不会。它通过 meta 记录已经消费过的 Entity / Entry ID，只处理新增部分。L2 消费新增 L1，L3 消费新增 L2，再经过 Chunk、LLM Fact Extraction、Reference Validation 和 AddOp 追加，最后才推进 seen watermark。

## Q4：为什么 Meta 要最后更新？

> 因为 Meta 是消费进度。如果 Chunk 只处理一半就提前推进 Seen 状态，失败输入以后可能永远不会再次处理。所以应该先完成实际数据写入，再推进 Watermark。

## Q5：LLM 抽出来的 Memory 直接相信吗？

> 不直接相信。LLM 只负责生成 Candidate Facts，系统还会检查 Fact 的 Reference 是否属于当前允许的证据池。非法引用会被 Drop，真正写入也通过统一 AddOp，而不是让模型直接覆盖 Memory 文件。

## Q6：如果 Memory 记错了怎么办？

> 用户可以直接编辑文档，也可以删除单条 Entry。如果底层 Trace 本身错误，可以删除 L1，再 Reset 对应 L2/L3 和 Meta，从源数据重新 Consolidate；维护 Run 还支持 Undo，因此 Memory Mutation 是可逆的。

## Q7：为什么 `preferences` 不自动聚合？

> 当前实现里 preferences 不走普通自动 Consolidation，而通过更明确的 `write_memory` 路径写入。代码事实就是它和 recent/profile/scope 的更新机制不同。从设计上理解，这也能降低系统从一次偶发行为中过度推断长期偏好的风险。

## Q8：Memory 和 Conversation History 有什么区别？

> History 解决当前 Session 前面发生了什么，是短期上下文；Memory 解决这个用户长期是谁，是跨 Session、跨 Surface 的长期状态。

## Q9：Memory 和 RAG 有什么区别？

> Memory 主要存关于用户自己的长期知识，比如背景、近期状态和偏好；RAG 主要检索外部领域知识，比如教材和知识库。可以简单理解成 Memory 回答“你是谁”，RAG 回答“外部知识是什么”。

## Q10：L3 怎么进入 Agent？

> 一种方式是作为 `memory_context` 进入 System Prompt 独立的 memory block；另一种是 Agent 运行过程中通过 `read_memory` Tool 按需读取。这样同时支持 Eager Context Injection 和 On-demand Retrieval。

---

# 四十七、最终速记

先记这三个等式：

```text
L1 = Evidence
L2 = Facts
L3 = Long-term Profile
```

```text
LLM = Candidate Fact Generator
Code = Consistency / Validation Controller
```

```text
History = 刚才聊了什么
Memory = 你长期是谁
RAG = 外部知识是什么
```

再记 Consolidator 的 7 步：

```text
Diff
↓
Chunk
↓
Extract
↓
Validate
↓
AddOp
↓
Checkpoint
↓
Advance Meta
```

最后记一句：

> **长期 Memory 的核心不是“尽量多记”，而是“有证据地记、增量地记、允许用户纠错，并让这些记忆真正进入 Agent 的决策上下文”。**
