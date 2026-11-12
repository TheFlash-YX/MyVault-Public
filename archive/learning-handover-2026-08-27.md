---
type: Archive
title: 学习计划与进度总览（换机交接版）
description: 2026-08-27 换电脑交接文档：四条学习线的当前进度、明日计划、未来路线与新机恢复步骤
created: 2026-08-27
updated: 2026-09-05
archived: 2026-09-05
---

# 学习计划与进度总览（换机交接版）

> 历史快照：以下进度、环境、命令与“今晚／下一讲”等表述仅对应 2026-08-27，不代表当前待办；当前流程见 [学习入口](../learning-plan.md)。原稿“每题带参考答案”不应作为完成度依据，正式题集仍需逐步补全回答与证据。

> 生成时间：2026-08-27。旧电脑将于 2026-08-28 恢复出厂设置，本文档是唯一的交接依据。
> 本仓库已推送到 GitHub：**https://github.com/TheFlash-YX/MyVault**（交接时无未推送提交）。

## 0. 换机必读（先干这件事！）

**今晚必须执行**（否则本文档丢失）：

```bash
cd ~/MyVault
git add learning-plan.md index.md
git commit -m "docs: 换机交接——学习计划与进度总览 2026-08-27"
git push origin main
```

**新机恢复三步**：

1. `git clone https://github.com/TheFlash-YX/MyVault.git`，用 Obsidian 打开该目录；
2. 装 Claude Code，在任意目录开新会话，对它说：「读 ~/MyVault/learning-plan.md，我们继续」；
3. 本机 `~/.claude` 下的 AI 记忆会被抹掉，**无所谓**——权威计划全部在 MyVault，本文档就是重新播种的入口。

## 1. 全景：四条学习线

| 线 | 状态 | 权威进度来源 |
|---|---|---|
| **A. Java 基础讲系列** | 第 1~10 讲完成，第 11 讲待开 | 最新 daily 的「明日计划」 |
| **B. 算法 LeetCode** | 链表主线 12/17，剩 5 题 | 最新 daily 的「明日计划」 |
| **C. 2027 秋招题库** | 725 题已全部迁入 `interview/` | `interview/meta/autumn-2027-migration-map.md` |
| **D. 项目研读** | DeepTutor 三篇完成 | `notes/agents/projects/deeptutor/` |

## 2. A 线：Java 基础讲系列

**已完成的 10 讲**（笔记在 `notes/java/`）：

| 讲 | 主题 | 笔记 |
|---|---|---|
| 1 | JDK / JRE / JVM | [jdk-jre-jvm](../notes/java/jdk-jre-jvm.md) |
| 2 | 编译型 vs 解释型 | [java-compile-interpret](../notes/java/java-compile-interpret.md) |
| 3 | 8 种基本数据类型 | [primitive-data-types](../notes/java/primitive-data-types.md) |
| 4 | 装箱拆箱与 Integer 缓存 | [boxing-unboxing-integer-cache](../notes/java/boxing-unboxing-integer-cache.md) |
| 5 | String 不可变性与常量池 | [string-immutability-pool](../notes/java/string-immutability-pool.md) |
| 6 | String vs StringBuilder vs StringBuffer | [string-vs-stringbuilder-stringbuffer](../notes/java/string-vs-stringbuilder-stringbuffer.md) |
| 7 | 堆与栈 | [heap-vs-stack](../notes/java/heap-vs-stack.md) |
| 8 | 类、对象、构造方法 | [class-object-constructor](../notes/java/class-object-constructor.md) |
| 9 | 封装、继承、多态 | [oop-three-features](../notes/java/oop-three-features.md) |
| 9 续 | 接口 vs 抽象类 | [interface-vs-abstract-class](../notes/java/interface-vs-abstract-class.md) |
| 10 | equals / hashCode / ==（含 Integer 缓存、final 补讲） | [equals-hashcode-double-equal](../notes/java/equals-hashcode-double-equal.md) |

**下一讲（已写死在 2026-08-26 日志）**：第 11 讲 **重载 vs 重写**（编译期静态分派 vs 运行期动态分派）。

**未来路线候选**（按面试高频排序，供之后逐讲展开）：

1. 重载 vs 重写（下一讲，已定）
2. 集合框架：ArrayList 扩容、HashMap 原理、ConcurrentHashMap（对应 [interview/java/collections-and-concurrency.md](../interview/java/collections-and-concurrency.md)）
3. 异常体系：checked vs unchecked、try-with-resources
4. 泛型与类型擦除
5. 并发基础：synchronized、volatile、线程池、JUC（JavaGuide 仓库就在手边：`~/JavaGuide`，注意它也会随机抹掉，它是公开仓库 https://github.com/Snailclimb/JavaGuide，新机重新 clone 即可）
6. JVM：运行时数据区、GC、类加载
7. IO / 反射 / 动态代理

