---
type: Note
domain: backend
title: Spring @Value 配置读取与默认值
aliases: []
description: 在 Spring 管理的组件中使用属性占位符读取配置并进行类型转换
tags: [spring, configuration, value]
status: learning
created: 2026-09-08
updated: 2026-09-08
---

# 概述

- 在 Spring 管理的组件中使用属性占位符读取配置并进行类型转换
- 来源：2026-09-07 龙旗科技一面后的 2026-09-08 互动复习；记录已学内容，不表示已完成代码实践或熟练掌握。

# 详细内容

## 配置与注入

```yaml
app:
  timeout: 60
server:
  port: 8080
```

```java
@Component
public class AppService {
    @Value("${server.port}")
    private int port;

    @Value("${app.timeout:30}")
    private int timeout;
}
```

- 示例要求组件被 Spring 扫描注册并完成注入；手动 new 不会自动获得注入结果。
- server.port 的值转为 int，得到 8080。
- app.timeout 已配置 60，使用 60；缺失才使用默认值 30，不是默认值覆盖配置。
- @Value 读取 Spring 配置环境中的属性，不限定只能来自 YAML 文件。
- 一组配置可以考虑 @ConfigurationProperties；本轮仅认识名称，绑定与注册方式待学。
- Java 注解定义和 Spring 赋予注解的处理行为是两个层面。

# 相关链接

- [Spring Boot 面试题](../../../interview/backend/spring-boot.md)
- [Java 注解基础](../../java/annotations-basics.md)
- [Spring @Value 官方文档](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/value-annotations.html)
