---
type: interview
scope: topic
area: agents
topics:
  - multi-tenancy
  - workspace
  - authorization
created: 2026-08-27
updated: 2026-09-05
---

# Partner、Workspace 与多租户隔离

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。

## Partner 是什么概念？

<!-- canonical_key: agents.multi-tenancy.partner-shi-shen-me-gai-nian -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Workspace 是什么？

<!-- canonical_key: agents.multi-tenancy.workspace-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 为什么要为不同 Partner 创建独立 Workspace？

<!-- canonical_key: agents.multi-tenancy.wei-shen-me-yao-wei-bu-tong-partner-chuang-jian-du-li-workspace -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Soul 是什么？

<!-- canonical_key: agents.multi-tenancy.soul-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Soul 和 System Prompt 有什么区别？

<!-- canonical_key: agents.multi-tenancy.soul-he-system-prompt-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Skill 是什么？

<!-- canonical_key: agents.multi-tenancy.skill-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Notebook 是什么？

<!-- canonical_key: agents.multi-tenancy.notebook-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Partner 的课程知识库怎么挂载？

<!-- canonical_key: agents.multi-tenancy.partner-de-ke-cheng-zhi-shi-ku-zen-me-gua-zai -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Partner 配置是在创建时加载还是每轮动态加载？

<!-- canonical_key: agents.multi-tenancy.partner-pei-zhi-shi-zai-chuang-jian-shi-jia-zai-hai-shi-mei-lun-dong-tai-jia-zai -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## UnifiedContext 如何装配这些组件？

<!-- canonical_key: agents.multi-tenancy.unifiedcontext-ru-he-zhuang-pei-zhe-xie-zu-jian -->

### 考察点

- 上下文注入、资源作用域与执行机制解耦

### 回答框架

- 先从服务端已认证身份与授权配置确定资源 Scope，再装配 Persona、KB、Skill、Notebook、Memory 和工具策略，形成当前 Turn 的上下文。
- UnifiedContext 是执行信息的载体；Synthetic Scope 决定底层资源解析归属；Workspace 是资源的组织位置，三者职责不同。
- 保持统一 Runtime 执行流程，通过 Context 和 Scope 传入差异；具体类字段与加载顺序以对应实现为准。

### 易追问

- Scope 与上下文身份不一致时在哪里拒绝？共享 Runtime 是否意味着共享可变会话状态？

### 关联笔记

- [DeepTutor Runtime 与 Partner 隔离](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)

## 如何保证 Partner A 读不到 Partner B 的数据？

<!-- canonical_key: agents.multi-tenancy.ru-he-bao-zheng-partner-a-du-bu-dao-partner-b-de-shu-ju -->

### 考察点

- 能力授权与资源归属两层边界

### 回答框架

- Runtime 按当前身份组装可调用工具，Resource Resolver 再将允许的操作限定到当前 Workspace；工具可调用不等于目标数据可访问。
- 资源路径、检索过滤条件和 Memory owner 应来自服务端授权上下文，不能相信模型传入的 partner_id / user_id。
- 关系记忆按 Partner × User 隔离，跨 Partner 与同 Partner 跨用户都要验证；获授权的 Shared L3 读取是明确的共享例外。

### 易追问

- 工具接受任意路径会不会绕过 Scope？如何检查 Tool Result 中已发生但最终回答隐藏的泄漏？

### 关联笔记

- [DeepTutor Runtime 与 Partner 隔离](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)

## “跨作用域泄漏 0%”怎么测？

<!-- canonical_key: agents.multi-tenancy.kua-zuo-yong-yu-xie-lou-0-zen-me-ce -->

### 考察点

- 隔离测试矩阵、观测层次与指标边界

### 回答框架

