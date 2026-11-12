---
type: Index
title: 学习知识库总览
description: OKF 格式的个人学习知识库入口
---

# 学习知识库总览

> [学习入口与沉淀流程](learning-plan.md)：从 ChatGPT 学习材料接续知识库整理；最新进度见下方日志，面试状态见 Dashboard。

这里按 OKF（Open Knowledge Format）思路组织个人学习笔记，覆盖 Java、后端、前端、Agent、算法、Python 六个领域。

## 最近日志

- [2026-10-12](daily/2026-10-12.md) — 62 二维/一维、139 标准版通过；DP 主题已覆盖，下一题 3，主线剩余 6 题
- [2026-10-11](daily/2026-10-11.md) — 122 持股双状态、221 最大正方形：数组版及空间优化版通过
- [2026-10-10](daily/2026-10-10.md) — 72 一维版、1143/322/70/64 通过；下一题 122，剩余未覆盖主线 10 题
- [2026-10-09](daily/2026-10-09.md) — 240/48、121、300 与 72 二维版通过；先 DP 数组再优化
- [2026-10-08](daily/2026-10-08.md) — 56/179/200/695 手写通过，54 已报告通过；下一题 240
- [2026-10-07](daily/2026-10-07.md) — 215 快速选择通过：随机双向划分、重复值和单侧区间收缩
- [2026-10-06](daily/2026-10-06.md) — 43/151/14 通过，字符串主题完成本轮复习；下一道未复习题 215
- [2026-10-05](daily/2026-10-05.md) — 补记 39/40 组合总和、5 回文中心扩展与华为笔试两题；主线下一题 415
- [2026-09-27](daily/2026-09-27.md) — 回溯：22/78 与扩展 90；31 下一个排列；补记 46/47/93；React、浏览器渲染及 KindVideo 视频组件
- [2026-09-24](daily/2026-09-24.md) — 560 前缀和频次、169 摩尔投票与 229 双候选扩展

## 主题笔记

### Java

- [try-catch-finally：返回与异常传播](notes/java/try-catch-finally-return.md)
- [Java 线程、任务与线程池](notes/java/thread-tasks-and-pools.md)
- [Java 自定义注解](notes/java/annotations-basics.md)

- [JDK、JRE、JVM 的关系与 Java 跨平台原理](notes/java/jdk-jre-jvm.md)
- [Java 是编译型还是解释型语言（混合模式与 JIT）](notes/java/java-compile-interpret.md)
- [Java 8 种基本数据类型与三大必考坑](notes/java/primitive-data-types.md)
- [自动装箱拆箱与 Integer 缓存陷阱](notes/java/boxing-unboxing-integer-cache.md)
- [String 不可变性与字符串常量池](notes/java/string-immutability-pool.md)
- [String vs StringBuilder vs StringBuffer](notes/java/string-vs-stringbuilder-stringbuffer.md)
- [JVM 堆（Heap）与栈（Stack）](notes/java/heap-vs-stack.md)
- [类、对象、构造方法（面向对象开篇）](notes/java/class-object-constructor.md)
- [封装、继承、多态（OOP 三大特性）](notes/java/oop-three-features.md)
- [接口 vs 抽象类（面向接口编程基础）](notes/java/interface-vs-abstract-class.md)
- [equals、hashCode 与 ==（Object 核心三件套）](notes/java/equals-hashcode-double-equal.md)

### 后端

**数据库（database）**

- [MySQL 联合索引、回表与 EXPLAIN](notes/backend/database/mysql-index-and-explain.md)

- [SQLite 与 MySQL 方言差异清单](notes/backend/database/sqlite-mysql-dialect-differences.md)
- [ORM 跨语言对照：SQLAlchemy 与 Java 生态](notes/backend/database/orm-python-vs-java.md)
- [乐观锁：版本号 CAS 与并发取舍](notes/backend/database/optimistic-locking.md)
- [SQLite 存储类型与外键约束：REAL 时间戳与 PRAGMA foreign_keys](notes/backend/database/sqlite-types-and-foreign-keys.md)

**Spring（spring）**

- [Spring @Value 配置读取与默认值](notes/backend/spring/value-configuration.md)

**架构设计（architecture）**

