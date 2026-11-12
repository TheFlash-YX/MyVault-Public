# DeepTutor / KindPartner 面试复习

## 第 1 点：`ask_user` 同 Turn 内容澄清机制

> 目标：能在面试中解释 Human-in-the-loop 的暂停 / 恢复、Tool 协议、Runtime 状态、上下文连续性和边界。

---

## 一、30 秒面试回答

我们没有把 `ask_user` 当成普通问答 Tool，而是把它设计成 Agent Runtime 的 **pause / resume 协议**。

当模型发现信息不足时调用 `ask_user`，Tool 返回带 `pause_for_user` 的 `ToolResult`，Tool Dispatcher 再把这个控制信号汇总成 `DispatchOutcome.pause = True`。AgentLoop 发现暂停后，会调用 TurnRuntime 注入的异步 waiter，并通过 Turn 级 `asyncio.Queue` 等待用户输入。

用户回复后，Runtime 不会重新创建一个新的 Turn，而是根据原始 `tool_call_id`，把用户回答回填到对应的 `role=tool` 消息中，然后恢复同一个 AgentLoop。

这样保留的不只是聊天历史，还包括完整的 Tool Calling 协议链、Loop state 和 Capability 的 Turn 内状态，因此能够减少重新恢复执行现场带来的上下文损失。

---

# 二、为什么需要同 Turn 澄清

假设用户说：

```text
帮我制定一个学习计划。
```

模型发现还缺信息：

```text
学习什么？
每天多少时间？
准备多久？
```

最简单的实现可能是：

```text
Turn 1
用户 → Agent
Agent：“你每天有多少时间？”
Turn 1 结束

Turn 2
用户：“2 小时”
重新创建一次 Agent 执行
```

这种方式的问题是：

```text
第一次 Agent 的运行现场已经结束。

前面调过什么 Tool？
为什么问这个问题？
下一步准备做什么？
当前 Capability 的状态是什么？

这些都需要重新恢复。
```

DeepTutor 希望实现的是：

```text
Turn 1

用户
 ↓
LLM
 ↓
Tool
 ↓
LLM
 ↓
发现缺信息
 ↓
ask_user
 ↓
[PAUSE]
 ↓
用户：“2 小时”
 ↓
[RESUME]
 ↓
LLM
 ↓
继续 Tool
 ↓
最终回答
```

整个过程仍然属于同一个 Turn。

这就是典型的：

**Human-in-the-loop Agent Runtime**

---

# 三、完整执行链路

先记住这一条主链：

```text
LLM
 ↓
① 返回 ask_user Tool Call
 ↓
② AskUserTool.execute()
 ↓
③ ToolResult(pause_for_user=...)
 ↓
④ Tool Dispatcher 汇总
 ↓
⑤ DispatchOutcome(
      pause=True,
      pause_payload=...,
      pause_tool_call_id=...
   )
 ↓
⑥ AgentLoop 发现 pause
 ↓
⑦ await wait_for_user_reply()
 ↓
⑧ TurnRuntime 的 asyncio.Queue 挂起
 ↓
⑨ 用户 submit_user_reply()
 ↓
⑩ queue.put(...)
 ↓
⑪ waiter 恢复
 ↓
⑫ 根据 pause_tool_call_id 找到对应 role=tool
 ↓
⑬ 把用户回复回填进去
 ↓
⑭ continue
 ↓
⑮ 同一个 AgentLoop 进入下一轮 LLM
```

面试时如果能完整讲清这条链，第一点基本就立住了。

---

# 四、`ask_user` 本身并不负责“等待用户”

这是一个非常重要的设计点。

普通 Tool 一般返回：

```python
ToolResult(
    content="工具执行结果"
)
```

而 `ask_user` 返回的特殊点是：

```python
ToolResult(
    content="[awaiting user reply ...]",
    metadata={
        "ask_user": payload
    },
    pause_for_user=payload
)
```

也就是说：

```text
AskUserTool
并不会自己去 await 用户输入
```