## 3. B 线：算法 LeetCode

**链表主线 12/17**，四大套路已成型（笔记在 `notes/algorithms/`）：

- 反转家族：206 / 92 / 25 / 143 / 234 → [linked-list-reversal](../notes/algorithms/linked-list-reversal.md)
- 合并家族：21 / 23 → [linked-list-merge](../notes/algorithms/linked-list-merge.md)
- 快慢家族：141 / 142 / 19 → [linked-list-fast-slow](../notes/algorithms/linked-list-fast-slow.md)
- 走两程：160

**明日计划（2026-08-27 日志写死的，今晚/明天第一件事）**：

1. 答 Q10：**82 删除排序链表重复元素 II 三连问**（整段删除判断标准 / 探边界+下刀两指针配合 / 没重复才推进前驱的原因）
2. 链表收尾：**83 → 148 排序链表 → 2 两数相加 → 24 两两交换**（24 = 25 的 k=2 特例）

**其他已完成主题笔记**：DP（打家劫舍、最大子数组）、快排、拓扑排序、拒绝采样、摩尔投票、栈表达式求值、双指针。

**遗留待办**：修 143 的 `cur = head` 初始化 bug 并跑通 5 个 case（2026-08-26 日志记录，截至交接未确认完成）。

**未来路线候选**：链表收尾 → 二叉树（遍历/层序/递归三件套）→ 二分查找家族 → 回溯 → 堆与 TopK → 图 BFS/DFS。

## 4. C 线：2027 秋招题库（interview/）

- 725 题（Q1~Q725）已从原始题库迁移完毕：722 moved、3 merged、0 丢失，账本是 [interview/meta/autumn-2027-migration-map.md](../interview/meta/autumn-2027-migration-map.md)。
- 按领域分文件，每题带参考答案：
  - **agents/**（15 个题集）：agent-runtime、human-in-the-loop、streaming-sse、function-calling、mcp、memory、rag、rag-frameworks、evaluation、multi-tenancy、安全、架构权衡、排障、AI 编码工程、系统设计
  - **backend/**：mysql、redis、kafka、spring-boot、docker、事务锁幂等
  - **python/**：asyncio 并发、fastapi
  - **resume/**：behavioral、motivation、kindpartner、kindvideo、resume-metrics
  - **java/**：collections-and-concurrency（待扩充）
- 未来用法：刷题 + 对答案 + 把薄弱点沉淀回 `notes/`（走 CLAUDE.md 的记题流/笔记流）。

## 5. D 线：项目研读与散点笔记

- **DeepTutor** 三篇：[Agent Runtime 与 Turn 生命周期](../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)、[Chat Agent Loop 与工具协议](../notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)、[知识库与 RAG Provider](../notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md)
- **backend 散点**：DDD、乐观锁、ORM Python vs Java、SQLite/MySQL 方言、缓存与异步任务、标注探针质量环
- **python 散点**：asyncio 协程、可变默认参数、除法取整、map 一行流
- **踩坑**（8 篇，`pitfalls/`）：代理/Docker、Alembic、422 空字段等——换机后代理相关的坑（httpx-socks、orbstack-pull）大概率还会再遇到，值得先看一眼。

## 6. 工作方式约定（写给新机上的 Claude）

- 本仓库的完整规则在 [CLAUDE.md](../CLAUDE.md)（OKF 五种流：笔记/日志/踩坑/书签/记题），新会话先读它。
- **接续规则**：读 `daily/` 最新一篇的「明日计划」，那里写死了下一讲的编号和主题；不要靠猜。
- **讲 ≠ 笔记 1:1**：一讲可覆盖 1~2 个主题，不要用 `notes/java/` 文件数推讲数。
- 不自动 `git commit`；文件间链接一律标准 Markdown 相对路径，禁止 Obsidian wiki 语法。

## 7. 换机后立即做（第一天 checklist）

1. [ ] 今晚 push 本文档（见第 0 节命令）
2. [ ] 新机 clone MyVault，Obsidian 打开
3. [ ] 重新 clone JavaGuide：`git clone https://github.com/Snailclimb/JavaGuide.git`
4. [ ] 装 Claude Code，新会话说「读 ~/MyVault/learning-plan.md」
5. [ ] 继续执行：算法 Q10（82 三连问）→ 链表收尾 5 题 → Java 第 11 讲
