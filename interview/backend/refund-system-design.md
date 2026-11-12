---
type: interview
scope: topic
area: backend
topics:
  - refund-system
  - payment
  - idempotency
  - state-machine
  - reconciliation
created: 2026-09-03
updated: 2026-09-03
---

# 退款系统场景设计

- 场景题主线：业务规则 -> 状态机 -> 幂等 -> 超时查单 -> 对账 -> 工程取舍。
- 关联知识与题集：
- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)
- [事务、锁与幂等](./transactions-locking-and-idempotency.md)
- [退款系统场景面试复盘原文](../meta/source/refund-system-scenario-interview-review-2026-09-03.md)

## 如何设计自动退款的判断逻辑？

<!-- canonical_key: backend.refund-system.design-risk-rules -->

### 考察点

- 能否先区分硬性业务规则、风险规则、风险评分和人工兜底，而不是直接回答“打分 + 阈值”。
- 是否能围绕退款资格、风险识别和资金正确性建立层次化判断。

### 回答框架

- 我会分三层处理。第一层是退款资格校验，例如订单是否已支付、是否在售后期限内、是否已经退款、剩余可退金额是否足够，这些不满足就直接拒绝。
- 第二层是高风险硬规则，例如短时间内退款次数过多、退款率异常、历史恶意退款、高金额退款或异常物流，命中后直接转人工。
- 第三层才做风险评分，对剩余灰度用户综合物流轨迹、历史客诉、账号行为和退款频率打分；低风险自动退款，中高风险人工审核。
- 这样做的好处是，明显不合法的订单不会被评分模型误放，复杂风控也可以独立迭代。

### 易追问

- 哪些条件是硬规则，哪些条件适合进入评分？
- 如果业务要求两周上线，复杂风控来不及怎么办？
- 自动退款和人工审核的边界怎么定？

### 关联笔记

- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)

## 退款系统的状态机如何设计？

<!-- canonical_key: backend.refund-system.state-machine -->

### 考察点

- 是否理解流程和状态机的区别。
- 是否能定义终态、非法迁移和人工审核兜底。

### 回答框架

- 状态机关注当前状态、触发事件和合法迁移，不是简单把“打分前、打分后、人工处理”列成流程。
- 一笔退款可以从 `CREATED` 进入 `CHECKING`，再进入 `AUTO_APPROVED` 或 `MANUAL_REVIEW`；审核通过后进入 `REFUND_PROCESSING`，最终落到 `REFUND_SUCCESS`、`REFUND_FAILED` 或 `REFUND_UNKNOWN`。
- 严格禁止的迁移包括成功后重新处理中、人工未审核通过直接退款、未调用支付渠道直接标成功、已全额退款订单再次全额退款。
- 状态机的核心价值是把资金系统的不变量落到代码约束里，防止重复退款、错退款和状态混乱。

### 易追问

- 为什么风控服务超时可以直接转人工？
- `UNKNOWN` 是不是终态？
- 状态迁移校验应该放在哪里？

### 关联笔记

- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)

## 前端显示退款已受理但后端没有记录，怎么排查？

<!-- canonical_key: backend.refund-system.frontend-accepted-no-backend-record -->

### 考察点

- 能否利用题目条件缩小排查范围。
- 是否会用 `trace_id`、`request_id`、`order_id` 顺链路找断点。

### 回答框架

- 我会先确认前端“退款已受理”的提示来自哪里，是前端本地状态、网关返回，还是退款服务真实返回。
- 然后用 `order_id`、`request_id` 或 `trace_id` 查整条链路：前端是否发请求、网关是否收到、退款服务是否收到、是否创建退款单、事务是否提交。
- 如果网关有日志但退款服务没有日志，重点查路由、RPC 或消息投递；如果退款服务收到请求但数据库没有记录，重点查事务回滚、异常吞掉或异步消息发送失败。
- 题目已经说明支付退款 API 没有被调用，所以优先查支付前链路，不要一开始就查支付渠道后置结果。

### 易追问

- 没有 `trace_id` 时怎么补救？
- 日志没有报错为什么仍可能事务回滚？
- 前端误展示和后端成功返回如何区分？

### 关联笔记

- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)

## 支付退款接口超时但实际成功，用户重复点击怎么办？

<!-- canonical_key: backend.refund-system.payment-timeout-idempotency -->

### 考察点

- 是否知道超时不等于失败。
- 是否能设计业务幂等键、唯一约束和幂等返回。

### 回答框架

- 首先退款请求必须具备业务幂等性，每次退款创建唯一的 `refund_id`、`refund_order_id` 或 `merchant_refund_no`，并用数据库唯一约束保证同一业务请求只能创建一次。
- 调用支付渠道时携带同一个商户退款单号；用户重复点击时，如果本地已有退款单，就不再发起第二次支付退款，而是返回已有退款单状态。
- 支付接口超时不能直接标记失败，因为渠道可能已经执行成功；此时应把本地状态保持为 `UNKNOWN` 或 `PROCESSING`。
- 后台用 `refund_id` 或 `merchant_refund_no` 主动查单，最终把状态修正为 `REFUND_SUCCESS` 或 `REFUND_FAILED`。

### 易追问

- 为什么不能用 `user_id + order_id` 做唯一幂等键？
- 重复请求应该丢弃还是返回历史结果？
- 本地成功但支付调用超时时怎么恢复？

### 关联笔记

- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)
- [事务、锁与幂等](./transactions-locking-and-idempotency.md)

## 月底对账发现退款金额对不上，第一步怎么查？

<!-- canonical_key: backend.refund-system.reconciliation-difference-first-step -->

### 考察点

- 是否先统一统计口径，而不是马上逐笔翻订单。
- 是否能按差异类型组织排查。

### 回答框架

- 我第一步会先确认双方统计口径是否一致，包括时间范围、退款成功状态定义、统计时间字段、币种、手续费、部分退款和退款渠道。
- 口径一致后，再用 `merchant_refund_no` 或 `refund_id` 逐笔匹配，把双方都存在且金额一致的记录排除。
- 剩余差异单再分类：本地成功但渠道无记录、渠道成功但本地无记录、金额不一致、重复退款、状态不一致。
- 最后按类型处理：能查单修复的自动修复，金额异常或无法解释的进入人工和财务复核。

### 易追问

- 创建时间和完成时间口径不同会发生什么？
- 差异单如何自动修复？
- 对账为什么是资金系统的兜底能力？

### 关联笔记

- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)

## 排期紧只能保留三项能力，风控、状态机、幂等和对账砍哪一个？

<!-- canonical_key: backend.refund-system.scope-cut-correctness-vs-risk-control -->

### 考察点

- 是否能区分资金正确性能力和业务损失控制能力。
- 是否能给出低成本兜底，而不是机械砍功能。

### 回答框架

- 如果只能砍一个，我会砍复杂风控评分体系，但不会完全取消风控。
- 状态机、幂等和对账直接关系到资金正确性，不能牺牲；它们保证不能多退、错退，异常后还能查回和对平。
- 风控可以先降级成高收益硬规则，例如高频退款、高金额退款、历史恶意账号、异常物流直接转人工。
- 复杂评分、设备关联和模型策略后续迭代，这样能按期上线，又能用人工客服兜住明显高风险订单。

### 易追问

- 为什么不是砍对账？
- 简单风控规则如何避免误伤正常用户？
- 两周上线如何拆一期、二期？

### 关联笔记

- [退款系统正确性：状态机、幂等、查单与对账](../../notes/backend/architecture/refund-system-correctness.md)
