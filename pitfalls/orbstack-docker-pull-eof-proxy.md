---
type: Pitfall
domain: backend
title: OrbStack 拉取 docker.io 镜像 EOF 失败（代理未生效）
description: OrbStack 默认只读 macOS 系统代理，终端代理环境变量不被继承，导致 VM 直连 docker.io 被重置；需手动设置 network_proxy
tags: [orbstack, docker, proxy, clash, macos]
created: 2026-08-20
updated: 2026-08-20
---

# 现象

- `docker pull mysql:9.6`（OrbStack context）反复失败：`Head "https://registry-1.docker.io/v2/...": EOF`
- 同一时刻宿主机 `curl https://registry-1.docker.io/v2/` 正常（401），colima 的 VM 也能正常拉取同一镜像
- 容器内测试：DNS 解析正常（返回真实 AWS IP），但 `curl https://registry-1.docker.io` 返回 000 连接失败

# 排查过程

- 用本地已有镜像进 OrbStack VM 内部分别测 DNS 与 TCP：DNS 通、HTTPS 连接被重置 → 定位是网络干扰而非 DNS 污染
- 检查终端环境变量：`http_proxy/https_proxy/all_proxy` 指向 `127.0.0.1:7897`（本机 Clash Verge/mihomo）→ 宿主机走代理所以通
- 检查 macOS 系统代理：`scutil --proxy` 全部 Enable=0（关闭状态）
- 检查 OrbStack 配置：`orb config show` 显示 `network_proxy: auto` —— auto 只读 macOS 系统代理，读不到就直连
- 对比 colima：Lima 会自动把宿主机终端的 `*_proxy` 环境变量传播进 VM，所以 colima 一直正常
- 确认 Clash 只监听 `127.0.0.1:7897`（mihomo 以 root 运行，未开 allow-lan），VM 内部无法直接访问该地址

# 根因

- OrbStack 的 `network_proxy: auto` 只读取 **macOS 系统代理**设置，不会继承终端的 `http_proxy` 等环境变量；系统代理未开启 → OrbStack 的 VM 直连 docker.io → 连接被重置（EOF）
- colima/Lima 与 OrbStack 行为差异：Lima 自动传播终端代理变量，OrbStack 不传播
- 关键点：OrbStack 的代理连接由**宿主机侧**网络栈发起，因此配置 `127.0.0.1:7897` 即可命中绑定在回环地址上的 Clash，无需开启 allow-lan

# 解决方案

```bash
orb config set network_proxy http://127.0.0.1:7897
orb restart
```

- 验证：`docker pull hello-world` 成功；容器内 `curl https://registry-1.docker.io/v2/` 返回 401（正常）
- 备选方案：开启 Clash Verge 的「系统代理」开关，OrbStack auto 模式即可自动读到（但会影响全系统流量）
- 注意：Clash 端口变更后需同步更新该配置，否则拉镜像会再次 EOF

# 相关链接

-
