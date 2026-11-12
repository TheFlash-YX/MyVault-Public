---
type: Pitfall
domain: backend
title: httpx 报 "Using SOCKS proxy, but 'socksio' not installed"——终端代理环境变量污染
description: 本机 Clash 的 all_proxy=socks5 环境变量被 httpx 自动读取，而依赖没装 socksio，跑脚本/集成测试直接 ImportError
tags: [httpx, proxy, socks, clash, environment]
created: 2026-08-24
updated: 2026-08-24
---

# 现象

- 跑后端脚本 / 集成测试（无网络场景）报：
  `ImportError: Using SOCKS proxy, but the 'socksio' package is not installed`
- 5 个本不相关的集成测试一起挂，报错与业务逻辑毫无关系

# 排查过程

- 逐个跑挂掉的测试，报错高度一致，都是 `httpx` 发起请求时触发 → 怀疑是共享环境因素
- `env | grep -i proxy`：终端里有 `all_proxy=socks5://127.0.0.1:7897`（Clash 默认导出）
- httpx 客户端默认读取 `all_proxy`/`HTTPS_PROXY` 等环境变量，把 socks5 代理套上，而项目依赖只装了 `httpx`（不带 `[socks]`）→ 缺 `socksio`
- `unset` 所有代理环境变量后重跑，全部通过

# 根因

- 本机 Clash 把代理导出为 **socks5 环境变量**，httpx 自动拾取；socks 协议需要额外依赖 `socksio`，而项目未装
- 属于**开发机环境**问题，与业务代码无关，隐蔽性高（本地代理开着才炸）

# 解决方案

- 跑脚本/集成测试前清掉代理：`unset all_proxy ALL_PROXY HTTP_PROXY http_proxy HTTPS_PROXY https_proxy NO_PROXY no_proxy`
- 或装 `httpx[socks]`（项目真要走 socks 代理时）
- 判断依据：报错与业务无关、且多个无关测试同挂 → 先查环境变量/网络代理，别怀疑代码

# 相关链接

- [OrbStack 拉镜像 EOF 代理坑](./orbstack-docker-pull-eof-proxy.md)（同类：本机代理环境影响的另一表现）
