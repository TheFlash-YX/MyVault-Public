---
type: Note
domain: frontend
title: 浏览器输入 URL 后的加载、渲染与水合
aliases: [浏览器输入 URL, HTML 与 DOM, Hydration 水合]
description: 从地址解析、网络请求到 DOM/CSSOM 与渲染，并区分 CSR、SSR 和水合
tags: [browser, http, dom, html, cssom, hydration, react]
status: learning
created: 2026-09-27
updated: 2026-09-27
---

# 概述

- 输入 URL 后，浏览器需要确定资源位置、获取响应、解析文档并绘制页面；JavaScript 框架还可能在此后接管交互。
- **HTML 是标记文本，DOM 是浏览器解析后可被 JavaScript 修改的对象树**。服务端返回了 HTML，不代表 React 事件已经能工作。

# 详细内容

## 从导航到显示

1. 浏览器解析 URL，确定协议、主机、端口、路径与查询参数；可能先命中浏览器缓存或被 Service Worker 处理，也可能遇到重定向。
2. 需要网络请求时，解析域名取得地址，建立连接。常见 HTTPS over TCP 需要 TCP 与 TLS 握手；HTTP/3 使用 QUIC，不能一概说“所有请求先建 TCP”。已有连接可以复用。
3. 浏览器发送 HTTP 请求，服务器返回状态、响应头和响应体。HTML 可能引用 CSS、JavaScript、图片等资源，浏览器还会按依赖继续请求。
4. 解析 HTML 建立 **DOM**，解析 CSS 建立 **CSSOM**，计算样式和布局，再绘制、合成像素。脚本、样式和资源加载时机可能影响这一过程；页面也可以分阶段显示。
5. JavaScript 运行后可以修改 DOM、发起 API 请求、注册事件。React 应用的组件渲染属于这一阶段的应用逻辑，不等于浏览器原生的 HTML 解析。

## HTML、DOM、JSX 的边界

| 概念 | 是什么 | 例子 |
|---|---|---|
| HTML | 文档的标记文本 | 响应体中的 `<button>保存</button>` |
| DOM | 浏览器当前内存中的节点树 | `document.querySelector('button')` 得到的节点 |
| JSX | React 组件中描述 UI 的语法 | `return <button>保存</button>` |

- 浏览器初次解析 HTML 会创建 DOM；脚本运行后 DOM 可以和原始 HTML 不同。查看“网页源代码”与开发者工具 Elements 面板看到的内容因此可能不同。

## CSR、SSR 与水合

- **CSR（客户端渲染）**：初始 HTML 通常只有挂载容器及脚本；浏览器下载并执行 JavaScript 后由 React 创建界面，常用 `createRoot`。
- **SSR（服务端渲染）**：服务器先生成可见 HTML，浏览器先显示内容；客户端 JavaScript 加载后用 `hydrateRoot` 将 React 逻辑接到已有 DOM 上，这一步称**水合（hydration）**。
- 水合不是“再生成一次 HTML”或“下载数据”本身；关键是 React 复用已有标记并建立可交互的组件树。服务器与客户端首次渲染内容应一致，否则可能出现 hydration mismatch。
- 即使页面由 Umi/React 构建，也不能仅凭框架名称断定它使用 SSR；要看该页面的实际渲染方式。

# 相关链接

- [React 核心概念与渲染流程](react-fundamentals-and-rendering.md)
- [2026-09-27 学习日志](../../daily/2026-09-27.md)
