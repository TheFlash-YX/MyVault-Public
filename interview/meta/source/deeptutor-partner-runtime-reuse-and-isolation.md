# DeepTutor / KindPartner 面试复习

- 归档说明：原复习稿正文保留；下文的实现描述与指标表述来自学习材料，本次整理未核验项目源码和测试报告。
- 知识沉淀：[Runtime 与 Partner 隔离](../../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)、[Split Memory](../../../notes/agents/projects/deeptutor/learning-memory-system.md)。
- 面试入口：[多租户隔离](../../agents/agent-multi-tenancy.md)、[工具权限](../../agents/function-calling-and-tools.md)、[KindPartner 项目答辩](../../resume/kindpartner.md)、[指标证据口径](../../resume/resume-metrics.md)。

## 第 3 点：个性化学习 Partner 的运行时复用与隔离

> 目标：讲清 Partner 为什么不需要独立 Agent Engine、如何通过 `UnifiedContext` + Synthetic Scope 复用统一 Runtime、Workspace / Tool / Memory 如何隔离，以及“跨作用域泄漏 0% / 越权调用拦截率 100%”怎么解释。

---

## 一、30 秒面试回答

Partner 没有单独实现一套 Agent Engine，而是复用产品 Chat 的同一条 `TurnEngine → AgenticChatPipeline → AgentLoop`。不同 Partner 的差异不在 Engine，而在 Context 和 Resource Scope：每个 Partner 都有独立 Workspace，里面装配自己的 `SOUL.md`、Knowledge Base、Skill、Notebook 和 Memory；运行时进入 Partner 对应的 Synthetic User Scope，再通过 `UnifiedContext` 把当前 Turn 的身份、资源和权限传给统一 Agent Runtime。

Tool 权限在每个 Turn 前通过 `compose_enabled_tools` 重新计算，使用 `builtin_whitelist / forced / suppressed` 得到最终 Tool Surface。Memory 采用 Split Model：用户 Shared L3 对 Partner 只读，Partner 只能把关系类记忆写进 `Partner × User` 独立 Relationship Memory。

一句话：

> **Shared Runtime + Independent Context + Isolated Workspace + Scoped Capabilities**

---

# 二、这一点解决的核心问题

假设用户创建两个 Partner：

```text
Partner A：算法面试教练
Partner B：英语学习搭子
```

它们不仅人格不同，还可能拥有不同的：

```text
Soul
Knowledge Base
Skill
Notebook
Memory
Tool Permission
```

所以这个问题不是“怎么换一个 Prompt”，而是：

> **怎么在共享同一套 Agent Runtime 的同时，让不同 Agent 的身份、资源和权限互相隔离。**

---

# 三、为什么不为每个 Partner 单独做 Agent Engine

错误方案：

```text
Partner A → Agent Engine A
Partner B → Agent Engine B
Partner C → Agent Engine C
```

这样 `AgentLoop / Tool Calling / Streaming / RAG / Memory / Dispatcher` 都要多份维护。

结果会出现：

```text
Chat 修一个 Runtime Bug
↓
Partner Engine 也要同步修

新增一个 Tool
↓
所有 Engine 都要接
```

最终变成多套 Runtime 分叉维护。

DeepTutor 的设计更接近：

```text
Partner
没有自己的 Engine
```

而是：

```text
User / IM Message
 ↓
Partner Runtime
 ↓
TurnEngine
 ↓
AgenticChatPipeline
 ↓
AgentLoop
```

普通 Chat 和 Partner 共用同一套执行机制。

---

# 四、`UnifiedContext` 的作用

可以把统一 Runtime 抽象成：

```python
run_agent(context)
```

Runtime 本身不应该硬编码：

```python
if partner_a:
    use kb_a
```

它只关心当前 Context：

```text
我是谁？
我能看到什么资源？
我能使用什么 Tool？
当前 Workspace 在哪里？
```

于是：

```text
Partner Config
│
├── Soul
├── KB
├── Skill
├── Notebook
├── Memory
└── Tool Policy
        ↓
   UnifiedContext
        ↓
Shared Agent Runtime
```

所以：

> **UnifiedContext 是把不同 Agent 的运行差异注入统一 Runtime 的载体。**

---

# 五、Synthetic User Scope 是什么

Partner 不是普通登录用户，但 Runtime 希望它表现得像一个拥有独立 Workspace 的用户。

可以概念化理解：

```text
scope = partner:<partner_id>
```

进入这个 Scope 后：

```text
PathService
RAG
Skill
Notebook
Memory
```

都会解析到当前 Partner 的资源。