它只是告诉 Runtime：

```text
“我的 Tool 执行已经结束，
但是整个 Turn 现在不能继续，
需要等待真人输入。”
```

因此职责划分是：

```text
Tool Layer
负责声明：
“我要暂停”

Runtime Layer
负责决定：
“怎么暂停、怎么恢复”
```

这使 `ask_user` 不需要直接依赖：

```text
WebSocket
TurnRuntime
Session
asyncio.Queue
前端 UI
```

Tool 层和 Runtime 层保持解耦。

---

# 五、`ToolResult`：Tool 到 Runtime 的统一协议

DeepTutor 中所有 Tool 都返回统一的：

```python
ToolResult
```

可以把它理解成三类信息：

```text
ToolResult
│
├── 数据结果
│   ├── content
│   ├── sources
│   └── metadata
│
├── 执行结果
│   └── success
│
└── 控制信号
    ├── terminate_turn
    └── pause_for_user
```

## 普通 Tool

比如 RAG：

```python
ToolResult(
    content="检索结果",
    sources=[...]
)
```

它只返回数据，不改变 Agent Loop 的控制流程。

## `ask_user`

```python
ToolResult(
    content="...",
    pause_for_user={
        "questions": [...]
    }
)
```

此时 `ToolResult` 不仅是数据通道，也成为：

**Tool → Agent Runtime 的控制通道**

---

# 六、`pause_for_user` 和 `terminate_turn` 的区别

这两个字段不要混。

## `terminate_turn`

表示：

```text
这个 Tool 执行完以后
整个 Turn 可以结束。
```

也就是：

```text
Running
 ↓
Completed
```

## `pause_for_user`

表示：

```text
任务还没完成，
只是现在缺少用户输入。
```

状态更像：

```text
Running
 ↓
Paused
 ↓
Running
 ↓
Completed
```

所以：

```text
pause ≠ finish
```

这是 `ask_user` 能实现同 Turn 恢复的基础。

---

# 七、Tool Dispatcher 为什么还需要 `DispatchOutcome`

Agent 一轮可能不只调用一个 Tool：

```text
LLM
 ↓
[
  rag(...),
  notebook(...),
  ask_user(...)
]
```

因此 AgentLoop 不应该直接处理每个 `ToolResult`。

Tool Dispatcher 会把一整轮的 Tool 执行结果汇总成：

```python
DispatchOutcome
```

核心字段可以理解为：

```python
DispatchOutcome(
    sources=[...],
    tool_messages=[...],

    terminate=False,
    terminate_payload=None,

    pause=True,
    pause_payload={...},
    pause_tool_call_id="call_x"
)
```

所以关系是：

```text
多个 ToolResult
      ↓
Tool Dispatcher
      ↓
一个 DispatchOutcome
      ↓
AgentLoop
```

`DispatchOutcome` 是 Tool 执行层和 Agent Loop 控制层之间的桥梁。

---

# 八、为什么 `ask_user` 是 Pausing Tool

Dispatcher 把：

```python
ask_user
```

看作特殊的暂停型 Tool。

执行顺序可以理解为：

```text
① rebinding tools
   串行

② ordinary tools
   尽量并行

③ pausing tools
   最后执行
```

也就是说，如果一轮模型同时返回：

```text
rag(...)
ask_user(...)
```

不会一看到 `ask_user` 就立刻暂停。

而是：

```text
先执行 rag
 ↓
得到这一轮需要的状态 / 数据
 ↓
再执行 ask_user
 ↓
暂停
```

---

# 九、为什么 `ask_user` 要最后执行

举一个典型场景：

```text
① mastery_quiz(...)
② ask_user(...)
```

`mastery_quiz` 可能先把题目真正写入状态或持久化：

```text
Question ID = q123
```

而 `ask_user` 最终展示给用户的，应该是：

```text
已经落地后的真实 Question
```

而不是模型刚生成的临时版本。

因此：

