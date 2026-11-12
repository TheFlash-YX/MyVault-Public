---
type: Note
domain: frontend
title: KindVideo 视频播放组件设计
aliases: [VideoPlaybackModal, VideoPlayerSection, useVideoPlayback]
description: 基于当前源码梳理通用视频弹窗、播放地址 Hook 与 BEV 标注页的播放器复用边界
tags: [kindvideo, react, antd, video, hooks, component-design]
status: learning
created: 2026-09-27
updated: 2026-09-27
---

# 概述

- KindVideo 当前前端有两条相关组件链路：**通用视频播放弹窗**用于目录文件查看与数据集/标签操作；**`VideoPlayerSection`**用于教评标注工作台，并被 BEV 标注页复用。它们不是同一个播放器容器。
- 共通思路是：页面决定业务上下文，Hook 获取播放地址并管理状态，展示组件负责播放和错误反馈；后端只返回播放 URL，视频流由浏览器的 `<video>` 请求。
- 以下是对 2026-09-27 所见源码的归纳，不自动证明历史版本或个人贡献。

# 详细内容

## 通用播放弹窗

- `VideoPlaybackModal` 接收 `open`、`catalogFileId`、数据集/标签数据及增删回调。数据集与标签操作由父层传入，弹窗自身调用 `useVideoPlayback` 获取播放地址，再组合 `VideoPlayer`、`DatasetCell` 和 `TagCell`。
- `VideoPlayer` 接收 `url`、`loading`、`error`、`videoError` 与 `onVideoError`；先区分获取地址中、获取地址失败、无地址，再渲染原生 `<video controls autoPlay>`。视频播放阶段的错误用覆盖层提示，和取地址失败分开。
- `<video key={url}>` 在地址改变时重新挂载播放器，避免沿用上一个视频的播放状态。容器采用 16:9 比例，视频内容用 `objectFit: contain` 完整显示。
- `useVideoPlayback` 在有效 `catalogFileId` 且 `autoFetch` 开启时通过 Effect 请求 `getCatalogFilePlayUrl`；请求前清空旧 URL 和错误，维护加载、取地址错误及媒体播放错误。当前实现没有在 Effect 清理中取消旧请求；快速切换 ID 时是否出现旧响应覆盖新响应，需要复现或测试，不能当成已验证故障。

## 教评与 BEV 工作台

- `VideoPlayerSection` 是教评标注页的视频区域，除 `<video>` 外还有前后项导航、倍速、可选通道以及标注画布等 props；这比通用弹窗承担更多工作台交互。
- BEV 页面复用该组件和 `useVideoPlayback`，只选可播放的 `catalog_file_id` 请求地址。它传入 `lessonVideos={[]}`、空画布状态和无操作处理函数，使通道与绘制等教评特性不在 BEV 场景使用；倍速通过 `videoRef` 在 Effect 中同步到播放器。
- BEV 区分“没有视频”“有视频但不可播放”和“正在获取/播放失败”等状态；四字段填写与视频可播状态解耦。这里的 `key={playUrl}` 同样使新 URL 对应新播放器实例。

## 地址如何到达浏览器

1. 前端用 `catalogFileId` 请求播放地址接口，Hook 保存返回的 `url`。
2. 当前后端 `PlayUrlAppService` 检查目录文件存在且 `status=normal`、关联存储存在，再以存储的 `http_path` 和文件的 `relative_path` 构建 URL；目录文件与存储元数据有 60 秒进程内 TTL 缓存。
3. React 把 URL 传给 `<video src={url}>`；媒体内容由浏览器从该地址获取，不经过这个播放地址服务转发。

## 源码定位

- 通用弹窗：`KindVideo/frontend/src/components/shared/VideoPlaybackModal/index.tsx`
- 基础播放器：`KindVideo/frontend/src/components/shared/VideoPlaybackModal/components/VideoPlayer.tsx`
- 播放地址 Hook：`KindVideo/frontend/src/hooks/useVideoPlayback.ts`
- 教评播放器区：`KindVideo/frontend/src/pages/annotation/components/VideoPlayerSection.tsx`
- BEV 复用：`KindVideo/frontend/src/pages/bev-annotation/task.tsx`
- 后端地址构建：`KindVideo/backend/src/kindvideo/application/play_url_service.py`

# 相关链接

- [React 核心概念与渲染流程](react-fundamentals-and-rendering.md)
- [文件存储三原则：NAS 存文件、索引在库、直链播放](../backend/architecture/file-storage-nas-direct-url.md)
- [2026-09-27 学习日志](../../daily/2026-09-27.md)