注意区分：

```text
UnifiedContext
= 当前 Turn 的 Agent 运行上下文

Synthetic Scope
= 当前底层资源访问属于哪个身份
```

---

# 六、Partner Workspace

可以理解为：

```text
data/partners/<partner_id>/workspace/
│
├── knowledge_bases/
│   └── <kb>/
│
└── user/
    └── workspace/
        ├── SOUL.md
        ├── skills/
        │   └── <skill>/SKILL.md
        ├── notebook/
        └── memory/
```

不同 Partner 有不同 Workspace。

关键设计是：Partner Workspace 尽量复用普通 Chat 用户 Workspace 的格式。

这样原有：

```text
RAG Tool
read_skill
Notebook Tool
Memory Tool
```

不需要重写 Partner 版本，只需要切换当前 Scope。

---

# 七、为什么这叫运行时复用

核心三句话：

```text
Engine 是共享的
Context 是不同的
Workspace 是隔离的
```

共享：

```text
AgentLoop
Tool Dispatcher
Streaming
Prompt Assembly
Context Budget
```

隔离：

```text
Identity
Soul
KB
Skill
Notebook
Memory
Permission
```

这可以理解成：

> **Mechanism / Policy Separation**

Runtime 负责“怎么执行 Agent”，Partner Config 负责“这个 Agent 是谁、能做什么”。

---

# 八、`SOUL.md` 不等于整个 Partner

`SOUL.md` 主要定义 Partner 的 Persona，比如：

```markdown
# Soul

你是一个算法面试学习伙伴。

- 不直接给答案
- 优先引导用户推导
- 用户答错时指出思维断点
```

但 Partner 的差异绝不只是 Prompt。

完整差异是：

```text
Persona
+
Data
+
Capability
+
State
+
Permission
```

---

# 九、Tool 权限为什么不能只靠 Prompt

错误方式：

```text
System Prompt：
“你不能调用 write_memory。”
```

这只是软约束。

模型仍可能生成 `write_memory(...)`。

真正的权限边界必须在 Runtime：

```text
Prompt
告诉模型应该做什么

Runtime Policy
决定模型实际上能做什么
```

一句话：

> **安全边界放在代码层，而不是 Prompt 层。**

---

# 十、Per-turn Tool Composition

每个 Turn 前会生成当前真正允许使用的 Tool List：

```text
UnifiedContext
 ↓
ToolMountFlags
 ↓
compose_enabled_tools()
 ↓
Final Enabled Tools
 ↓
AgentLoop
```

模型看到的是当前 Turn 的 Tool Surface，而不是整个全局 Tool Registry。

---

# 十一、三个最关键的 Tool Policy

## 1. `builtin_whitelist`

含义：

> 最多允许哪些 Built-in Tool 被挂载。

它只做减法，不负责强制开启。

例如：

```text
whitelist:
rag
ask_user
read_skill
```

即使 Context 存在 Notebook，如果 `list_notebook / write_note` 不在 whitelist，也不能自动出现。

所以：

```text
Whitelist = Capability Ceiling
```

---

## 2. `forced`

Partner 的基础工具：

```text
partner_read
partner_memorize
partner_search
```

通过 `forced` 保证存在。

含义：

```text
不依赖普通 Context Gate
不依赖 builtin_whitelist
```

所以：

```text
Forced = 必须有什么
```

---

## 3. `suppressed`

Partner Turn 明确移除普通 Chat 的：

```text
read_memory
write_memory
```

`suppressed` 是最终 Hard Deny：

```text
不管 Tool 是通过什么路径进入
↓
最终都删除
```

所以：

```text
Suppressed = 绝对不能有什么
```

---

# 十二、三个词直接背

```text
Whitelist
= 最多能有什么

Forced
= 必须有什么

Suppressed
= 绝对不能有什么
```

---

# 十三、为什么 Partner 禁止普通 `write_memory`

因为 Partner 的 Memory 权限模型不是：

```text
Partner
↓
直接修改用户全局 Memory
```

而是 Split Memory：

```text
                 Partner
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
   User Shared L3        Relationship Memory
          │                   │
       Read-only            Read / Write
          │                   │
          └─────────┬─────────┘
                    ↓
               partner_read
```

因此普通 `write_memory` 必须被 Suppress，改用 `partner_memorize`。

---

# 十四、Relationship Memory 的隔离粒度

关系记忆可以理解为：

```text
data/partners/<partner_id>/
    users/<user_id>/
        workspace/memory/
```

所以它不是只按 Partner 隔离，而是：

