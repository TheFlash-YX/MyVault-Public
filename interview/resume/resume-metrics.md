---
type: interview
scope: resume
area: resume
category: metrics
resume_version: 2026-08
topics:
  - metrics
  - evidence
  - measurement
created: 2026-08-27
updated: 2026-09-07
---

# 简历指标与证据口径

- 本文件是简历量化指标的唯一口径来源；原始题库只有问题，以下字段全部等待真实日志、测试报告、代码或数据库记录核验。

## 你写“重复执行率 0%”，这个指标怎么测出来的？

<!-- canonical_key: resume.metrics.ni-xie-zhong-fu-zhi-xing-lu-0-zhe-ge-zhi-biao-zen-me-ce-chu-lai-de -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 98% 的恢复成功率是怎么统计的？

<!-- canonical_key: resume.metrics.98-de-hui-fu-cheng-gong-lu-shi-zen-me-tong-ji-de -->

### 简历表述

- 若简历使用“恢复成功率 98%”或“上下文连续率 98%”，必须说明这是自建业务测试集口径，不是 DeepTutor 官方 benchmark。

### 指标定义

- 一条需要 `ask_user` 的多轮任务，在同 Turn pause/resume 后仍能关联原 `turn_id`、原 `tool_call_id`、pause 前关键 Tool Result，并正确使用用户补充信息完成原始任务，记为一次恢复成功。

### 样本量

- 建议口径：构造 N 条需要澄清的任务；若对外写 98%，需要能拿出 N、成功条数和失败样例。没有测试记录时不要直接承诺该数字。

### baseline

- baseline 可以是“直接结束当前 Turn，用户回复后新建 Turn”的普通多轮方案；对比它在 Tool Result 复用、状态连续和重复执行上的失败率。

### 统计周期

- 以实际测试批次或回归测试时间为准；未核验前不要写成线上长期指标。

### 计算公式

- 恢复成功率 = 成功恢复并完成原任务的样本数 / 触发 `ask_user` 的测试样本总数。

### 数据来源

- 当前笔记只提供合理定义；最终数据应来自测试报告、回归用例、日志或数据库记录。

### 环境

- 线上 / 测试 / 离线评测：优先标成离线评测或测试环境，除非已有线上埋点与统计表证明。

### 我的个人贡献

- 说明自己负责的是 pause/resume 机制、测试口径设计、用例构造还是数据统计；不能把未参与的线上指标写成个人贡献。

### 可能偏差

- Runtime continuity 不等于 Model semantic correctness：Runtime 可以保证消息和 `tool_call_id` 没断，但不能保证模型一定正确理解补充信息。
- 如果测试样本主要是短任务、单进程或单用户场景，不能外推到多实例、服务重启和长上下文任务。

### 当前证据状态

- 待补真实证据：测试样本列表、执行日志、成功/失败判定规则、失败样例归因。

## 剩下 2% 失败的原因是什么？

<!-- canonical_key: resume.metrics.sheng-xia-2-shi-bai-de-yuan-yin-shi-shen-me -->

### 简历表述

- 与“为什么不是 100%”同口径，优先引用失败归因，不维护第二份完整答案。

### 指标定义