```text
mastery_quiz
 ↓
状态更新
 ↓
重新绑定 ask_user 参数
 ↓
ask_user
 ↓
PAUSE
```

所以暂停型 Tool 放在一轮 Tool 执行的最后是有工程意义的。

---

# 十、TurnRuntime 如何真正等待用户

真正负责“等人”的不是 `AskUserTool`，而是 `TurnRuntime`。

TurnRuntime 会给当前：

```text
turn_id
```

维护一个：

```python
asyncio.Queue
```

概念上类似：

```python
reply_queue = asyncio.Queue()

self._reply_queues[turn_id] = reply_queue
```

然后定义一个 waiter：

```python
async def _wait_for_user_reply():
    execution.awaiting_user_reply = True

    reply = await reply_queue.get()

    execution.awaiting_user_reply = False

    return reply
```

关键是：

```python
await reply_queue.get()
```

这不是轮询。

不是：

```python
while True:
    check_database()
    sleep(1)
```

而是：

```text
Agent coroutine
 ↓
await Queue
 ↓
挂起
 ↓
Event Loop 去执行其他任务
```

因此等待用户期间：

```text
不会 CPU 空转
不会重复查询数据库
不会继续调用 LLM
不会继续消耗推理 Token
```

这是一个典型的异步事件驱动设计。

---

# 十一、为什么用 `asyncio.Queue`

这个场景本质上就是：

**生产者 - 消费者模型**

```text
消费者：
AgentLoop / TurnRuntime
await queue.get()

生产者：
用户通过 WebSocket / API 提交回答
queue.put(reply)
```

用户没回答：

```text
Agent coroutine suspended
```

用户一回答：

```text
queue.put(...)
 ↓
原 coroutine 被唤醒
```

因此非常适合 Human-in-the-loop 场景。

---

# 十二、为什么通过 `UnifiedContext` 注入 waiter

如果 `AgenticPipeline` 直接：

```python
import TurnRuntime
```

然后自己访问：

```text
WebSocket
Queue
Session
```

会导致 Runtime 和 Agent Pipeline 强耦合。

DeepTutor 的思路是：

```text
TurnRuntime
 ↓
创建 _wait_for_user_reply callback
 ↓
放入 context.metadata
 ↓
UnifiedContext
 ↓
AgentPipeline
```

Agent Pipeline 只需要：

```python
waiter = context.metadata.get(
    "wait_for_user_reply"
)
```

然后：

```python
reply = await waiter()
```

于是依赖方向是：

```text
TurnRuntime
     ↓
注入 callback
     ↓
UnifiedContext
     ↓
AgentPipeline
```

而不是：

```text
AgentPipeline
     ↓
直接依赖 TurnRuntime
```

可以把它理解成一种轻量级：

**Dependency Injection**

---

# 十三、用户回答后发生什么

假设模型之前生成：

```text
tool_call_id = call_123

ask_user(
    "你每天可以学习多久？"
)
```

用户回答：

```text
2 小时
```

外层会通过：

```text
turn_id
```

找到这一条正在等待的 Turn。

然后：

```text
reply_queue.put(...)
```

之前阻塞在：

```python
await reply_queue.get()
```

的 coroutine 被唤醒。

但是这里还没结束。

Runtime 还需要解决另一个问题：

```text
这个回答属于哪个 Tool Call？
```

答案就是：

```text
pause_tool_call_id
```

---

# 十四、`turn_id` 和 `tool_call_id` 的区别

这是面试里很好用的一句话：

> `turn_id` 定位的是“哪个 Turn”，`tool_call_id` 定位的是“这个 Turn 里的哪个 Tool Call”。

例如：

```text
turn_id = turn_001
```

这一轮里面可能有：

```text
call_1 → rag
call_2 → notebook
call_3 → ask_user
```

当用户回答时：

```text
turn_id
 ↓
找到当前 Turn 的 Queue

pause_tool_call_id = call_3
 ↓
找到这一 Turn 里的 ask_user
```