> **Partner × User**

例如：

```text
Partner A × User1
Partner A × User2
Partner B × User1
```

三份关系记忆互相独立。

---

# 十五、为什么 User Shared L3 只读

用户全局长期 Memory 可能同时被：

```text
Product Chat
Partner A
Partner B
其他 Capability
```

使用。

如果 Partner A 可以随意修改用户全局 Profile，一次局部对话就可能污染其他 Agent。

所以：

```text
User Shared L3
Read ✅
Write ❌
```

Partner 只能把双方特有的信息写到：

```text
Partner Relationship Memory
```

---

# 十六、为什么 Relationship Memory 有业务意义

用户对算法 Partner 说：

```text
以后给我算法题时先别给提示。
```

这适合写：

```text
算法 Partner Relationship Memory
```

但不一定应该影响英语 Partner。

所以：

```text
Global User Memory
≠
Partner Relationship Memory
```

---

# 十七、Tool Surface + Resource Scope 双层隔离

这是最重要的设计之一。

## 第一层：Capability Isolation

```text
write_memory
↓
不挂载

partner_memorize
↓
允许挂载
```

## 第二层：Resource Scope Isolation

即使调用：

```text
partner_memorize
```

真正写入的仍然是：

```text
Partner Own Relationship Memory
```

而不是用户 Shared L3。

一句话：

> **Tool Surface 决定“能不能做”，Resource Scope 决定“对谁的数据做”。**

---

# 十八、如果 LLM 知道 `write_memory` 名字怎么办

核心判断：

```text
模型知道 Tool 名字
≠
模型拥有 Tool Capability
```

正常 Partner Turn 里 `write_memory` 已经不在 Final Tool Surface。

因此真正的权限来源是：

```text
Runtime Tool Composition
```

而不是模型意图。

面试金句：

> **Tool authority comes from Runtime composition, not from model intention.**

---

# 十九、为什么不要到处写 `if partner`

错误方式：

```python
if is_partner and tool == "write_memory":
    deny()
```

然后整个 AgentLoop 到处都是 Partner 分支。

更好的方式：

```text
Turn 开始
↓
一次性计算 Final Tool Surface
↓
AgentLoop 只面对最终 Tool List
```

所以：

> **Policy 决策前置到 Tool Composition，AgentLoop 保持通用。**

这也符合 `Least Privilege` 思想：

```text
Agent 只获得当前任务真正需要的能力
```

---

# 二十、跨作用域泄漏率怎么定义

简历中的：

```text
跨作用域泄漏 0%
```

回答的是：

> 当前 Agent 有没有读到不属于当前 Scope 的数据？

可以定义：

```text
Cross-scope Leakage Rate
=
发生非授权跨 Scope 数据读取的测试数
/
所有隔离测试数
```

典型测试：

```text
Partner A 读取 Partner B KB
Partner A 读取 Partner B Notebook
A × User1 读取 A × User2 Memory
A × User1 读取 B × User1 Memory
```

---

# 二十一、越权调用拦截率怎么定义

简历中的：

```text
越权调用拦截率 100%
```

回答的是：

> 当前 Runtime 不允许的 Tool Call 有没有被阻止？

定义：

```text
Unauthorized Call Blocking Rate
=
成功拦截的越权 Tool Call
/
所有越权 Tool Call 测试数
```

典型测试：

```text
Partner 调 write_memory
Partner 调 read_memory
Partner 调 whitelist 外 Tool
Product Chat 调 partner_memorize
```

---

# 二十二、两个指标不要混

```text
Leakage Rate
= 数据有没有泄漏

Blocking Rate
= 非法 Capability 有没有执行
```

一个看：

```text
Data Isolation
```

一个看：

```text
Authorization
```

---

# 二十三、Partner A 为什么理论上不能读 Partner B

资源链：

```text
Partner A
↓
Synthetic Scope A
↓
PathService
↓
Workspace A
```

Partner B：

```text
Partner B
↓
Synthetic Scope B
↓
PathService
↓
Workspace B
```

隔离依赖：

```text
Runtime Scope
+
Resource Resolver
```

而不是：

```text
Prompt：
“不要看 Partner B”
```

---

# 二十四、Canary Data 测试

给不同 Scope 放唯一 Token：

```text
Partner A:
CANARY_PARTNER_A_7F21

Partner B:
CANARY_PARTNER_B_9C83
```

然后从 A 进行：

```text
直接询问
间接询问
搜索所有文件
Prompt Injection
要求调用 Tool
总结所有 KB
搜索历史
```

只要 A 的：

