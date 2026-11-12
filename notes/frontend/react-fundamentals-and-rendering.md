---
type: Note
domain: frontend
title: React 核心概念与渲染流程
aliases: [React 渲染, Props 与 State, React Hooks]
description: 从 React、Umi、Ant Design 的分工，到组件通信、状态快照、渲染提交、Effect 与性能优化
tags: [react, typescript, umi, antd, jsx, hooks, rendering]
status: learning
created: 2026-09-27
updated: 2026-09-27
---

# 概述

- **React** 管组件与界面更新，**TypeScript** 在开发时检查类型，**Umi** 组织路由、构建和应用约定，**Ant Design** 提供可复用的 React UI 组件。Umi 和 Ant Design 都不是 React 的替代品。
- 理解 React 的主线：输入（props、state）变化 -> 调用组件生成新的 UI 描述 -> React 提交必要的 DOM 变更 -> Effect 与外部系统同步。

# 详细内容

## 组件、JSX 与 DOM

- **组件**是返回 UI 的函数。父组件在 JSX 中使用子组件，并通过 **props** 传入数据或函数；子组件不能直接修改收到的 props。
- **JSX** 是在 JavaScript/TypeScript 中描述 UI 的语法，经编译后得到 React 元素；它不是 HTML 字符串，也不是浏览器里的 DOM 节点。
- **DOM** 是浏览器解析 HTML 后建立的可操作对象树。React 根据组件输出更新真实 DOM；HTML 源码、DOM 当前状态和 React 元素是三个不同层次。
- **State** 属于某次挂载的组件实例。调用 `setState` 是请求下一次渲染，不会立刻改写当前事件处理函数里捕获的状态值；同一轮更新可能被批处理。

```tsx
function Counter() {
  const [count, setCount] = React.useState(0);
  return <button onClick={() => setCount((previous) => previous + 1)}>{count}</button>;
}
```

- 上例传给 `onClick` 的是一个**回调函数**：点击发生时才调用。回调只是“把函数交给别处稍后调用”，并不意味着异步。
- 子组件通知父组件时，父组件传入回调，子组件在事件中调用；共享状态通常提升到最近的共同父组件。Context 适合跨层传递数据，但不能替代对状态归属的判断。

## 为什么会多次渲染

- 首次挂载、组件自身 state 变化、父组件重新渲染，或订阅的 Context 值变化，都可能让组件函数再次执行。
- **渲染（render）** 是计算下一版 UI；**提交（commit）** 才把必要变化写入 DOM。重新执行组件函数不等于整个页面重建，也不保证真实 DOM 一定发生变化。
- 每次渲染读取的是该次的 props/state **快照**。在一次事件处理函数中连续调用 `setCount(count + 1)` 可能都基于相同旧值；需要依赖前一次更新时用函数式写法 `setCount((previous) => previous + 1)`。
- 列表里的稳定 **key** 帮 React 识别元素身份；用数组下标作为可增删、可排序列表的 key，可能导致局部状态错位。改变 key 则会重建对应组件及其局部状态。

## Effect 与性能

- `useEffect` 用于与 React 外部系统同步，例如订阅、定时器、浏览器 API 或请求。依赖变化后重新同步，卸载或下次同步前执行清理函数；不要把本可在渲染中算出的值无故放入 Effect。
- `useEffect(() => { ... }, [id])` 中的依赖应包含 Effect 用到的响应式值；`[]` 表示不依赖变化的 props/state，不是“只运行一次”的跨环境保证。开发环境 Strict Mode 可能额外执行一次 setup/cleanup 以检查副作用。
- `React.memo` 可在 props 不变时跳过子组件的非必要渲染；`useMemo` 缓存计算结果，`useCallback` 缓存函数引用。先确认性能问题，再考虑缓存；它们不能代替正确的数据流。

## React、Umi、Ant Design 在项目中的分工

- 以 KindVideo 当前依赖为例：`frontend/package.json` 使用 `react`、`@umijs/max` 和 `antd`。React 写组件与 Hook，Umi 管应用框架，Ant Design 的 `Modal`、`Button`、`Spin` 等提供现成界面元素。
- TypeScript 给组件 props、回调和接口响应提供静态类型约束；运行时是否有效仍取决于实际数据和校验。

# 相关链接

- [浏览器输入 URL 后发生什么](browser-navigation-and-dom.md)
- [KindVideo 视频播放组件设计](kindvideo-video-playback-components.md)
- [2026-09-27 学习日志](../../daily/2026-09-27.md)