然后才能把用户回答准确写回：

```text
call_3 对应的 role=tool
```

---

# 十五、为什么用户回复不是新的 `role=user`

这是“同 Turn”的关键。

传统方式可能是：

```text
Assistant:
你每天能学习多久？

User:
2 小时
```

这种关联主要靠：

**自然语言语义**

模型需要自己理解：

```text
“2 小时”
是在回答上一句话。
```

而 Tool Calling 方式是：

```text
assistant.tool_calls:
{
    id: "call_123",
    name: "ask_user"
}

tool:
{
    tool_call_id: "call_123",
    content: "用户回答：2 小时"
}
```

这时两者关系是：

```text
call_123
 ↓
协议级精确关联
```

因此可以总结为：

```text
普通 Chat：
语义关联

同 Turn ask_user：
协议级关联
```

面试时可以说：

> 我们不只是依赖自然语言历史让模型猜用户这句话在回答什么，而是通过 `tool_call_id` 保持协议级关联。

---

# 十六、为什么它能保持上下文连续性

同 Turn 方案保留的并不只是聊天记录。

可以分三层理解。

## 1. Conversation State

```text
messages
```

其中包含：

```text
user
assistant tool_call
role=tool result
后续 assistant
```

## 2. Runtime State

例如：

```text
rounds
exploration_rounds
settlement_rounds
tool_steps
sources
```

## 3. Capability / Control State

例如：

```text
context.metadata
Capability 的 Turn 内 scratch state
当前 pause 状态
```

因此同 Turn 保存的是：

```text
Conversation State
+
Runtime State
+
Control State
```

而不是只有历史聊天。

---

# 十七、为什么新 Turn 更复杂

假设 Agent 已经执行：

```text
LLM
 ↓
Tool A
 ↓
Tool B
 ↓
修改状态
 ↓
Tool C
 ↓
ask_user
```

如果这个时候结束，再重新创建 Turn：

```text
New Turn
```

Runtime 就需要重新判断：

```text
Tool A 是否已经执行？
Tool B 的结果在哪？
Tool C 有没有副作用？
之前选中的目标是什么？
为什么 ask_user？
下一步原计划是什么？
```

这已经从：

```text
pause / resume
```

升级成：

```text
Workflow Recovery
Checkpoint Recovery
```

工程复杂度会明显上升。

DeepTutor 当前的方案则是：

```text
不要结束 coroutine
 ↓
直接 suspend execution
 ↓
用户回来
 ↓
resume
```

因此在线场景下非常直接。

---

# 十八、同 Turn 的真正本质

面试时不要只说：

```text
因为 turn_id 没变
```

那只是表面。

真正原因是三个东西都还存在：

```text
① 同一个 AgentLoop coroutine

② 同一个 messages 列表

③ 同一条 Tool Calling 协议链
   assistant.tool_call
          ↓
      role=tool
```

Pause 前：

```text
messages
AgentLoop state
tool_call_id
context
```

都还活着。

用户回复只是：

```text
补齐 pending Tool Result
```

然后：

```text
continue
```

进入 AgentLoop 下一轮。

---

# 十九、同一轮多个 `ask_user` 怎么处理

Runtime 当前一次只应该维护一个 pending 的 Ask Me 状态。

所以如果模型一轮生成：

```text
ask_user("每天多久？")

ask_user("目标是什么？")
```

后面的 `ask_user` 会被视为重复 / 不应独立执行。

正确方式是：

```python
ask_user(
    questions=[
        "你每天可以学习多久？",
        "你的目标是什么？"
    ]
)
```

也就是：

```text
一轮只保持一个 ask_user Tool Call
内部可以包含多个 questions
```

这样 Runtime 才只有一个明确的 pending reply。

---

# 二十、为什么 `ask_user` 不是直接写在 AgentLoop 里

如果把逻辑硬编码成：

```python
if model_says_need_more_info:
    ask_user(...)
```

