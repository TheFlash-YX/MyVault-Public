---
type: Note
domain: backend
title: DDD 领域驱动设计：战略与战术两层
aliases: [领域驱动设计, Domain-Driven Design]
description: DDD 核心主张、战略三件套（统一语言/限界上下文/防腐层）、战术构建块（实体/VO/聚合根/仓储）、四层架构三铁律与适用边界
tags: [ddd, 架构设计, 领域模型, 仓储模式, 依赖倒置]
status: learning
created: 2026-08-17
updated: 2026-08-17
---

# 概述

- DDD 是把设计重心放在业务领域模型上的方法论，分**战略**（划系统边界）和**战术**（代码级构建块）两层
- 核心主张：复杂软件的主要复杂度不在技术而在业务领域本身——让代码结构直接反映业务结构
- KindVideo 后端按四层 DDD 组织，痛点驱动（早期仓储返回 dict、ORM 泄漏到用例层），"痛过才用"是最好的面试答法

# 详细内容

## 三种设计风格对比

- 数据库驱动：先设计表，代码围着表转（transaction script）——业务规则散落，改一个规则找五个文件
- 框架驱动：代码围着 MVC/ORM 转——ORM 对象泄漏上层，换框架=重写
- **领域驱动**：先建领域模型，技术实现服务于模型——业务规则集中在 domain 层

## 战略设计三件套

- **统一语言**：业务专家、产品、开发、代码用同一套词汇；需求术语直接映射类名/方法名/表名（如「标为困难」→ `mark_difficult`）
- **限界上下文**：一个模型和它的语言只在边界内成立；上下文 ≠ 微服务，单体里可以有多个上下文，边界是将来拆微服务的天然切缝
- **防腐层**：与外部系统集成时先经过一层适配翻译，防止外部模型污染内部模型（如 httpx 客户端封装旧平台）

## 战术构建块

- **实体**：靠身份标识（id）区分，有生命周期、可变——两个属性完全相同仍是两个实体
- **值对象**：无身份、靠值判等、不可变——描述"是什么"不回答"是谁"（如 `Decision`、`ProbeMatchResult`）
- **聚合根**：聚合=一致性边界，根是唯一对外入口；**不变量在聚合内同事务维护**（如"缓存票数≡投票表行数"靠插票+票数+1 同事务保证）
- **仓储**：聚合存取抽象成"像集合"；接口在 domain、实现在 infrastructure（依赖倒置）；**不管事务**
- **领域服务**：不属于单一实体/VO 的纯函数业务规则，不碰 ORM 可脱离 DB 单测
- **用例/应用服务**：编排层，取聚合→调领域逻辑→存回→声明事务边界

## 四层架构与三条铁律

```
interfaces/rest   controllers / schemas / converters / DI
      ↓ 只依赖 application 的 DTO
application       use_cases(@transactional_write) / DTO / assemblers
      ↓ 依赖 domain 的接口
domain            entities / VO / repositories(I*接口) / services —— 不碰任何框架
      ↑ 实现依赖倒置
infrastructure    SQLAlchemy models / repo 实现 / external 客户端
```

- 铁律一：**Converters 不 import domain**——HTTP 层永远不知道领域模型长啥样
- 铁律二：**仓储不 commit**——事务边界在用例层声明（一个用例一个事务，可能跨多仓储；仓储 commit 会提交"半个用例"破坏原子性，Unit of Work 思想）
- 铁律三：**依赖方向指向 domain**——领域逻辑单测不需要数据库

## 常见误区（答错暴露没真用过）

- DDD = 分层架构？分层只是载体，**只分层没有领域模型 = 没有 DDD**
- 贫血模型也算 DDD？实体只有 getter/setter、逻辑全在 service = transaction script 穿了 DDD 的衣服；充血才是实体有行为
- Repository 就是 DAO？DAO 面向表、管连接；Repository 面向聚合、集合语义、不管事务
- DDD 必须事件溯源/CQRS？那些是可选增强

## 代价与适用边界

- 样板代码多（一个模块 20+ 文件）；简单 CRUD 是过度设计；学习曲线陡
- 判断标准：业务规则复杂、生命周期长、多人协作 → 值得；短平快工具/纯 CRUD → 不值得
- 读侧跨多仓储的展示组装放 application 层 assembler（DTO 组装），不为查询扭曲领域模型——CQRS 思想的轻量应用

# 相关链接

- [2026-08-17 学习日志](../../../daily/2026-08-17.md)
- [乐观锁：版本号 CAS 与并发取舍](../database/optimistic-locking.md)
- [ORM 跨语言对照：SQLAlchemy 与 Java 生态](../database/orm-python-vs-java.md)
- [标注探针机制：无 GT 场景的质量闭环](annotation-probe-quality-loop.md)