- 同 [为什么不是 100%？](#为什么不是-100)。

### 样本量

- 同上。

### baseline

- 同上。

### 统计周期

- 同上。

### 计算公式

- 同上。

### 数据来源

- 同上。

### 环境

- 线上 / 测试 / 离线评测：同上。

### 我的个人贡献

- 同上。

### 可能偏差

- 同上。

### 当前证据状态

- 同上。

## 7000+ 探针数据从哪里来的？

<!-- canonical_key: resume.metrics.7000-tan-zhen-shu-ju-cong-na-li-lai-de -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 100+ 困难样本为什么只有 100 多？

<!-- canonical_key: resume.metrics.100-kun-nan-yang-ben-wei-shen-me-zhi-you-100-duo -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 任务周期缩短 35% 的 baseline 是什么？

<!-- canonical_key: resume.metrics.ren-wu-zhou-qi-suo-duan-35-de-baseline-shi-shen-me -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 1000+ 视频是多久积累的？

<!-- canonical_key: resume.metrics.1000-shi-pin-shi-duo-jiu-ji-lei-de -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 同 Turn 恢复成功率 98% 如何统计？

<!-- canonical_key: resume.metrics.tong-turn-hui-fu-cheng-gong-lu-98-ru-he-tong-ji -->

### 简历表述

- 语义重复题，统一引用：[98% 的恢复成功率是怎么统计的？](#98-的恢复成功率是怎么统计的)。

### 指标定义

- 同上。

### 样本量

- 同上。

### baseline

- 同上。

### 统计周期

- 同上。

### 计算公式

- 同上。

### 数据来源

- 同上。

### 环境

- 线上 / 测试 / 离线评测：同上。

### 我的个人贡献

- 同上。

### 可能偏差

- 同上。

### 当前证据状态

- 同上。

## 为什么不是 100%？

<!-- canonical_key: resume.metrics.wei-shen-me-bu-shi-100 -->

### 简历表述

- “不是 100%”要主动拆成 Runtime 连续性和模型语义正确性两层，避免把失败全部归因到 Runtime。

### 指标定义

- 失败样例可包括：正确恢复了 `tool_call_id` 但模型没有利用用户补充信息；长上下文下忽略 pause 前 Tool Result；多实例/重启场景下原内存 Queue 不存在。

### 样本量

- 以 98% 指标对应测试集为准。

### baseline

- 同 [98% 的恢复成功率是怎么统计的？](#98-的恢复成功率是怎么统计的)。

### 统计周期

- 同上。

### 计算公式

- 失败率 = 1 - 恢复成功率；失败样例必须逐条归因。

### 数据来源

- 测试报告、失败日志、模型输出与对应 `turn_id`/`tool_call_id` 追踪记录。

### 环境

- 线上 / 测试 / 离线评测：与统计口径保持一致。

### 我的个人贡献

- 说明自己是否负责失败归因、修复 Runtime 问题、补幂等校验或补测试集。

### 可能偏差

- 如果失败来自模型语义理解，不应说 Runtime 丢失上下文；如果失败来自多实例/重启，则要承认当前 live in-memory 方案边界。

### 当前证据状态

- 待补真实失败样例和归因表。

## 重复执行率 0% 有多少测试样本？

<!-- canonical_key: resume.metrics.zhong-fu-zhi-xing-lu-0-you-duo-shao-ce-shi-yang-ben -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 后台静默沉淀率 0% 是什么意思？

<!-- canonical_key: resume.metrics.hou-tai-jing-mo-chen-dian-lu-0-shi-shen-me-yi-si -->

### 简历表述

- 如果简历写“后台静默沉淀率 0%”，口径必须是：长期 Memory 不在用户不可见、不可撤销的情况下自动写入。

### 指标定义

- 触发长期 Memory 写入、修改、删除、Reset 或 Consolidator Run 时，用户是否有明确入口、可见结果和可回滚/纠错路径；没有用户可见控制的后台自动写入记为静默沉淀。

### 样本量

- 需要列出覆盖的写入入口和任务类型，例如手动写入、候选确认、Consolidator update、dedup、merge、undo。

### baseline

- baseline 是完全自动 Memory：系统从对话中后台抽取并写入长期画像，用户事前不可见、事后难纠错。

### 统计周期

- 以实际测试批次、代码版本或产品迭代周期为准。

### 计算公式

- 后台静默沉淀率 = 用户不可见且不可控的长期 Memory 写入次数 / 长期 Memory 变更总次数。

### 数据来源

- 需要来自 Memory mutation 日志、Consolidator Run 记录、UI 操作记录或测试用例，不应只靠设计描述。

### 环境

- 线上 / 测试 / 离线评测：未有线上埋点前，应标成测试或离线评测口径。

### 我的个人贡献

- 说明自己负责的是写入控制、用户确认、可见文档、Undo、Audit，还是测试口径设计。

### 可能偏差

- 0% 只能说明“没有不可见的长期 Memory 写入”，不等于没有自动候选抽取，也不等于 Memory 内容完全正确。

### 当前证据状态

- 待补真实证据：Memory 变更入口清单、Run 日志、测试用例和用户可见确认路径截图/记录。

## 引用覆盖率 100% 是业务指标还是技术指标？

<!-- canonical_key: resume.metrics.yin-yong-fu-gai-lu-100-shi-ye-wu-zhi-biao-hai-shi-ji-shu-zhi-biao -->

### 简历表述

- 更适合表述为技术一致性指标，不是业务效果指标。

### 指标定义

- 所有由 Consolidator 自动生成的 L2/L3 Memory Entry 都必须带合法引用；L2 引用 L1 Trace，L3 引用 L2 Entry，并且引用能被系统解析。

### 样本量

- 样本量应等于统计周期内自动生成或更新的 Memory Entry 数量；手动自由文本是否纳入要单独说明。

### baseline

- baseline 可以是无引用校验的 LLM 总结式 Memory，模型生成事实后直接覆盖或追加文档，无法保证来源可追溯。

### 统计周期

- 以一次回归测试、一次 Consolidator Run 批次或某个版本周期为准。

### 计算公式

- 引用覆盖率 = 带合法可解析引用的自动 Memory Entry 数 / 自动 Memory Entry 总数。

### 数据来源

- 来自 Reference Validation 结果、AddOp 写入日志、Memory 文档 footnote/reference rebuild 结果和测试报告。

### 环境

- 线上 / 测试 / 离线评测：优先写清是测试环境还是离线评测；没有线上统计时不要说成线上指标。

### 我的个人贡献

- 说明自己负责的是引用 schema、Reference Validation、AddOp invariant、测试断言，还是指标统计。

### 可能偏差

- 引用覆盖率 100% 只说明每条自动 Memory 都有合法来源，不代表 LLM 抽取出的语义事实 100% 正确。

### 当前证据状态

- 待补真实证据：Memory Entry 样本、引用校验日志、失败用例和修复记录。

## 跨作用域泄漏 0% 测了多少场景？

<!-- canonical_key: resume.metrics.kua-zuo-yong-yu-xie-lou-0-ce-le-duo-shao-chang-jing -->

### 简历表述

- 材料提到“跨作用域泄漏 0% / 越权调用拦截率 100%”；实际结果待测试报告核实，只能在有证据后限定为对应版本和回归集的结果。

### 指标定义

- 泄漏衡量非授权数据是否进入当前作用域；拦截衡量不允许的 Tool Call 是否在执行前被阻止，两者分别统计。
- 通用定义与测试方法见 [泄漏评测](../agents/agent-multi-tenancy.md#跨作用域泄漏-0怎么测) 和 [越权拦截评测](../agents/agent-multi-tenancy.md#越权调用拦截率-100怎么测)。

### 样本量

- 待补隔离用例总数、越权调用总数、失败数及去重方式；材料未提供真实数量，不能根据列出的示例数推算。

### baseline

- 待补是否存在对照实现或修复前结果；没有 baseline 时按验证性测试汇报。

### 统计周期

- 待补实际执行日期、项目提交版本、配置版本及重复运行次数。

### 计算公式

- 使用通用题集中的定义；报告需同时提供分子、分母，明确用例与调用的统计粒度，分母为 0 时记为未评测。

### 数据来源

- 当前输入是 [Partner 运行时复用与隔离复习稿](../meta/source/deeptutor-partner-runtime-reuse-and-isolation.md)，不是执行报告。
- 待补测试代码、Scope 矩阵、Canary 数据、最终工具集合、分发拒绝日志、Tool Result / Runtime Context 断言和 Memory 前后快照。

### 环境

- 材料按回归测试口径描述，真实测试环境和执行记录待补；不能转述为线上长期统计。

### 我的个人贡献

- 待补实际负责的 Scope Resolver、Tool Composition、关系记忆隔离或测试断言模块及对应提交。

### 可能偏差

- 只检查最终回答会漏掉工具结果或模型上下文中已经发生的泄漏；Canary 未命中也不能覆盖敏感信息被改写后的全部泄漏形式。
- 未构造非法调用、只观察模型未调用工具，不能证明拦截成功；正向用例占比与重复计数会影响泄漏率分母。

### 当前证据状态

- 已整理材料中的机制和测试设计；源码、测试样本数、执行结果及个人贡献证据均待补，本次未执行 DeepTutor 隔离测试。
- [相关笔记：DeepTutor Runtime 与 Partner 隔离](../../notes/agents/projects/deeptutor/agent-runtime-and-turn-lifecycle.md)

## F1 78% 是 answer F1 还是 retrieval F1？

<!-- canonical_key: resume.metrics.f1-78-shi-answer-f1-hai-shi-retrieval-f1 -->

### 简历表述

- 材料记录教育 QA 测试集平均 F1 78%；这是待报告核实的表述，不是本次重新评测的结果。

### 指标定义

- 按材料描述为 Generated Answer 与 Ground Truth Answer 的覆盖程度，即 Answer F1；具体是否为 Token F1、中文如何分词，仍需评分脚本确认。

### 样本量

- 待补 QA 总数、大学课件与中小学试题的分布、无效样本排除规则；不能从 78% 推算样本量。

### baseline

- 待补相同语料、问题、模型与预算下的对照方案及分数；没有对照记录时不声称 LightRAG 带来了特定提升。

### 统计周期

- 待补执行日期、项目提交、LightRAG 版本、索引与模型配置、重复运行次数。

### 计算公式

- 通用公式和聚合约定见 [Answer F1 计算题](../agents/rag.md#f1-78-是怎么计算出来的)；需要真实脚本确认逐题宏平均、多标准答案和空答案处理。

### 数据来源

- 来自 [LightRAG 复习稿](../meta/source/kindpartner-lightrag-knowledge-retrieval.md) 的口径描述：大学课程课件和中小学近十年期末考试真题；具体文件、年份与答案复核记录待补。

### 环境

- 材料描述为自建测试集评测；运行环境待核实，不转述为线上流量统计。

### 我的个人贡献

- 材料将工作定位为在既有框架中接入 LightRAG Provider、模型适配、Source 回填与效果评测；需补本人提交和评测脚本。

### 可能偏差

- 同义改写、分词、数字单位和额外内容会影响词面 F1；题目或标准答案进入检索库会造成污染，F1 也不等于答案事实准确率。

### 当前证据状态

- 已整理材料口径，未获得真实测试报告；待补逐题预测、标准答案、评分实现与汇总结果。相关机制见 [RAG Provider 笔记](../../notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md)。

## Recall@5 91% 对应多少测试样本？

<!-- canonical_key: resume.metrics.recall-5-91-dui-ying-duo-shao-ce-shi-yang-ben -->

### 简历表述

- 材料记录平均 Recall@5 91%；实际指标定义与结果待核实，不能用示例容量或百分比反推测试样本数。

### 指标定义

- 确认评测的是前五条最终 Evidence 的召回率；原稿单证据命中示例同时符合 Hit@5，不能据此确定多证据题使用了哪种实现。

### 样本量

- 待补 Query 总数、每题相关 Evidence 数、证据粒度、空 Ground Truth 和检索失败样本的处理。

### baseline

- 待补同一测试集上的 naive、hybrid 或其它对照；比较时固定模型、索引语料和预算，避免把多项变化归因于检索模式。

### 统计周期

- 待补运行日期、项目和依赖版本、索引版本、mode、内部 top_k / chunk_top_k、重排与去重配置。

### 计算公式

- Recall@5 与 Hit@5 的区别见 [召回率计算题](../agents/rag.md#recall5-91-是怎么计算的)；统计脚本需确认宏平均或微平均，以及最终 Top-5 的截取阶段。

### 数据来源

- 当前只有 [LightRAG 复习稿](../meta/source/kindpartner-lightrag-knowledge-retrieval.md)。待补 Question、标准证据集合、最终检索列表、排序结果与逐题分数。

### 环境

- 材料描述为教育 QA 回归评测，具体环境待补；不是线上长期召回统计。

### 我的个人贡献

- 需用提交或标注记录说明负责的 Provider 接入、证据标注、结果归一化和评测部分，区分既有 RAG 框架与本人工作。

### 可能偏差

- 相关文件与相关 Chunk 混算、只命中一个证据就算全部召回、试题答案泄漏、索引变更后旧 Evidence ID 失配，都可能导致指标失真。

### 当前证据状态

- 待补真实数据与脚本，当前不能确认 91% 是集合召回率还是命中率；原稿 26000 条向量仅为容量示例，不是 QA 样本量。相关知识见 [RAG Provider 笔记](../../notes/agents/projects/deeptutor/knowledge-base-and-rag-providers.md)。

## 这些指标是在真实生产流量还是测试环境得到的？

<!-- canonical_key: resume.metrics.zhe-xie-zhi-biao-shi-zai-zhen-shi-sheng-chan-liu-liang-hai-shi-ce-shi-huan-jing-de-dao-de -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 有没有置信区间？

<!-- canonical_key: resume.metrics.you-mei-you-zhi-xin-qu-jian -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 如果我现在让你现场解释指标计算代码，你能解释吗？

<!-- canonical_key: resume.metrics.ru-guo-wo-xian-zai-rang-ni-xian-chang-jie-shi-zhi-biao-ji-suan-dai-ma-ni-neng-jie-shi-ma -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 这些数据是否有日志、测试报告或数据库记录可以支撑？

<!-- canonical_key: resume.metrics.zhe-xie-shu-ju-shi-fou-you-ri-zhi-ce-shi-bao-gao-huo-shu-ju-ku-ji-lu-ke-yi-zhi-cheng -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 哪些指标是你亲自统计的，哪些来自团队已有数据？

<!-- canonical_key: resume.metrics.na-xie-zhi-biao-shi-ni-qin-zi-tong-ji-de-na-xie-lai-zi-tuan-dui-yi-you-shu-ju -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-

## 98% 是真实线上还是你的测试？

<!-- canonical_key: resume.metrics.98-shi-zhen-shi-xian-shang-hai-shi-ni-de-ce-shi -->

### 简历表述

-

### 指标定义

-

### 样本量

-

### baseline

-

### 统计周期

-

### 计算公式

-

### 数据来源

-

### 环境

- 线上 / 测试 / 离线评测：

### 我的个人贡献

-

### 可能偏差

-

### 当前证据状态

-
