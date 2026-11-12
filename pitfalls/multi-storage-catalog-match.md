---
type: Pitfall
domain: backend
title: 单 storage 假设（storages[0]）在多 storage 环境匹配不到 catalog 文件
description: import-from-dir 按 storages[0] 匹配 catalog_file，生产/联调有 3 个 storage，文件落在非首个 storage 时必报 400 MISSING_CATALOG_FILES
tags: [storage, nas, catalog-file, import, use-case]
created: 2026-08-24
updated: 2026-08-24
---

# 现象

- 普教课堂 NAS 目录导入（import-from-dir）在本地单 storage 下通过，但在生产/联调环境报 `400 MISSING_CATALOG_FILES`，提示文件缺失——明明文件已经扫描入库
- 报错可稳定复现：文件路径存在且已入 `catalog_file` 表，却匹配不到

# 排查过程

- 先怀疑扫描/入库环节漏文件，反复核对入库数量一致
- 查生产备份环境配置：该环境有 **3 个 storage**（`basic_ed`、`import`、…），而非本地的 1 个
- 看 `_match_catalog_files`：只查 `storages[0]` 这个 storage 下的文件路径，其余 storage 完全没查
- 文件恰好落在非首个 storage → 按路径匹配永远为空 → 报缺失

# 根因

- **单 storage 假设**：实现时按本地开发环境只有 1 个 storage 的惯例，硬编码 `storages[0]`；多 storage 环境下第 0 个 storage 与实际文件所在 storage 不一致
- 本地单 storage 环境无法暴露此问题，属于环境相关缺陷

# 解决方案

- `_match_catalog_files` 改为**遍历全部 storage**，按 `(storage, 相对路径)` 合并匹配，缺失路径统一收集后再上报
- 匹配逻辑与 storage 数量解耦，不再依赖"第 0 个 storage"
- 补一条多 storage 回归测试（两个 storage 各放文件，目标文件在第二个 storage）
- 教训：涉及环境差异（storage / 分区 / 多租户）的实现，先确认真实环境配置，不要拿单机惯例拍

# 相关链接

- [普教 NAS→KV 导入设计](../notes/backend/architecture/file-storage-nas-direct-url.md)
