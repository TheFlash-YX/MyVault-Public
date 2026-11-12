---
type: interview
scope: topic
area: agents
topics:
  - function-calling
  - tools
  - tool-execution
created: 2026-08-27
updated: 2026-09-05
---

# Function Calling 与 Tool System

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。

## Function Calling 的基本原理是什么？

<!-- canonical_key: agents.tools.function-calling-de-ji-ben-yuan-li-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 模型为什么知道有哪些工具可以调用？

<!-- canonical_key: agents.tools.mo-xing-wei-shen-me-zhi-dao-you-na-xie-gong-ju-ke-yi-diao-yong -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool Schema 如何传给模型？

<!-- canonical_key: agents.tools.tool-schema-ru-he-chuan-gei-mo-xing -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## JSON Schema 在 Function Calling 中起什么作用？

<!-- canonical_key: agents.tools.json-schema-zai-function-calling-zhong-qi-shen-me-zuo-yong -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果模型生成非法参数怎么办？

<!-- canonical_key: agents.tools.ru-guo-mo-xing-sheng-cheng-fei-fa-can-shu-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 你们是否使用 Pydantic 做参数校验？

<!-- canonical_key: agents.tools.ni-men-shi-fou-shi-yong-pydantic-zuo-can-shu-xiao-yan -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool 调用失败以后怎么处理？

<!-- canonical_key: agents.tools.tool-diao-yong-shi-bai-yi-hou-zen-me-chu-li -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool exception 会直接返回给模型吗？

<!-- canonical_key: agents.tools.tool-exception-hui-zhi-jie-fan-hui-gei-mo-xing-ma -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool Result 应该包含多少信息？

<!-- canonical_key: agents.tools.tool-result-ying-gai-bao-han-duo-shao-xin-xi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool Result 太长怎么办？

<!-- canonical_key: agents.tools.tool-result-tai-zhang-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool description 怎么写会更容易被模型正确调用？

<!-- canonical_key: agents.tools.tool-description-zen-me-xie-hui-geng-rong-yi-bei-mo-xing-zheng-que-diao-yong -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 工具数量过多会发生什么？

<!-- canonical_key: agents.tools.gong-ju-shu-liang-guo-duo-hui-fa-sheng-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果有 100 个工具，是否全部放入 Prompt？

<!-- canonical_key: agents.tools.ru-guo-you-100-ge-gong-ju-shi-fou-quan-bu-fang-ru-prompt -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何做工具动态加载？

<!-- canonical_key: agents.tools.ru-he-zuo-gong-ju-dong-tai-jia-zai -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何做工具检索？

<!-- canonical_key: agents.tools.ru-he-zuo-gong-ju-jian-suo -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 你简历里写“工具白名单”，具体如何实现？

<!-- canonical_key: agents.tools.ni-jian-li-li-xie-gong-ju-bai-ming-dan-ju-ti-ru-he-shi-xian -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何防止 Partner 调用没有权限的工具？

<!-- canonical_key: agents.tools.ru-he-fang-zhi-partner-diao-yong-mei-you-quan-xian-de-gong-ju -->

### 考察点

- 工具集合组装、拒绝优先与服务端执行授权

### 回答框架

- 每个 Turn 按授权上下文生成最终 Tool Surface，模型只获得当前任务允许的工具；实际分发也应校验同一集合，不能从全局 Registry 按任意名称执行。
- 三类策略可概括为：白名单筛选普通 Built-in 候选，forced 补入基础能力，suppressed 最终硬删除。forced 不受普通候选白名单约束，但不能覆盖 suppressed。
- 对 Partner 场景，可以抑制普通记忆工具、提供受限关系记忆工具；允许工具执行后仍要验证 Resource Scope，阻止它访问其它用户的数据。
- 通过构造伪造调用验证拒绝与无副作用；测试方法见 [越权拦截评测](./agent-multi-tenancy.md#越权调用拦截率-100怎么测)。

### 易追问

- forced 与 suppressed 同时含一个工具时谁优先？模型知道工具名是否等于有调用权？

### 关联笔记

- [DeepTutor Runtime 的每 Turn 工具策略](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md#partner-的每-turn-工具策略)

## 权限检查应该在 Prompt 层还是代码层？

<!-- canonical_key: agents.tools.quan-xian-jian-cha-ying-gai-zai-prompt-ceng-hai-shi-dai-ma-ceng -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 为什么不能只通过 Prompt 告诉模型“不要调用某工具”？

<!-- canonical_key: agents.tools.wei-shen-me-bu-neng-zhi-tong-guo-prompt-gao-su-mo-xing-bu-yao-diao-yong-mou-gong-ju -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果模型绕过 Prompt 直接构造 Tool Call 怎么办？

<!-- canonical_key: agents.tools.ru-guo-mo-xing-rao-guo-prompt-zhi-jie-gou-zao-tool-call-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool 权限系统如何设计？

<!-- canonical_key: agents.tools.tool-quan-xian-xi-tong-ru-he-she-ji -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool 是否应该支持版本管理？

<!-- canonical_key: agents.tools.tool-shi-fou-ying-gai-zhi-chi-ban-ben-guan-li -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 两个 Tool 名字或者语义非常接近时怎么办？

<!-- canonical_key: agents.tools.liang-ge-tool-ming-zi-huo-zhe-yu-yi-fei-chang-jie-jin-shi-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何评价一个 Tool Calling 系统的效果？

<!-- canonical_key: agents.tools.ru-he-ping-jia-yi-ge-tool-calling-xi-tong-de-xiao-guo -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Tool Calling 的准确率怎么测试？

<!-- canonical_key: agents.tools.tool-calling-de-zhun-que-lu-zen-me-ce-shi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何构建 Agent Tool Benchmark？

<!-- canonical_key: agents.tools.ru-he-gou-jian-agent-tool-benchmark -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 有副作用的 Tool 和只读 Tool 在执行策略上有什么区别？

<!-- canonical_key: agents.tools.you-fu-zuo-yong-de-tool-he-zhi-du-tool-zai-zhi-xing-ce-lue-shang-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 发送邮件、删除数据、支付这类 Tool 是否需要人工确认？

<!-- canonical_key: agents.tools.fa-song-you-jian-shan-chu-shu-ju-zhi-fu-zhe-lei-tool-shi-fou-xu-yao-ren-gong-que-ren -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何设计 Tool timeout、retry 和 circuit breaker？

<!-- canonical_key: agents.tools.ru-he-she-ji-tool-timeout-retry-he-circuit-breaker -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果 Tool 已执行成功但响应丢失，Agent 应该怎么处理？

<!-- canonical_key: agents.tools.ru-guo-tool-yi-zhi-xing-cheng-gong-dan-xiang-ying-diu-shi-agent-ying-gai-zen-me-chu-li -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-
