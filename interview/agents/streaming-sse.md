---
type: interview
scope: topic
area: agents
topics:
  - streaming
  - sse
  - reconnect
created: 2026-08-27
updated: 2026-08-27
---

# Agent 流式输出与 SSE

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。

## StreamEvent 是什么？

<!-- canonical_key: agents.streaming.streamevent-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## StreamEvent 有哪些事件类型？

<!-- canonical_key: agents.streaming.streamevent-you-na-xie-shi-jian-lei-xing -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 中如何区分 token、tool call、ask_user、done、error？

<!-- canonical_key: agents.streaming.sse-zhong-ru-he-qu-fen-token-tool-call-ask-user-done-error -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果 SSE 断线了怎么办？

<!-- canonical_key: agents.streaming.ru-guo-sse-duan-xian-le-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 重连后如何恢复之前的事件？

<!-- canonical_key: agents.streaming.sse-zhong-lian-hou-ru-he-hui-fu-zhi-qian-de-shi-jian -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 为什么用 SSE 而不是 WebSocket？

<!-- canonical_key: agents.streaming.wei-shen-me-yong-sse-er-bu-shi-websocket -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## WebSocket 相比 SSE 有什么优缺点？

<!-- canonical_key: agents.streaming.websocket-xiang-bi-sse-you-shen-me-you-que-dian -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果前端收到 ask_user 事件后刷新页面，如何恢复 UI 状态？

<!-- canonical_key: agents.streaming.ru-guo-qian-duan-shou-dao-ask-user-shi-jian-hou-shua-xin-ye-mian-ru-he-hui-fu-ui-zhuang-tai -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 的底层原理是什么？

<!-- canonical_key: agents.streaming.sse-de-di-ceng-yuan-li-shi-shen-me -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 和普通 HTTP 请求有什么区别？

<!-- canonical_key: agents.streaming.sse-he-pu-tong-http-qing-qiu-you-shen-me-qu-bie -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 使用什么 Content-Type？

<!-- canonical_key: agents.streaming.sse-shi-yong-shen-me-content-type -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 为什么天然适合大模型流式输出？

<!-- canonical_key: agents.streaming.sse-wei-shen-me-tian-ran-shi-he-da-mo-xing-liu-shi-shu-chu -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 是单向还是双向通信？

<!-- canonical_key: agents.streaming.sse-shi-dan-xiang-hai-shi-shuang-xiang-tong-xin -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 浏览器断开连接后后端如何感知？

<!-- canonical_key: agents.streaming.liu-lan-qi-duan-kai-lian-jie-hou-hou-duan-ru-he-gan-zhi -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## FastAPI 怎么实现 StreamingResponse？

<!-- canonical_key: agents.streaming.fastapi-zen-me-shi-xian-streamingresponse -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 模型 token 是如何一路传到 React 前端的？

<!-- canonical_key: agents.streaming.mo-xing-token-shi-ru-he-yi-lu-chuan-dao-react-qian-duan-de -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何解决 token 输出和工具调用事件混杂的问题？

<!-- canonical_key: agents.streaming.ru-he-jie-jue-token-shu-chu-he-gong-ju-diao-yong-shi-jian-hun-za-de-wen-ti -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 一个 Agent 回答期间发生多个工具调用，你怎么组织事件流？

<!-- canonical_key: agents.streaming.yi-ge-agent-hui-da-qi-jian-fa-sheng-duo-ge-gong-ju-diao-yong-ni-zen-me-zu-zhi-shi-jian-liu -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何保证事件顺序？

<!-- canonical_key: agents.streaming.ru-he-bao-zheng-shi-jian-shun-xu -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 传输过程中发生异常怎么处理？

<!-- canonical_key: agents.streaming.sse-chuan-shu-guo-cheng-zhong-fa-sheng-yi-chang-zen-me-chu-li -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果模型生成速度比前端消费速度快怎么办？

<!-- canonical_key: agents.streaming.ru-guo-mo-xing-sheng-cheng-su-du-bi-qian-duan-xiao-fei-su-du-kuai-zen-me-ban -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 有没有 backpressure 问题？

<!-- canonical_key: agents.streaming.you-mei-you-backpressure-wen-ti -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## nginx 等反向代理会不会缓冲 SSE？

<!-- canonical_key: agents.streaming.nginx-deng-fan-xiang-dai-li-hui-bu-hui-huan-chong-sse -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## SSE 心跳怎么设计？

<!-- canonical_key: agents.streaming.sse-xin-tiao-zen-me-she-ji -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如果后续加入语音实时对话，你还会选择 SSE 吗？

<!-- canonical_key: agents.streaming.ru-guo-hou-xu-jia-ru-yu-yin-shi-shi-dui-hua-ni-hai-hui-xuan-ze-sse-ma -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何给每个 StreamEvent 分配 event id？

<!-- canonical_key: agents.streaming.ru-he-gei-mei-ge-streamevent-fen-pei-event-id -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 前端如何避免重复消费重连后的事件？

<!-- canonical_key: agents.streaming.qian-duan-ru-he-bi-mian-zhong-fu-xiao-fei-zhong-lian-hou-de-shi-jian -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-

## 如何记录一次完整 Agent 流式请求的 trace？

<!-- canonical_key: agents.streaming.ru-he-ji-lu-yi-ci-wan-zheng-agent-liu-shi-qing-qiu-de-trace -->

### 考察点

- 核心概念、运行机制、异常边界与设计取舍

### 回答框架

- 先定义概念和职责边界，再说明正常流程、异常路径与工程取舍；只引用已验证的知识或实现。

### 易追问

-

### 关联笔记

-
