---
type: Pitfall
domain: frontend
title: 前端把 null/空值发给后端 → 原始英文 422 报错，交互体验差
description: BEV 标注四字段空值点保存，toast 弹出 Pydantic 原始英文校验消息；根因是前端未做本地必填/边界预校验，把 null 发给了后端
tags: [pydantic, validation, 422, frontend, antd, alert]
created: 2026-08-24
updated: 2026-08-24
---

# 现象

- BEV 标注工作台：座位数/行数/列数/组数四个 `InputNumber` 有空值时点「保存」，toast 弹出原始英文：
  `body.seat_count: Input should be a valid integer`（Pydantic 422 报错原文拼进统一错误提示）
- 标注员看不懂，误以为系统坏了；一闪而过的全局 toast 也无法承载错误详情

# 排查过程

- 前端 `InputNumber` 清空后值为 `null` → 四个字段组装成 payload 时保持 `null` → `PUT` 原样发给后端
- 后端 schema 四字段为非空 int → Pydantic 校验失败 → `422`，`detail` 是英文逐字段消息
- 前端 `extractErrorMessage` 把 detail 拼接进 `res.error`，全局 `message.error` 原样展示

# 根因

- 前端**没有本地预校验**就把空值/非法值发给后端，把校验责任完全甩给后端英文消息
- Pydantic 校验消息面向开发者，不是给标注员看的交互文案

# 解决方案

- 前端保存前做**本地预校验**（对齐后端 FR-023~025 规则：四字段必填、`seat_count ≥ 0`、行/列/组 ≥ 1、`seat_count ≤ 行×列`、`组数 ≤ 列数`），返回中文错误提示，非法时直接拦截不发请求
- 保存请求加 `silentError: true` 抑制全局 toast，改为表单区上方**常驻 `Alert`** 展示错误，可关闭、修改字段即清除（比一闪而过的 toast 更适合表单校验）
- 后端领域层继续保留中文语义化校验兜底（`ValidationDomainException`），双保险
- 配套 2 个前端单测：空字段 → 「请填写全部四项」且不发出请求；座位数超行×列 → 中文提示且阻止保存

# 相关链接

- （frontend 主题笔记待补充，暂无已存在笔记可链）