会让 AgentLoop 和具体业务行为耦合。

现在的结构是：

```text
LLM
 ↓
Tool Calling Protocol
 ↓
AskUserTool
 ↓
ToolResult.pause_for_user
 ↓
Dispatcher
 ↓
AgentLoop
```

也就是说：

```text
AgentLoop
不需要理解“为什么提问”

它只理解：
pause / terminate / continue
```

这让 Runtime 更通用。

---

# 二十一、职责边界

可以把整个架构按职责记成：

```text
AskUserTool
│
└── 描述需要用户输入
    返回 pause_for_user

Tool Dispatcher
│
└── 执行 Tool
    汇总 ToolResult
    生成 DispatchOutcome

AgentLoop
│
└── 读取 pause / terminate
    控制循环继续还是等待

AgenticPipeline
│
└── 从 UnifiedContext 获取 waiter
    组织 pause / resume

TurnRuntime
│
└── 管理 turn_id
    asyncio.Queue
    submit_user_reply
    live execution

Frontend / WebSocket
│
└── 展示问题
    提交用户回复
```

一句话总结：

> Tool 层负责声明控制意图，Dispatcher 负责归一化，AgentLoop 负责控制流程，TurnRuntime 负责具体的异步生命周期。

---

# 二十二、这套设计的优点

## 1. 上下文完整

无需重建 Agent 执行现场。

## 2. Function Calling 协议完整

用户回答能精确回填原：

```text
tool_call_id
```

## 3. Runtime 与 Tool 解耦

`AskUserTool` 不依赖 WebSocket / Queue。

## 4. 异步非阻塞

通过：

```python
await asyncio.Queue.get()
```

挂起 coroutine。

## 5. 易于扩展

未来不仅：

```text
ask_user
```

理论上其他需要真人审批的 Tool，也可以通过相似控制信号接入 Runtime。

---

# 二十三、局限：它不是 Durable Workflow

这是非常重要的边界。

当前方案优势是：

```text
在线
同进程
低延迟
上下文完整
实现相对直接
```

但 Pause 状态很大程度依赖：

```text
live coroutine
+
内存中的 Queue
+
当前 AgentLoop state
```

如果：

```text
服务器进程崩溃
服务重启
```

原来的：

```python
await queue.get()
```

执行现场不会像 Temporal 这类 Durable Workflow Engine 一样自动从 checkpoint 恢复。

因此更准确的描述是：

**In-memory Human-in-the-loop Agent Runtime**

而不是：

```text
跨进程 Durable Resume Runtime
```

面试时不要把这一点吹过头。

---

# 二十四、Web Chat 与 Partner IM 的边界

这一点也需要讲准确。

Web Chat 的 Turn Runtime 可以注入：

```text
wait_for_user_reply
```

所以支持：

```text
same Turn
pause
resume
```

但 Partner 的 IM 场景语义不同。

例如微信、Telegram 这类 IM：

```text
Agent 调用 ask_user
 ↓
把问题作为当前 Turn 的回复发出去
 ↓
当前 Turn 结束

用户下一条 IM
 ↓
开启新的 Turn
```

因此不要说：

```text
DeepTutor 所有 ask_user 场景
都是同 Turn pause/resume
```

更准确的是：

> Web Turn Runtime 支持真正的同 Turn pause/resume；Partner IM 更接近“当前 Turn 发出问题，下一条 IM 开启新 Turn”的消息语义。

---

# 二十五、关于“上下文连续率 98%”应该怎么解释

如果简历写了：

```text
上下文连续率 98%
```

面试官很可能会问：

```text
98% 是怎么测出来的？
```

这个数字必须有清晰定义。

合理的测试方法应该是：

构造 N 个需要 `ask_user` 的多轮任务，判断恢复后：

```text
1. 是否仍然对应原 Turn
2. 是否正确关联原 tool_call_id
3. 是否保留 pause 前关键 Tool Result
4. 是否正确使用用户补充信息
5. 是否最终完成原始任务
```

