---
name: consolidate
description: 将用户明确要求记录的学习内容沉淀到个人 MyVault 知识库。适用于记笔记、学习日志、踩坑记录、资源书签、面试题、面经和“/consolidate”等请求。
---

# Consolidate / 学习知识沉淀

把学习内容固定写入 MyVault，并保持查重、索引和标准 Markdown 双向链接完整。

## 输出与权威规则

```bash
OUTPUT_ROOT="${MYVAULT_ROOT:-/Users/yangxin/MyProjects/MyVault}"
```

- 写入前完整读取 `{OUTPUT_ROOT}/CLAUDE.md`；它是目录、schema、路由与链接规则的权威来源。
- 若本 skill 与 `CLAUDE.md` 的 interview 规则不一致，以 `CLAUDE.md` 为准，停止自动记题并向用户报告规则漂移；修复并验证后再继续。
- 只修改 `{OUTPUT_ROOT}` 内的知识库文件；本 skill 的维护任务除外。

## 基础结构探测

普通沉淀流程只要求存在：`notes/`、`daily/`、`pitfalls/`、`resources/`、`interview/`、`templates/`、`index.md`。

缺少任意一项则停止并提示：

> 输出目录 `{OUTPUT_ROOT}` 似乎不是一个 MyVault 知识库项目。如需初始化，请告诉我“初始化知识库结构”。

`interview-plan/` 不属于基础强制结构；仅在用户提出秋招、面试计划、模拟面试或高频复习时检查。

## 通用约束

- 文件名使用小写 kebab-case。
- 文件链接一律使用 `[显示文本](相对路径)`；禁止 `[[...]]` 和 `[[...]](path)`。
- 新链接必须指向真实文件，不留悬空链接。
- 更新已有笔记、踩坑、书签或题集时，将 `updated` 改为当天日期。
- 正文以清晰短段和列表组织；每条只表达一件事。
- 新建文件更新对应索引；根 `index.md` 不展开全部 interview 题集。
- 不自动执行 `git commit`、stash 或 reset；除非用户明确要求当前 Git 操作。
- 不向外部系统发送知识库内容。

## notes 领域路由

| 内容 | 目录 | domain |
|---|---|---|
| Java 语言 | `notes/java/` | java |
| 后端 | `notes/backend/{sub}/` | backend |
| Web 前端 | `notes/frontend/` | frontend |
| AI Agent / LLM 应用 | `notes/agents/` | agents |
| 算法与数据结构 | `notes/algorithms/` | algorithms |
| Python 语言 | `notes/python/` | python |

backend 笔记必须进入子域：

- `database`：数据库、ORM、锁与事务、分库分表。
- `redis`：Redis、缓存结构与缓存模式。
- `mq`：Kafka、RabbitMQ、可靠投递与削峰。
- `architecture`：DDD、存储、选型取舍与质量闭环。

## 意图路由

| 触发意图 | 工作流 |
|---|---|
| 记笔记、今天学了、沉淀一下 | 主题笔记 |
| 写学习日志、学习日志 | 日志 |
| 记个坑、踩坑 | 踩坑 |
| 收藏、记个书签 | 书签 |
| 记道面试题、面试题、面经 | 面试题 |
| 秋招、面试计划、模拟面试、高频复习 | interview-plan |

意图确实无法判断时才询问类型，不凭空猜测内容。

## 主题笔记

1. 按领域与 backend 子域路由。
2. 扫描 `notes/` 全部文件，结合文件名和 frontmatter 的 title/description/aliases 做中英文、同义和近义查重；宁可更新旧文件，不制造重复笔记。
3. 命中时只合并新认知，语义重复则改写原条目；更新 `updated`。
4. 未命中时使用 `templates/note.md` 新建并更新根 `index.md` 对应主题分组。
5. 当天日志存在时，与笔记建立标准 Markdown 双向链接。

不得因为 interview 中存在一道题而机械创建 notes。

## 学习日志

