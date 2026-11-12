---
type: interview
scope: topic
area: backend
topics:
  - spring-boot
  - spring-transactions
  - orm
created: 2026-08-27
updated: 2026-09-08
---

# Spring Boot

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。

## Spring IoC 是什么？

<!-- canonical_key: backend.spring.spring-ioc-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Dependency Injection 是什么？

<!-- canonical_key: backend.spring.dependency-injection-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Spring Bean 生命周期是什么？

<!-- canonical_key: backend.spring.spring-bean-sheng-ming-zhou-qi-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Spring AOP 是什么？

<!-- canonical_key: backend.spring.spring-aop-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## @Transactional 原理是什么？

<!-- canonical_key: backend.spring.transactional-yuan-li-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 为什么同类方法内部调用可能导致 @Transactional 失效？

<!-- canonical_key: backend.spring.wei-shen-me-tong-lei-fang-fa-nei-bu-diao-yong-ke-neng-dao-zhi-transactional-shi-xiao -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Spring 事务什么时候会回滚？

<!-- canonical_key: backend.spring.spring-shi-wu-shen-me-shi-hou-hui-hui-gun -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Checked Exception 会不会默认回滚？

<!-- canonical_key: backend.spring.checked-exception-hui-bu-hui-mo-ren-hui-gun -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## Spring Boot 自动配置原理是什么？

<!-- canonical_key: backend.spring.spring-boot-zi-dong-pei-zhi-yuan-li-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## MyBatis 和 JPA 有什么区别？

<!-- canonical_key: backend.spring.mybatis-he-jpa-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## MyBatis 的 #{} 和 ${} 有什么区别？

<!-- canonical_key: backend.spring.mybatis-de-he-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何防止 SQL Injection？

<!-- canonical_key: backend.spring.ru-he-fang-zhi-sql-injection -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何读取配置文件中的值并设置默认值？
<!-- canonical_key: backend.spring.value-property-default -->

### 考察点

- 概念、执行过程与边界。

### 回答框架

- Spring 管理的组件里可用 @Value("${server.port}") 注入端口。
- @Value("${app.timeout:30}") 在属性缺失时用30；已有60则注入60，并转换为字段类型。
- 一组配置可考虑 @ConfigurationProperties，本轮未展开其绑定实现。

### 易追问

- 自己 new 的对象会自动注入吗？

### 关联笔记

- [Spring 配置读取](../../notes/backend/spring/value-configuration.md)