- [DDD 领域驱动设计：战略与战术两层](notes/backend/architecture/ddd-domain-driven-design.md)
- [退款系统正确性：状态机、幂等、查单与对账](notes/backend/architecture/refund-system-correctness.md)
- [文件存储三原则：NAS 存文件、索引在库、直链播放](notes/backend/architecture/file-storage-nas-direct-url.md)
- [不引入 Redis/MQ 的架构取舍：DB 任务表与进程内缓存](notes/backend/architecture/cache-and-async-task-alternatives.md)
- [标注探针机制：无 GT 场景的质量闭环](notes/backend/architecture/annotation-probe-quality-loop.md)

**Redis / MQ**

（待补充）

### 前端

- [React 核心概念与渲染流程](notes/frontend/react-fundamentals-and-rendering.md)
- [浏览器输入 URL 后的加载、渲染与水合](notes/frontend/browser-navigation-and-dom.md)
- [KindVideo 视频播放组件设计](notes/frontend/kindvideo-video-playback-components.md)

### Agent

**AI 编程与工程实践**

- [AI 编程时代的软件基本功：设计概念、反馈循环与深模块](notes/agents/software-fundamentals-for-ai-coding.md)

**项目源码研读（projects）**

- [DeepTutor Agent Runtime：Turn 生命周期与事件流](notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)
- [DeepTutor Chat Agent Loop：模型轮次、工具协议与暂停恢复](notes/agents/projects/deeptutor/chat-agent-loop-and-tool-protocol.md)
- [DeepTutor 三级可控学习记忆：L1 Trace、L2 Facts 与 L3 Long-term Memory](notes/agents/projects/deeptutor/learning-memory-system.md)
- [DeepTutor：知识库与 RAG Provider 架构](notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md)

### 算法

- [序列最优状态：股票交易与最长递增子序列](notes/algorithms/dp-sequence-and-tail-state.md)
- [双序列 DP：编辑距离、公共子序列与滚动数组](notes/algorithms/dp-two-sequences.md)
- [DP 转移与初始化：代价、计数、网格和可达性](notes/algorithms/dp-cost-count-and-grid.md)
- [排序应用：合并区间与拼接最大数](notes/algorithms/sorting-intervals-and-comparators.md)
- [网格遍历与变换：岛屿、螺旋、矩阵搜索和旋转](notes/algorithms/grid-traversal-and-boundaries.md)
- [数字字符串模拟：竖式加乘、版本比较与 atoi](notes/algorithms/string-number-simulation.md)
- [字符串扫描：单词反转与最长公共前缀](notes/algorithms/string-words-and-prefix.md)
- [排列题：全排列、去重与下一个排列](notes/algorithms/permutations-and-next-permutation.md)
- [子集与组合总和：起点控制、同层去重与剪枝](notes/algorithms/subsets-and-deduplication.md)
- [最长回文子串：奇偶中心与扩展边界](notes/algorithms/palindrome-center-expansion.md)
- [笔试算法：稀疏注意力模拟与连续分段 DP](notes/algorithms/sparse-attention-and-stage-partition.md)
- [受约束的回溯：IP 地址与括号生成](notes/algorithms/backtracking-constrained-construction.md)
- [前缀和与频次表：统计和为 K 的连续子数组](notes/algorithms/prefix-sum-frequency.md)
- [LRU 缓存：哈希定位与双向链表维护顺序](notes/algorithms/lru-cache.md)
- [哈希与原地归位：两数之和、缺失正数、连续序列](notes/algorithms/hash-array-techniques.md)
- [二分查找：闭区间、答案边界与 lower_bound](notes/algorithms/binary-search-boundaries.md)
- [两个有序数组的中位数：二分划分](notes/algorithms/median-of-two-sorted-arrays.md)
- [最小栈：辅助栈保存前缀最小值](notes/algorithms/min-stack.md)
- [括号栈：有效性判断与最长连续长度](notes/algorithms/stack-parentheses.md)
- [双栈实现队列：搬运时机与均摊复杂度](notes/algorithms/queue-with-two-stacks.md)
- [字符串解码：用栈保存和恢复嵌套状态](notes/algorithms/stack-string-decoding.md)
- [二叉树层序遍历：队列、层边界与锯齿输出](notes/algorithms/binary-tree-level-order.md)
- [二叉树递归返回值：LCA 与最大路径和](notes/algorithms/binary-tree-recursion-return-values.md)
- [二叉树遍历与 BST 验证：中序、前序和前中序还原](notes/algorithms/binary-tree-traversal-and-construction.md)
- [双指针：快慢指针、就位区、逆向合并、判环与走两程](notes/algorithms/two-pointers.md)
- [摩尔投票法（多数元素）](notes/algorithms/moore-voting.md)
- [打家劫舍：选或不选型 DP 与破环技巧](notes/algorithms/dp-house-robber.md)
- [最大子数组问题：以 i 结尾型 DP（53/152/718）](notes/algorithms/dp-max-subarray.md)
- [表达式求值：栈与中缀优先级（227 计算器 II）](notes/algorithms/stack-expression-evaluation.md)
- [拓扑排序与环检测（207 课程表）](notes/algorithms/topological-sort.md)
- [快排与快速选择：双向划分与第 K 大](notes/algorithms/quick-sort.md)
- [随机数扩域与拒绝采样（470 Rand7→Rand10）](notes/algorithms/rejection-sampling.md)
- [反转链表家族：206 整链、92 区间与 25 分组翻转](notes/algorithms/linked-list-reversal.md)
- [链表合并家族：21/23 合并、148 归并与 2 竖式相加](notes/algorithms/linked-list-merge.md)
- [链表快慢指针家族：141 判环、142 找入口与找中点](notes/algorithms/linked-list-fast-slow.md)
- [链表删除家族：83 留一个与 82 整段删](notes/algorithms/linked-list-deletion.md)

