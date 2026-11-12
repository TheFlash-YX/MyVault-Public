---
type: Pitfall
domain: agents
title: DeepTutor 记忆修改 JSON 被尾随花括号干扰：有效指令解析为空
description: 上游 PR 645 的记忆修改解析案例，以及公共 JSON parser 对尾随解释、think 标签和前置 schema 示例的后续修复
tags: [deeptutor, memory, json, llm-output, regression-test]
created: 2026-09-08
updated: 2026-09-23
---

# 现象

- 案例性质：上游开源 Bug 学习，非已验证的个人实习排障或 PR 贡献；与 KindPartner 可控记忆系统的学习相关，但尚未检查 KP 是否存在同一问题。
- DeepTutor 的 `parse_edits_payload` 解析模型生成的记忆修改指令，支持替换、插入、删除等操作。
- 当合法 JSON 后附带含花括号的解释文字时，旧逻辑可能解析失败并返回空修改列表，使有效指令被丢弃。
- 影响边界：确认的是修改指令丢失，不是已有记忆文件被删除；也没有证据证明所有记忆更新路径都受影响。

# 排查过程

- 官方 [PR #645](https://github.com/HKUDS/DeepTutor/pull/645) 由 `santhreal` 提交，2026-07-23 合并；修复提交为 `69582220ef0231a6faf08577b3b5250f0ebc8d7b`。
- 查看 [代码与测试差异](https://github.com/HKUDS/DeepTutor/pull/645/files)：修改点是 `deeptutor/services/memory/consolidator/line_doc.py` 中的 `_extract_json`，新增测试位于 `tests/services/memory/test_line_doc.py`。
- 沿调用链检查：`_extract_json` 提取文本，`parse_edits_payload` 再调用 `json.loads`；旧实现解析失败时记录 warning 并返回 `[]`，不是完全没有日志。
- 2026-09-08 会话中已在本地运行核心逻辑最小复现：旧截取触发 `Extra data`，`raw_decode` 成功读取修改对象；未运行 DeepTutor 全量测试，也未验证 KP 本地实现。

# 根因

- 旧算法选择第一个 `{` 或 `[` 作为起点、最后一个 `}` 或 `]` 作为终点，没有按 JSON 语法识别边界。
- 模型返回结构化数据后仍可能追加自然语言；解释文字中的括号会扩大截取范围。
- 调用方用空列表表达解析失败，使“没有修改”和“修改无法解析”在返回值层面难以区分。

# 解决方案

## 上游修复

- 保留寻找对象/数组起点的逻辑，改用 `json.JSONDecoder().raw_decode` 获取第一个完整 JSON 值及其结束偏移。
- 结束偏移相对于传入的子串，因此截取原文本时使用 `text[start:start + end]`。
- JSON 之外的尾随文字不进入后续解析；新增回归测试确认尾随 `trailing {brace}` 时仍产生一条 `ReplaceLineOp`，且 `new_text == "ok"`。
- 实现与测试依据：[修复提交](https://github.com/HKUDS/DeepTutor/commit/69582220ef0231a6faf08577b3b5250f0ebc8d7b)。

## 最小复现

- 以下仅演示本案例的对象输入，省略上游数组起点选择、代码围栏清理和操作类型转换；可直接用 Python 标准库运行。

```python
import json

raw = (
    '{"edits":[{"op":"replace","line":1,"new_text":"ok",'
    '"refs":["a:b"],"reason":"y"}]} trailing {brace}'
)
start = raw.find("{")

# 旧逻辑把尾随解释也截入 JSON。
old = raw[start:max(raw.rfind("}"), raw.rfind("]")) + 1]
try:
    json.loads(old)
except json.JSONDecodeError as exc:
    print(exc.msg)  # Extra data
else:
    raise AssertionError("旧截取应解析失败")

# 修复后的边界识别。
parsed, end = json.JSONDecoder().raw_decode(raw[start:])
snippet = raw[start:start + end]
assert json.loads(snippet) == parsed
assert len(parsed["edits"]) == 1
assert parsed["edits"][0]["new_text"] == "ok"
assert raw[start + end:] == " trailing {brace}"
```

## 修复边界与进一步思考

- 此修复处理合法 JSON 后追加文字，不修复非法 JSON，也不保证修改内容业务正确。
- 起点仍来自首个括号；若前置解释先出现非 JSON 花括号，不能保证找到后面的有效对象。
- 语法解析成功后仍需验证字段结构、操作类型、引用和目标范围。
- 可考虑分别表达“无修改”和“解析失败”，记录失败指标；这是后续设计建议，不是该 PR 已实现的内容。
- 可补测纯 JSON、数组、代码围栏、尾随解释、字符串内括号和损坏 JSON；该 PR 新增测试明确覆盖的是尾随花括号解释这一回归场景。

## 2026-09-21 补充：公共 JSON Parser 的后续修复

- 本次依据本地 Git 历史、`deeptutor/utils/json_parser.py` 与 `tests/utils/test_json_parser.py` 复核；没有重新在线核对 GitHub，也未重新执行测试。以下是相关修复，不能全部归为 PR #645 的改动。
- `5c3e5bf5`：先用 `raw_decode` 提取合法 JSON，再尝试 `json-repair`。原因是合法对象后追加含括号的解释，repair 可能把对象和解释包装成数组，使期待对象的调用方拿到错误类型。
- `b020686c`（#675）：处理 `<think>...</think>` 推理内容，避免其中的花括号干扰 JSON 提取。
- `34ca1cb2`（#692）：扫描可解码的对象/数组候选并选择消费文本最长的值，防止推理前言里的短 schema 示例先被匹配、遮住后面的真正结果；提交记录描述的影响包括 Book SpineAgent 回退到单个 Overview 章节。

```text
前置示例：{"chapters": []}
真正输出：{"chapters": [{"title": "Intro"}, {"title": "Methods"}]}
```

- “第一个合法 JSON”会返回空章节示例；“最长可解码 JSON”在这个案例中会返回实际章节对象。合法 JSON 与正确业务结果是两层判断。
- 当前公共解析顺序：整段 `json.loads` → 提取 Markdown 围栏后重试 → 清理 think 内容后重试 → 最长可解码对象/数组 → `json-repair` → 失败返回 fallback（默认 `{}`）。
- 流程图展示公共入口各阶段的顺序、成功返回路径与失败兜底；`raw_decode` 扫描器只尝试 `{}` 对象和 `[]` 数组，最终选择解析文本长度最长的候选。

```mermaid
flowchart TD
    A[parse_json_response response] --> B{响应为空?}
    B -- 是 --> F[返回 fallback 默认 {}]
    B -- 否 --> C[json.loads 整段响应]
    C --> D{解析成功?}
    D -- 是 --> R1[返回 Python 对象]
    D -- 否 --> E[提取 Markdown 代码块 若存在]
    E --> G[json.loads 提取结果]
    G --> H{解析成功?}
    H -- 是 --> R2[返回 Python 对象]
    H -- 否 --> I{含 think 标签?}
    I -- 是 --> J[清理成对或未闭合 think 前缀]
    I -- 否 --> M[扫描完整 JSON 候选]
    J --> K{清理后为空?}
    K -- 是 --> F
    K -- 否 --> L[json.loads 清理结果]
    L --> N{解析成功?}
    N -- 是 --> R3[返回 Python 对象]
    N -- 否 --> M
    M --> O[从对象或数组起点调用 raw_decode]
    O --> P{找到候选?}
    P -- 是 --> Q[返回消费文本最长的 JSON 值]
    P -- 否 --> S{json-repair 可用?}
    S -- 否 --> F
    S -- 是 --> T[repair_json 修复后再 json.loads]
    T --> U{解析成功?}
    U -- 是 --> R4[返回 Python 对象]
    U -- 否 --> F
```
- 整段 JSON 优先解析，可以保留合法 JSON 字符串中的代码围栏或字面 `<think>` 文本，避免预清理破坏本来正确的数据。
- Memory Consolidator 的 `_parse_facts`、`line_doc.py` 等仍有各自的解析路径，不能说所有模块都已经统一采用“最长值”策略。
- 最长候选只是启发式：更长的示例仍可能胜出，损坏的外层 JSON 也可能包含合法内层对象；解析后仍应验证预期类型、必需字段及业务约束。
- 现有测试包含尾随解释、前置 schema、相邻 JSON、think 标签及合法字符串保留等案例；本次记录是阅读测试覆盖，不能当成新执行的通过报告。

## 面试使用边界

- 可以作为“研究过的记忆模块可靠性案例”讨论，讲清现象、解析边界、回归测试及局限。
- 不能直接说“我在实习中发现并修复”；若后续在 KP 复现，应另行记录实际版本、操作、修复和验证证据。

# 相关链接

- [2026-09-21 学习日志](../daily/2026-09-21.md)
- [DeepTutor 三级可控学习记忆](../notes/agents/projects/deeptutor/learning-memory-system.md)
- [官方 PR #645](https://github.com/HKUDS/DeepTutor/pull/645)
- [固定提交源码](https://github.com/HKUDS/DeepTutor/blob/69582220ef0231a6faf08577b3b5250f0ebc8d7b/deeptutor/services/memory/consolidator/line_doc.py)