- 预先定义 Partner × User × 资源类型 × 操作的测试矩阵，包含跨 KB、Notebook、Skill 和关系记忆的非法读取，以及同 Scope 的合法对照。
- 每个私有 Scope 放唯一 Canary Token，覆盖直接查询、搜索与 Prompt Injection；在 Tool Result、Runtime Context 和 Final Response 任一层发现非授权数据都判失败。
- 泄漏率 = 发生非授权跨 Scope 读取的测试数 / 纳入统计的隔离测试总数；说明正反用例是否纳入分母，分母为 0 时不能报 0%。
- 0% 只描述该测试集未观察到泄漏，不能推出绝对安全；真实样本与报告见 [项目指标口径](../resume/resume-metrics.md#跨作用域泄漏-0-测了多少场景)。

### 易追问

- 模型改写敏感内容后 Canary 检测会漏报吗？一次执行多次读取按用例还是调用计数？

### 关联笔记

- [DeepTutor Runtime 与 Partner 隔离](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)

## “越权调用拦截率 100%”怎么测？

<!-- canonical_key: agents.multi-tenancy.yue-quan-diao-yong-lan-jie-lu-100-zen-me-ce -->

### 考察点

- 授权执行边界与无副作用断言

### 回答框架

- 在进入 Loop 前检查最终工具集合，再主动构造非法 Tool Call 进入分发路径，验证被拒绝且工具实现没有执行、目标数据没有变化。
- 覆盖白名单外普通 Built-in、suppressed 工具、forced 与 suppressed 冲突，以及从普通 Chat 请求 Partner 专属工具；同时验证合法工具正常执行。
- 拦截率 = 被成功阻止的越权 Tool Call 数 / 构造的越权 Tool Call 总数；不能把模型根本没尝试调用算成一次成功拦截，分母必须大于 0。
- 100% 仅限有执行记录的测试集；数据泄漏率另行统计，项目证据统一见 [指标口径](../resume/resume-metrics.md#跨作用域泄漏-0-测了多少场景)。

### 易追问

- 仅从模型 Schema 隐藏工具是否足够？拦截后如何检查数据库或 Memory 没有副作用？

### 关联笔记

- [DeepTutor Runtime 与 Partner 隔离](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)

## 隔离是在数据库层、Service 层还是 Agent Runtime 层？

<!-- canonical_key: agents.multi-tenancy.ge-li-shi-zai-shu-ju-ku-ceng-service-ceng-hai-shi-agent-runtime-ceng -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果模型明确要求读取另一个 Partner 的 Memory，会发生什么？

<!-- canonical_key: agents.multi-tenancy.ru-guo-mo-xing-ming-que-yao-qiu-du-qu-ling-yi-ge-partner-de-memory-hui-fa-sheng-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 为什么“学生记忆只读、关系记忆独立可写”？

<!-- canonical_key: agents.multi-tenancy.wei-shen-me-xue-sheng-ji-yi-zhi-du-guan-xi-ji-yi-du-li-ke-xie -->

### 考察点

- 全局画像与局部关系偏好的所有权

### 回答框架

- 共享用户记忆会被多个场景消费，限制 Partner 只读可以避免局部互动修改全局画像、影响其它 Partner。
- 关系记忆保存某个 Partner 与某个用户之间的偏好，写入边界是 Partner × User，例如算法教练的提示习惯不必影响英语搭子。
- 用专属写工具限定关系记忆，并在资源访问层验证双维 owner；读取 Shared L3 仍须绑定当前获授权的真实用户。

### 易追问

- 共享记忆与关系偏好冲突时如何表达？同一 Partner 面对不同用户如何隔离？

### 关联笔记

- [DeepTutor 学习记忆与 Split Memory](../../notes/agents/projects/deeptutor/learning-memory-system.md)

## 关系记忆是什么？

<!-- canonical_key: agents.multi-tenancy.guan-xi-ji-yi-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Partner 删除以后，其关系记忆怎么处理？

<!-- canonical_key: agents.multi-tenancy.partner-shan-chu-yi-hou-qi-guan-xi-ji-yi-zen-me-chu-li -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 多个 Partner 能否共享课程知识库？

<!-- canonical_key: agents.multi-tenancy.duo-ge-partner-neng-fou-gong-xiang-ke-cheng-zhi-shi-ku -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果共享知识库，权限怎么控制？

<!-- canonical_key: agents.multi-tenancy.ru-guo-gong-xiang-zhi-shi-ku-quan-xian-zen-me-kong-zhi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 这个系统本质上是否是一个多租户系统？

<!-- canonical_key: agents.multi-tenancy.zhe-ge-xi-tong-ben-zhi-shang-shi-fou-shi-yi-ge-duo-zu-hu-xi-tong -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 多租户系统有哪些常见隔离方案？

<!-- canonical_key: agents.multi-tenancy.duo-zu-hu-xi-tong-you-na-xie-chang-jian-ge-li-fang-an -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果未来服务十万学生，这套架构有哪些瓶颈？

<!-- canonical_key: agents.multi-tenancy.ru-guo-wei-lai-fu-wu-shi-wan-xue-sheng-zhe-tao-jia-gou-you-na-xie-ping-jing -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Workspace 的配置如何版本化？

<!-- canonical_key: agents.multi-tenancy.workspace-de-pei-zhi-ru-he-ban-ben-hua -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Partner 升级 Skill 后，历史会话怎么兼容？

<!-- canonical_key: agents.multi-tenancy.partner-sheng-ji-skill-hou-li-shi-hui-hua-zen-me-jian-rong -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-