```text
Tool Result
Runtime Context
Final Response
```

任意位置出现：

```text
CANARY_PARTNER_B_9C83
```

就算 Leakage。

---

# 二十五、为什么不能只检查最终回答

假设：

```text
Tool 已经读到 B 的 secret
↓
LLM 最后回答“不知道”
```

如果只检查自然语言回答，会漏掉真正的数据泄漏。

权限测试应该检查：

```text
Final Enabled Tools
Tool Call
Tool Result
Runtime Context
Resource Scope
Memory Before / After
Final Response
```

一句话：

> **权限测试应该尽量在 Runtime / Tool 层断言，而不是只在自然语言层断言。**

---

# 二十六、Tool Policy Test Matrix

| 场景 | 预期 |
|---|---|
| Partner 调允许 Tool | ✅ |
| Partner 调 whitelist 外 Tool | ❌ |
| Partner 调 `write_memory` | ❌ |
| Partner 调 `read_memory` | ❌ |
| Partner 调 `partner_memorize` | ✅ |
| Product Chat 调 `partner_memorize` | ❌ / 不存在 |
| Partner 无 KB 时 RAG | 不自动挂载 |
| Partner 有 KB 且允许 RAG | ✅ |

---

# 二十七、Cross-Scope Test Matrix

| 当前 Scope | 尝试读取 | 预期 |
|---|---|---|
| Partner A | A KB | ✅ |
| Partner A | B KB | ❌ |
| Partner A | A Notebook | ✅ |
| Partner A | B Notebook | ❌ |
| A × User1 | A × User1 Memory | ✅ |
| A × User1 | A × User2 Memory | ❌ |
| A × User1 | B × User1 Memory | ❌ |
| Partner A | User Shared L3 | ✅ Read |
| Partner A | 修改 User Shared L3 | ❌ |

注意：

```text
Partner 读取 User Shared L3
```

本身是设计允许的，不算泄漏。

---

# 二十八、怎么解释“跨作用域泄漏 0%”

推荐回答：

> 我把隔离测试按 Partner 和 User 两个维度做成矩阵，例如 A 读取 B 的 KB、Notebook、Skill、Relationship Memory，以及同一 Partner 下 User1 尝试读取 User2 的关系记忆。测试数据里放唯一 Canary Token，不只检查最终回答，还检查 Tool Result 和 Runtime Context；只要非授权 Canary 出现在任一层就判定为泄漏。在我的回归测试集里没有发现跨作用域数据，所以记录为 0% leakage。

注意：

> 具体样本数必须与你实际测试一致，不要现场编。

---

# 二十九、怎么解释“越权调用拦截率 100%”

推荐回答：

> 越权测试主要覆盖 Partner 配置不允许的 Built-in Tool，以及明确禁止的普通 Chat Memory Tool，例如 `read_memory`、`write_memory`。测试会直接检查 `compose_enabled_tools` 的 Final Tool Surface，也会构造非法 Tool Call 验证不会真正进入 Tool Execution。在我的回归测试集中，预定义越权场景全部被拦截，所以记录为 100%。

---

# 三十、0% / 100% 不是绝对安全证明

不要说：

```text
系统绝对不会泄漏。
```

正确说法：

> 0% 和 100% 是当前回归测试集上的结果，不代表数学意义上的绝对安全。

架构提供的是：

```text
Synthetic Scope
Tool Surface
Resource Resolver
Path Isolation
```

这些确定性隔离机制。

测试指标只是：

```text
当前已覆盖场景的验证结果
```

---

# 三十一、完整架构图

```text
Partner Config
│
├── Soul
├── KB
├── Skill
├── Notebook
├── Tool Policy
└── Memory Policy
       │
       ↓
Synthetic Partner Scope
       │
       ↓
Independent Workspace
       │
       ↓
UnifiedContext
       │
       ↓
Tool Composition
├── whitelist
├── forced
└── suppressed
       │
       ↓
Final Tool Surface
       │
       ↓
Shared Agent Runtime
       │
       ↓
AgentLoop
```

Memory：

```text
User Shared L3
      ↓
   Read Only

Partner × User Relationship Memory
      ↓
   Read / Write
```

安全验证：

```text
Scope Isolation Tests
+
Tool Authorization Tests
+
Canary Data
```

---

# 三十二、完整面试回答