例如：

```text
100 条测试任务

98 条能够正确：
ask_user
→ pause
→ resume
→ 利用之前 Tool Result
→ 完成原任务

上下文连续成功率 = 98 / 100 = 98%
```

需要注意：

> 公开 DeepTutor 代码可以证明这套机制，但不能单独证明简历里的 98% 指标。

如果面试官问，应该明确说明：

```text
这是自己构造的业务测试集结果，
不是 DeepTutor 官方 Benchmark。
```

---

# 二十六、为什么可能不是 100%

这里可以区分：

```text
Runtime Continuity
```

和：

```text
Model Semantic Correctness
```

Runtime 可以保证：

```text
messages 没丢
tool_call_id 没断
Tool Result 被正确回填
AgentLoop 能恢复
```

但它不能保证：

```text
LLM 一定正确理解用户回答
LLM 一定利用所有历史 Tool Result
LLM 不会因为长上下文忽略关键信息
```

所以：

> Runtime continuity ≠ Model semantic correctness

这是一句很好的面试回答。

---

# 二十七、常见面试追问

## Q1：为什么要用 Queue？

推荐回答：

> 因为它本质上是一个生产者-消费者问题。Agent 是消费者，在 `await queue.get()` 处挂起；用户通过 WebSocket / API 回复时作为生产者，把回答写进当前 Turn 的 Queue。这样不需要轮询，也不会阻塞整个 asyncio Event Loop。

---

## Q2：为什么不直接重新开启一个 Turn？

推荐回答：

> 新 Turn 虽然可以重放历史对话，但会丢失原来的 live Agent execution state，而且 `ask_user` 对应的 Tool Calling 协议链需要重新恢复。同 Turn 方案保留原 messages、AgentLoop state 和 `tool_call_id`，用户回答只需要补齐 pending Tool Result，因此上下文和控制流更稳定。

---

## Q3：怎么知道用户回复的是哪个问题？

推荐回答：

> 两级定位。外层通过 `turn_id` 找到正在等待的 Turn 和它的 reply queue，内层通过 `pause_tool_call_id` 找到原来的 `ask_user` Tool Call，然后把用户回答回填到对应的 `role=tool` message。

---

## Q4：为什么不直接把回答作为新的 `role=user`？

推荐回答：

> 因为 `ask_user` 是一个 Tool Call。把回答作为对应的 `role=tool` 结果，可以通过 `tool_call_id` 保持协议级关联；如果只是新的 `role=user`，主要依赖模型从自然语言历史中重新猜测这句话在回答什么。

---

## Q5：为什么 Tool Dispatcher 不直接等待用户？

推荐回答：

> Dispatcher 应该是通用 Tool 执行层，不应该知道 WebSocket、Session 或具体 UI。`AskUserTool` 只通过 `ToolResult.pause_for_user` 声明控制意图，Dispatcher 转成 `DispatchOutcome.pause`，真正的等待由 Runtime 注入的 waiter 完成，从而保持 Tool 层和 Runtime 层解耦。

---

## Q6：用户一直不回复怎么办？

推荐回答：

> 当前机制主要面向在线 live Turn。Agent coroutine 会处于等待状态，是否进一步做超时、取消或资源回收属于 TurnRuntime 生命周期策略。它并不是一个跨进程 Durable Workflow，因此服务重启后不能简单依赖原内存 Queue 恢复。

---

## Q7：如果模型一轮调用多个 `ask_user` 呢？

推荐回答：

> Runtime 一次只应该维护一个 pending Ask Me 状态，所以同一批 Tool Call 里只保留一个有效的 `ask_user`。多个澄清问题应该合并到同一个 `ask_user` 的 `questions` 列表中。

---

## Q8：如果一轮同时调用 RAG 和 `ask_user` 呢？

推荐回答：

> 普通 Tool 先执行，`ask_user` 作为 pausing tool 最后执行。这样它展示给用户的问题可以基于这一轮其他 Tool 已经产生或持久化的最新状态，而不是基于旧参数。