1. `daily/YYYY-MM-DD.md` 已存在时先询问覆盖、追加或取消。
2. 只提取当前会话和可验证的当日 Git 摘要，不编造学习内容。
3. 使用 `templates/daily.md`，保留今日学习、明日计划、疑问卡点、感想收获、关联笔记五节；空节保留 `-`。
4. 根 `index.md` 最近日志顶部插入，只保留最近 10 篇。
5. 每篇选择 0～3 个真正可复用内容沉淀到 notes、pitfalls 或 resources，并建立双向链接。

## 踩坑与书签

- 踩坑先按文件名和 title/description 查重；命中时补充新结论并更新 `updated`，未命中时使用 `templates/pitfall.md`，更新根索引并链接已有相关笔记。
- 书签按 url 或 title 查重；未命中时使用 `templates/resource.md`，写明简介与简评并更新根索引。

## 面试题

面试题一个主题一个文件、一文件多题；`notes` 管完整知识，`interview` 管 60～90 秒表达，`interview/resume` 管个人经历真实性。

### Schema 与路由

正式题集必须包含：

```yaml
type: interview
scope: topic | resume
area: agents | backend | java | python | frontend | resume
topics: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
```

- 通用知识、八股和系统设计使用 `scope: topic`，按 area 进入对应目录。
- “你的项目”“你的简历”“你的指标”等个人经历问题使用 `scope: resume`、`area: resume`。
- resume 可增加 `project: kindpartner | kindvideo`、`category: behavioral | motivation`、`resume_version: 2026-08`。
- interview 禁止使用 `domain` 或 `domains` 路由。

### 全局查重与 canonical_key

1. 扫描 `interview/**/*.md` 做全局语义查重，再选择目标文件。
2. Exact Duplicate 只保留一个 canonical question；Semantic / Contextual Duplicate 可分别保留通用题和项目场景题，但场景题只链接通用题，不复制完整原理。
3. 每道正式题必须在标题后紧跟全局唯一 `<!-- canonical_key: {namespace}.{topic}.{slug} -->`。
4. canonical_key 使用小写 kebab-case 语义标识，不含 Q 编号、日期或 `question`，建立后原则上不改。
5. 新题以考察点、回答框架、易追问、关联笔记组织；不默认编写完整教程或虚构项目证据。
6. 新建题集更新 `interview/index.md`；根 `index.md` 只保留面试题库与 Dashboard 入口。

### 来源迁移账本

批量导入带 source_id 的题库时，必须维护 `interview/meta/` 中的 migration ledger。每个 source_id 恰好一行，记录合法 disposition（moved、merged、split、excluded）、canonical_file、canonical_key、merged_with 和必要说明。所有非 excluded key 最终必须能定位到正式题集。

### 关联笔记

- 只链接已经存在的 notes；不存在时保留 `-`，不新建、不留悬空链接。
- 建立 interview ↔ notes 双向链接。
- 项目题需要底层知识时链接通用 interview 题，不复制通用答案。

## interview-plan

- 只保存模块掌握度、薄弱题链接、模拟面试记录和下一步动作。
- 不保存完整技术答案、教程、项目答辩正文或指标口径副本。
- 逐题不维护红黄绿；模块级状态写在 Dashboard。

## 相对链接速查

- 日志 → 普通笔记：`../notes/{domain}/xxx.md`；backend：`../notes/backend/{sub}/xxx.md`。
- 普通笔记 → 日志：`../../daily/YYYY-MM-DD.md`；backend：`../../../daily/YYYY-MM-DD.md`。
- interview → 普通笔记：`../../notes/{domain}/xxx.md`；backend：`../../notes/backend/{sub}/xxx.md`。
- 普通笔记 → interview：`../../interview/{area}/xxx.md`；backend：`../../../interview/{area}/xxx.md`。
- resume → 通用 interview：`../agents/xxx.md`、`../backend/xxx.md` 或 `../python/xxx.md`。
- interview-plan → interview：`../interview/{area}/xxx.md`；→ notes：`../notes/{domain}/xxx.md`。

## 交付

汇报新增与更新文件、查重命中、索引和链接变更。提醒用户 review 后自行 `git add` 与 `git commit`；若用户已明确授权某次 Git 操作，则只执行该次授权范围。