### Python

- [Python 除法与取整语义：/、//、int() 的方向之争](notes/python/division-and-rounding.md)
- [map() 与内置函数组合的一行流](notes/python/map-one-liners.md)
- [Python 可变默认参数陷阱与 dataclass 的 default_factory](notes/python/mutable-default-args.md)
- [Python asyncio、协程与异步生成器](notes/python/asyncio-coroutines-and-generators.md)

## 面试准备

- [面试题库](interview/index.md)
- [2027 秋招 Dashboard](interview-plan/autumn-2027.md)

## 踩坑记录

- [KindVideo 慢 SQL 审计：日志噪音、索引失效、回表与重复轮询](pitfalls/kindvideo-slow-sql-audit-and-remediation.md)
- [DeepTutor 记忆修改 JSON 被尾随花括号干扰（上游案例）](pitfalls/deeptutor-memory-json-trailing-braces.md)
- [示例全过、提交就挂：链尾边界防御缺失（链表题三次同型翻车）](pitfalls/leetcode-examples-pass-submit-fails.md)
- [SQLAlchemy 只 flush 不 commit：SQLite 写锁卡死后续 INSERT](pitfalls/sqlalchemy-flush-no-commit-sqlite-lock.md)
- [Python 多重赋值把解包目标写成属性，静默改掉旧对象](pitfalls/python-tuple-unpack-attribute-target.md)
- [修 bug 只改组件不同步测试断言，留下永远失败的过期测试](pitfalls/stale-test-after-bugfix.md)
- [OrbStack 拉取 docker.io 镜像 EOF 失败（代理未生效）](pitfalls/orbstack-docker-pull-eof-proxy.md)
- [VARCHAR 列宽两次调整（20→64→32）：定宽前未核对真实数据长度](pitfalls/source-platform-id-column-width.md)
- [Alembic --autogenerate 在 schema 漂移的本地库上生成无关迁移](pitfalls/alembic-autogenerate-drift.md)
- [单 storage 假设（storages[0]）在多 storage 环境匹配不到 catalog 文件](pitfalls/multi-storage-catalog-match.md)
- [前端把 null/空值发给后端 → 原始英文 422 报错，交互体验差](pitfalls/empty-field-raw-422.md)
- [httpx 报 "Using SOCKS proxy, but 'socksio' not installed"——终端代理环境变量污染](pitfalls/httpx-socks-proxy-socksio.md)

## 资源书签

- [JavaGuide（Java 学习 + 面试指南）](resources/javaguide.md)

## 使用说明

1. 对我说「记笔记」「今天学了 X」即可归档主题笔记；「写学习日志」生成当天日志；「记个坑」记录踩坑；「收藏」保存资源书签；「记道面试题」积累面试题。任意会话中也可用 `/consolidate`。
2. 手动记录时，复制 `templates/` 下对应模板到目标目录并填写。
3. 文件之间的链接一律用标准 Markdown 相对路径链接，形成知识网络。