---

## Q9：为什么这个机制比普通多轮聊天更适合 Agent？

推荐回答：

> 普通聊天主要需要维护语言上下文，而 Agent 还存在 Tool Result、执行阶段、Capability 状态和副作用等运行时状态。中途重新开 Turn 会逐渐变成一个 Workflow Recovery 问题。同 Turn suspend/resume 可以直接保留 live execution state。

---

## Q10：它最大的工程限制是什么？

推荐回答：

> 最大限制是 pause 状态主要依赖内存中的 coroutine 和 Queue，因此它适合在线 human-in-the-loop，但不是 Temporal 那种 durable workflow。进程重启后的 mid-loop 恢复需要额外的 checkpoint 和状态重建能力。

---

# 二十八、两分钟完整面试回答

如果面试官让你详细展开，可以使用下面这个结构：

> 这个功能的背景是 Agent 在执行复杂任务时，经常会遇到信息不足。最简单的方案是让 Agent 输出一个问题，然后结束当前 Turn，用户回答后再新建一个 Turn。但这种方式会打断原来的 Tool Calling 链路，而且需要重新恢复 Tool Result、执行状态以及为什么提出这个问题。
>
> 所以我们把 `ask_user` 设计成了一个 pause/resume 协议。`AskUserTool` 本身不会直接等待用户，而是返回一个带 `pause_for_user` 的统一 `ToolResult`。Tool Dispatcher 执行完本轮 Tool 后，把这个信号汇总成 `DispatchOutcome.pause=True`，同时保存 `pause_payload` 和 `pause_tool_call_id`。
>
> AgentLoop 检测到 pause 后，从 `UnifiedContext.metadata` 中获取 Runtime 注入的 `wait_for_user_reply` callback。TurnRuntime 为每个 Turn 维护一个 `asyncio.Queue`，Agent 在 `await queue.get()` 处挂起，因此不会阻塞 Event Loop，也不会继续调用 LLM。
>
> 用户通过 WebSocket 或 API 回答后，Runtime 用 `turn_id` 找到对应 Queue，把回答写进去。原 coroutine 被唤醒以后，再通过 `pause_tool_call_id` 找到对应的 `role=tool` 消息，把用户回答回填进去，最后 `continue` 同一个 AgentLoop。
>
> 这个方案最大的价值是保留的不只是自然语言对话历史，还包括 Tool Calling 的协议关联、AgentLoop state 和 Capability 的 Turn 内状态。因此不需要重新做 Workflow Recovery。但它的边界是当前主要属于内存中的 live pause/resume，如果进程重启，并不是一个完整的 Durable Workflow 方案。

---

# 二十九、最终速记

面试前只记这一条：

```text
LLM
 ↓
ask_user
 ↓
ToolResult.pause_for_user
 ↓
DispatchOutcome.pause
 ↓
AgentLoop
 ↓
Runtime waiter
 ↓
asyncio.Queue
 ↓
用户回答
 ↓
turn_id 找 Turn
 ↓
pause_tool_call_id 找 Tool
 ↓
回填 role=tool
 ↓
continue
 ↓
同一个 AgentLoop
```

再记 5 句话：

```text
1. ask_user 本身不等待用户，它只发出 pause 控制信号。

2. ToolResult 是 Tool → Runtime 的统一数据 + 控制协议。

3. turn_id 定位 Turn，tool_call_id 定位 Turn 内具体 Tool Call。

4. 用户回复回填 role=tool，而不是简单新建 role=user。

5. 它是 live in-memory pause/resume，不是完整 Durable Workflow。
```

---

# 三十、一个最核心的理解

这一点真正想解决的并不是：

```text
“Agent 怎么问用户一个问题？”
```

而是：

```text
“一个正在执行的 Agent Workflow，
如何在不丢失执行现场的情况下，
插入一次真人交互，
然后继续原任务？”
```

这才是这个简历点的工程价值。
