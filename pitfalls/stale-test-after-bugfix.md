---
type: Pitfall
domain: frontend
title: 修 bug 只改组件不同步测试断言，留下永远失败的过期测试
description: 6fcd842e 修复 login SSO 回调循环时只改组件、没改测试期望值，断言停留在旧 bug 行为；4 天后任何人跑全量测试都会撞见这条"与自己改动无关"的失败
tags: [stale-test, testing, vitest, bugfix, test-sync]
created: 2026-08-24
updated: 2026-08-24
---

# 现象

- 修完一个与登录页完全无关的 bug（标注页面包屑）跑全量 `make test`，2143 条测试挂 1 条：
  `AssertionError: expected '/' to contain '/login'`（login.test.tsx SSO 回跳用例）
- 隔离单跑该文件仍失败（`1 failed | 6 passed`），排除测试间污染
- `git diff main --name-only` 证明本次改动不含任何 login 文件

# 排查过程

- **排除自身改动**：`git diff main --name-only` → 只改了 annotation/task.tsx，与 login 零交集
- **排除测试污染**：`npx vitest run tests/unit/pages/login.test.tsx` 单跑 → 仍失败，是稳定失败
- **定位行为变更点**：`git log -S "pathname === '/login'" -- <组件>` → 找到 `6fcd842e`（4 天前，他人提交，修 SSO 回调循环 bug）
- **确认脱节**：`git show 6fcd842e --stat` → 该提交**只改组件 index.tsx，没动测试文件**。测试期望值还停留在修复前的旧行为

# 根因

- **修 bug 只改实现、不同步断言**：`6fcd842e` 把"在 /login 页无 redirect 时回调目标 = /login"改成 `= '/'`（防 SSO 成功后 302 回登录页卡死），但测试断言 `toContain('/login')` 没跟着改
- 过期测试（stale test）的本质：**断言期望值是照"当时的真实行为"写的，行为被有意变更后期望值没更新**——测试从"守护正确行为"变成"给旧 bug 背书"
- 反直觉信号：这条测试**失败恰恰说明修复生效**；哪天它"通过"反而说明防循环逻辑坏了、bug 复发

# 解决方案

- **修 bug 改变可观测行为时，必须同步检索并更新断言该行为的测试**：
  - 提交前跑受影响测试文件，而不是只跑新增测试
  - `git grep` 断言关键字（如回调地址、错误文案）找出会受影响的用例
- **三步法判断测试失败是否与自己改动相关**：
  1. `git diff main --name-only` 看改动面是否触及
  2. 隔离单跑排除测试间状态污染
  3. `git log -S "<行为特征串>"` 定位行为引入/变更的提交与作者
- 修过期测试时改断言为**新行为**，顺带补齐新逻辑分支的覆盖（如本例：/ 分支 + 非 /login 页用当前 URL 分支）
- 存量失败不阻塞合入（用上面三步法证明无关后可继续），但要记录并跟进，不能装作没看见

# 相关链接

- （frontend 测试主题笔记待沉淀，暂无已存在笔记可链；案例详细分析见 KindVideo 仓库 `local/过期测试-login-SSO回跳断言.md`）
