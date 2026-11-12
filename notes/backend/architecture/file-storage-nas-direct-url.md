---
type: Note
domain: backend
title: 文件存储三原则：NAS 存文件、索引在库、直链播放
aliases: [视频存储架构, NAS 存储, 直链播放]
description: 视频类平台存储设计——文件本体在 NAS 不过后端、数据库只存索引（storages+catalog_file）、播放走 HTTP 直链；路径越界防护与对账
tags: [nas, 存储架构, 视频平台, 直链, 路径穿越, 目录扫描, mysql, 数据库索引]
status: learning
created: 2026-08-17
updated: 2026-08-24
---

# 概述

- 核心设计："**文件在 NAS、索引在 MySQL、播放走 HTTP 直链**"——视频本体不落库、不过后端进程
- 好处：后端不扛视频带宽（标注页几十路并发拉视频不垮）；文件丢失可对账；访问有路径越界防护

# 详细内容

## 三层结构

- **物理层**：NAS，NFS 挂载到应用服务器（如 /mnt/nas）做文件系统操作；另有 HTTP base URL 做播放直链前缀
- **索引层**：`storages` 表记存储实例（mount_point + http_path）；`catalog_file` 记每个文件（storage_id + relative_path + path_hash 唯一约束防重复登记；name/size/duration/md5/status）
- **业务层**：数据集成员关联表（`dataset_catalog_file` 装散装视频、`dataset_lesson` 装课节——两种业务粒度）

## 入库靠扫描

- 扫描任务递归找挂载点下所有 .mp4，登记进 catalog_file；可选可播放性探测 + MD5 校验
- 两类扫描：discovery（新文件登记）+ check（存量对账，文件丢失标 status=missing）
- 任务表记进度（processed/total/current_scan_root），可断点续扫

## KindVideo 的数据库索引设计

- **物理文件幂等键**：`catalog_file` 用唯一索引 `uq_catalog_file_storage_path(storage_id, path_hash)` 防止同一存储中的同一路径被重复登记；仓储层先把 `relative_path` 计算成 SHA-256，再走该索引查询，避免直接索引最长 1000 字符的路径
- **文件列表与扫描查询**：`catalog_file` 有 `idx_catalog_file_status(status)`、`idx_catalog_file_created_at_id(file_created_at, id)`、`idx_catalog_file_storage_created_at_id(storage_id, file_created_at, id)`，分别服务状态过滤、全局排序分页和按存储排序分页
- **课堂资源幂等键**：`basic_ed_classroom` 用唯一索引 `uq_basic_ed_classroom_org_source(org_id, source_platform_id)` 判断同一学校、同一外部课堂是否已经导入；另有 `idx_basic_ed_classroom_org_id(org_id)` 支持按学校筛选
- **课堂视频查询**：`basic_ed_classroom_video` 用 `idx_basic_ed_classroom_video_classroom_channel(classroom_id, channel)` 支持按课堂读取并按通道组织视频；`catalog_file_id` 由 MySQL 外键配套索引
- **学校匹配**：`organizations.name` 和 `organizations.code` 都有唯一索引；NAS 课堂导入按学校名称精确复用组织时可直接命中 `name` 索引
- **分层去重**：物理文件靠 `(storage_id, path_hash)` 去重，业务课堂靠 `(org_id, source_platform_id)` 去重；同一视频文件可以被业务资源引用，但不会重复登记为物理文件
- **当前边界**：`basic_ed_classroom_video` 尚无 `(classroom_id, channel, part_index)` 数据库唯一约束，重复通道分片由创建用例在应用层校验；目录导入应复用该用例，不能绕过业务校验直接写表

## 播放：拼直链不转发

- 后端 `PlayUrlService` 用 `storages.http_path + catalog_file.relative_path` 拼出直链，前端 `<video>` 直连 NAS 前面的 HTTP 服务——**后端只产字符串**
- 列表页用批量接口：catalog_file 与 storage 两层缓存 + 批量 IN 查询，避免 N+1
- 为什么不代理流式播放：代理会把视频带宽打到应用服务器；直链让 NAS 的 HTTP 服务扛流量，代价是直链鉴权交给网关层（内网取舍）

## 安全与健壮性

- **路径越界防护**：所有文件访问把相对路径 resolve 后强制 `is_relative_to` 挂载点，越界抛异常——挡 `../../` 目录穿越
- 直链鉴权在 NAS 前面的 HTTP 服务/网关；平台自身 API 在 SSO 之后
- 索引与磁盘对账：status 字段支持 missing 标记与恢复

## 为什么不用 OSS/S3

- 文件源头是校内录播 NAS，扫描登记 = 纳管已有资产，零搬迁成本
- 若迁移：storages 表加存储类型 + PlayUrlService 换签名 URL——存储细节隔离在 infrastructure，domain 无感

# 相关链接

- [2026-08-17 学习日志](../../../daily/2026-08-17.md)
- [不引入 Redis/MQ 的架构取舍](cache-and-async-task-alternatives.md)
- [DDD 领域驱动设计：战略与战术两层](./ddd-domain-driven-design.md)
