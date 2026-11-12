---
type: interview
scope: topic
area: agents
topics:
  - agent-runtime
  - agent-loop
  - function-calling
  - turn-lifecycle
created: 2026-08-27
updated: 2026-08-27
---

# Agent Runtime 与 Agent Loop

- 原始题库只提供问题；回答框架只定义组织方式，不据此推断项目事实或指标证据。

## 一轮 Agent Loop 包含哪些阶段？

<!-- canonical_key: agents.runtime.yi-lun-agent-loop-bao-han-na-xie-jie-duan -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## Agent 在什么情况下继续调用工具，什么情况下结束？

<!-- canonical_key: agents.runtime.agent-zai-shen-me-qing-kuang-xia-ji-xu-diao-yong-gong-ju-shen-me-qing-kuang-xia-jie-shu -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 如何防止 Agent 无限循环？

<!-- canonical_key: agents.runtime.ru-he-fang-zhi-agent-wu-xian-xun-huan -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 最大迭代次数如何设置？

<!-- canonical_key: agents.runtime.zui-da-die-dai-ci-shu-ru-he-she-zhi -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 如果模型一直重复调用同一个工具怎么办？

<!-- canonical_key: agents.runtime.ru-guo-mo-xing-yi-zhi-zhong-fu-diao-yong-tong-yi-ge-gong-ju-zen-me-ban -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## Tool Calling 的状态如何维护？

<!-- canonical_key: agents.runtime.tool-calling-de-zhuang-tai-ru-he-wei-hu -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## Tool Result 是如何重新传递给模型的？

<!-- canonical_key: agents.runtime.tool-result-shi-ru-he-zhong-xin-chuan-di-gei-mo-xing-de -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 多次 Tool Call 是串行还是并行？

<!-- canonical_key: agents.runtime.duo-ci-tool-call-shi-chuan-xing-hai-shi-bing-xing -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 如果模型一次返回多个 Tool Call，你们怎么处理？

<!-- canonical_key: agents.runtime.ru-guo-mo-xing-yi-ci-fan-hui-duo-ge-tool-call-ni-men-zen-me-chu-li -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## Context 太大会不会导致 Prompt 过长？

<!-- canonical_key: agents.runtime.context-tai-da-hui-bu-hui-dao-zhi-prompt-guo-zhang -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 你如何管理 Agent 的上下文窗口？

<!-- canonical_key: agents.runtime.ni-ru-he-guan-li-agent-de-shang-xia-wen-chuang-kou -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## Agent Runtime 有没有状态机？

<!-- canonical_key: agents.runtime.agent-runtime-you-mei-you-zhuang-tai-ji -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## Agent Runtime 中哪些状态必须持久化，哪些可以只存在内存？

<!-- canonical_key: agents.runtime.agent-runtime-zhong-na-xie-zhuang-tai-bi-xu-chi-jiu-hua-na-xie-ke-yi-zhi-cun-zai-nei-cun -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## Agent 的可恢复执行需要保存哪些信息？

<!-- canonical_key: agents.runtime.agent-de-ke-hui-fu-zhi-xing-xu-yao-bao-cun-na-xie-xin-xi -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 如果某个 Tool 有副作用，Agent 重试时如何避免重复执行？

<!-- canonical_key: agents.runtime.ru-guo-mou-ge-tool-you-fu-zuo-yong-agent-zhong-shi-shi-ru-he-bi-mian-zhong-fu-zhi-xing -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 你如何区分模型生成阶段、工具执行阶段和结果汇总阶段？

<!-- canonical_key: agents.runtime.ni-ru-he-qu-fen-mo-xing-sheng-cheng-jie-duan-gong-ju-zhi-xing-jie-duan-he-jie-guo-hui-zong-jie-duan -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## 如果 Tool 调用耗时很长，Runtime 如何支持取消？

<!-- canonical_key: agents.runtime.ru-guo-tool-diao-yong-hao-shi-hen-zhang-runtime-ru-he-zhi-chi-qu-xiao -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)

## Runtime 中如何统一处理异常？

<!-- canonical_key: agents.runtime.runtime-zhong-ru-he-tong-yi-chu-li-yi-chang -->

### 考察点

- Agent Loop 的控制信号、状态边界、终止条件与异常恢复

### 回答框架

- 先定义职责边界，再说明正常执行链路、终止/失败路径和工程取舍；只引用已验证的实现。

### 易追问

-

### 关联笔记

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](../../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