> Partner 没有单独实现一套 Agent Engine，而是复用产品 Chat 的 `TurnEngine → AgenticChatPipeline → AgentLoop`。不同 Partner 的差异通过 `UnifiedContext` 和 Synthetic User Scope 注入，每个 Partner 拥有独立 Workspace，里面装配 Soul、KB、Skill、Notebook 和 Memory，所以 RAG、Skill、Notebook 等已有 Tool 在同一套 Runtime 下会自然解析到当前 Partner 的资源。
>
> Tool 权限不是靠 Prompt 约束，而是在每个 Turn 前通过 `compose_enabled_tools` 生成独立 Tool Surface。`builtin_whitelist` 限制最多允许挂载哪些 Built-in；`forced` 保证 `partner_read`、`partner_memorize`、`partner_search` 等 Partner 专属能力始终存在；`suppressed` 则把普通 Chat 的 `read_memory` 和 `write_memory` 无条件移除。
>
> Memory 采用 Split Model：用户 Shared L3 对 Partner 只读，Partner 自己的关系记忆按 `Partner × User` 隔离，而且 `partner_memorize` 只能写这个独立 Memory Scope。因此我们实际上做了 Tool Surface 和 Resource Scope 两层权限控制。
>
> 隔离测试上，我不会只看最终回答，而会给不同 Partner / User Scope 放唯一 Canary Data，同时检查 Final Tool Surface、Tool Result、Runtime Context 和 Memory Before/After。简历里的跨作用域泄漏 0% 和越权调用拦截率 100%，应该理解为当前回归测试集上的结果，而不是绝对安全证明。

---

# 三十三、常见面试追问

## Q1：为什么 Partner 不单独做 Agent Engine？

> 因为 Partner 的核心差异在身份、数据和能力配置，而不是 Agent Loop。单独做 Engine 会重复维护 Tool Calling、Streaming、RAG、Memory 等 Runtime 能力，所以更适合复用统一 Runtime，把差异放进 Context 和 Scope。

## Q2：`UnifiedContext` 的作用是什么？

> 把当前 Turn 的身份、资源、Tool Policy 等运行信息传给统一 Agent Runtime，使 Runtime 不需要硬编码不同 Partner 的分支。

## Q3：Synthetic Scope 和 UnifiedContext 有什么区别？

> UnifiedContext 是当前 Turn 的 Agent 运行上下文；Synthetic Scope 决定底层资源访问解析到哪个 Partner Workspace。

## Q4：为什么 Workspace 和普通 User Workspace 用相同格式？

> 这样 RAG、Skill、Notebook 等已有 Tool 可以直接复用，只要切换 Scope 就能自然读取 Partner 自己的 Workspace。

## Q5：Whitelist / Forced / Suppressed 分别是什么？

> Whitelist 表示最多允许什么；Forced 表示必须有什么；Suppressed 表示绝对不能有什么。

## Q6：为什么 Partner 不能用普通 `write_memory`？

> 因为用户 Shared L3 对 Partner 是只读 Context，Partner 只能通过 `partner_memorize` 写自己的 `Partner × User` Relationship Memory。

## Q7：如果 LLM 知道 `write_memory` 的名字呢？

> Tool 权限来自 Runtime 的 Final Tool Surface，不来自模型意图。知道名字不等于拥有 Capability。

## Q8：为什么既限制 Tool，又限制 Resource Scope？

> Tool Surface 解决“能不能做”，Resource Scope 解决“对谁的数据做”。两层同时限制，才能避免允许的 Tool 被用于错误的数据作用域。

## Q9：怎么测跨 Partner 泄漏？

> 给不同 Partner 的 KB、Notebook、Memory 放唯一 Canary Token，然后从其他 Scope 做直接询问、搜索、Prompt Injection 等攻击，同时检查 Tool Result、Runtime Context 和最终输出是否出现非授权 Token。

## Q10：0% 泄漏是不是绝对安全？

> 不是，它只是当前回归测试集结果。架构层通过 Scope、Tool Surface 和 Resource Resolver 提供确定性隔离，但测试指标不能等同于绝对安全。

---

# 三十四、最终速记

先记：

```text
Shared Runtime
Synthetic Scope
Independent Workspace
UnifiedContext
```

再记：

```text
Whitelist
= 最多能有什么

Forced
= 必须有什么

Suppressed
= 绝对不能有什么
```

再记：

```text
User Shared L3
= Read Only

Partner × User Relationship Memory
= Read / Write
```

最后记：

```text
Tool Surface
决定“能不能做”

Resource Scope
决定“对谁的数据做”
```

以及：

> **Partner 不是“另一套 Agent”，而是“同一套 Agent Runtime + 不同 UnifiedContext + 不同 Synthetic Scope + 独立 Workspace + 独立 Tool / Memory Policy”。**
