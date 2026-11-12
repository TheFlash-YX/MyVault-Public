---
type: interview
scope: resume
area: resume
project: kindvideo
resume_version: 2026-08
topics:
  - annotation
  - quality-loop
  - idempotency
  - api-key
  - authentication
created: 2026-08-27
updated: 2026-09-23
---

# KindVideo 项目答辩

- 原始题库只提供问题；回答框架只定义组织方式，不据此补造项目事实、实现细节或指标证据。
- 关联知识与题集：
- [标注探针机制：无 GT 场景的质量闭环](../../notes/backend/architecture/annotation-probe-quality-loop.md)
- [乐观锁：版本号 CAS 与并发取舍](../../notes/backend/database/optimistic-locking.md)
- [事务、锁与幂等](../backend/transactions-locking-and-idempotency.md)

## KindVideo 的 API-Key 鉴权是如何实现的？

<!-- canonical_key: resume.kindvideo.api-key-authentication -->

### 考察点

- API-Key 的生成、存储与请求校验
- 身份认证与路由授权的边界
- 个人负责的实现范围与源码证据

### 回答框架

- 当前源码中，已登录用户调用 `POST /api/v1/auth/api-key` 创建 Key；每个用户最多一个，默认有效期 90 天。服务端使用 `kv_` 前缀和 `secrets.token_urlsafe(32)` 生成随机 Key。
- 数据库记录 `user_id`、`key_hash`、`key_ciphertext`、`key_prefix`、`created_at`、`expires_at`、`last_used_at`、`revoked_at`。`key_hash` 是原始 Key 的 SHA-256 摘要，请求校验时用它查找记录；`key_ciphertext` 是 Fernet 加密值，供已认证用户查询自己现有的 Key 时解密返回。
- 调用方将 Key 放入 HTTP 请求头 `X-API-Key`。`SessionAuthMiddleware` 在请求进入路由前读取该头，对 Key 计算 SHA-256 并按哈希查库；随后检查 Key 是否存在、已撤销或过期，以及关联用户是否存在且有效。通过后更新 `last_used_at`，并将关联用户设置为当前请求的 `principal`。
- API-Key 负责识别调用者，不单独携带权限范围。路由继续通过 `require_authenticated` 或 `require_admin` 检查用户权限；具体业务用例还可按用户、任务或数据集做资源级校验。比如普教课堂目录导入接口要求关联用户是管理员。
- 鉴权实现证据：`KindVideo/backend/src/kindvideo/application/usecases/create_api_key_use_case.py`、`KindVideo/backend/src/kindvideo/application/services/api_key_crypto_service.py`、`KindVideo/backend/src/kindvideo/interfaces/rest/middleware/sso_auth_middleware.py`、`KindVideo/backend/src/kindvideo/interfaces/rest/dependencies/sso_auth.py`。
- 个人贡献只陈述本人实际负责的环节；当前源码可以核实系统行为，但不能单独证明代码归属。面试前确认自己负责的是 Key 管理接口、哈希/加密存储、中间件校验，还是路由/业务授权。

### 易追问

- 为什么同时存哈希和密文？哈希供请求校验且不可逆；密文供系统按用户读取并展示现有 Key。生产环境的 `API_KEY_ENCRYPTION_SECRET` 必须保密。
- `X-API-Key` 是请求的哪一部分？它是 HTTP 请求头字段；HTTPS/TLS 保护传输，不是请求头本身加密。
- Key 自己带了哪些权限？当前实现没有独立的 Key scope；Key 映射到用户，具体接口再检查用户角色和业务资源权限。
- Key 无效时会怎样？请求返回 401；如果请求带了 `X-API-Key`，中间件按 API-Key 分支处理，不回退到 Cookie 会话。

### 关联笔记

-

## 什么是标注探针？

<!-- canonical_key: resume.kindvideo.shen-me-shi-biao-zhu-tan-zhen -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么需要探针？

<!-- canonical_key: resume.kindvideo.wei-shen-me-xu-yao-tan-zhen -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## “可信样本”是怎么产生的？

<!-- canonical_key: resume.kindvideo.ke-xin-yang-ben-shi-zen-me-chan-sheng-de -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么要隐蔽注入？

<!-- canonical_key: resume.kindvideo.wei-shen-me-yao-yin-bi-zhu-ru -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 如果标注人员知道哪些是探针，会发生什么？

<!-- canonical_key: resume.kindvideo.ru-guo-biao-zhu-ren-yuan-zhi-dao-na-xie-shi-tan-zhen-hui-fa-sheng-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 探针的注入比例怎么确定？

<!-- canonical_key: resume.kindvideo.tan-zhen-de-zhu-ru-bi-li-zen-me-que-ding -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 探针是否会重复分配？

<!-- canonical_key: resume.kindvideo.tan-zhen-shi-fou-hui-zhong-fu-fen-pei -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 你怎么避免探针被过度使用？

<!-- canonical_key: resume.kindvideo.ni-zen-me-bi-mian-tan-zhen-bei-guo-du-shi-yong -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 什么叫“统一策略接口”？

<!-- canonical_key: resume.kindvideo.shen-me-jiao-tong-yi-ce-lue-jie-kou -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么不同 task_kind 需要不同匹配策略？

<!-- canonical_key: resume.kindvideo.wei-shen-me-bu-tong-task-kind-xu-yao-bu-tong-pi-pei-ce-lue -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 策略模式在这里是怎么应用的？

<!-- canonical_key: resume.kindvideo.ce-lue-mo-shi-zai-zhe-li-shi-zen-me-ying-yong-de -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 快照构建是什么？

<!-- canonical_key: resume.kindvideo.kuai-zhao-gou-jian-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么审核前需要快照？

<!-- canonical_key: resume.kindvideo.wei-shen-me-shen-he-qian-xu-yao-kuai-zhao -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 如果原始数据在审核过程中被修改怎么办？

<!-- canonical_key: resume.kindvideo.ru-guo-yuan-shi-shu-ju-zai-shen-he-guo-cheng-zhong-bei-xiu-gai-zen-me-ban -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 匹配阈值怎么配置？

<!-- canonical_key: resume.kindvideo.pi-pei-yu-zhi-zen-me-pei-zhi -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 不同任务的匹配算法分别是什么？

<!-- canonical_key: resume.kindvideo.bu-tong-ren-wu-de-pi-pei-suan-fa-fen-bie-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 自动审核逻辑是什么？

<!-- canonical_key: resume.kindvideo.zi-dong-shen-he-luo-ji-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 什么情况下转人工审核？

<!-- canonical_key: resume.kindvideo.shen-me-qing-kuang-xia-zhuan-ren-gong-shen-he -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 探针结果为什么要回流？

<!-- canonical_key: resume.kindvideo.tan-zhen-jie-guo-wei-shen-me-yao-hui-liu -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么回流必须幂等？

<!-- canonical_key: resume.kindvideo.wei-shen-me-hui-liu-bi-xu-mi-deng -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 幂等是怎么实现的？

<!-- canonical_key: resume.kindvideo.mi-deng-shi-zen-me-shi-xian-de -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 使用数据库唯一索引还是业务 key？

<!-- canonical_key: resume.kindvideo.shi-yong-shu-ju-ku-wei-yi-suo-yin-hai-shi-ye-wu-key -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 如果接口被重复调用 10 次，会发生什么？

<!-- canonical_key: resume.kindvideo.ru-guo-jie-kou-bei-zhong-fu-diao-yong-10-ci-hui-fa-sheng-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 7000+ 探针是怎么累计的？

<!-- canonical_key: resume.kindvideo.7000-tan-zhen-shi-zen-me-lei-ji-de -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 探针池会不会出现错误样本污染？

<!-- canonical_key: resume.kindvideo.tan-zhen-chi-hui-bu-hui-chu-xian-cuo-wu-yang-ben-wu-ran -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 如何检测探针本身的质量？

<!-- canonical_key: resume.kindvideo.ru-he-jian-ce-tan-zhen-ben-shen-de-zhi-liang -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 探针准确率如果下降怎么办？

<!-- canonical_key: resume.kindvideo.tan-zhen-zhun-que-lu-ru-guo-xia-jiang-zen-me-ban -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 什么叫困难样本？

<!-- canonical_key: resume.kindvideo.shen-me-jiao-kun-nan-yang-ben -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么单人无法裁定？

<!-- canonical_key: resume.kindvideo.wei-shen-me-dan-ren-wu-fa-cai-ding -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 多人投票机制如何设计？

<!-- canonical_key: resume.kindvideo.duo-ren-tou-piao-ji-zhi-ru-he-she-ji -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 投票结果如何计算？

<!-- canonical_key: resume.kindvideo.tou-piao-jie-guo-ru-he-ji-suan -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么需要管理员最终裁定？

<!-- canonical_key: resume.kindvideo.wei-shen-me-xu-yao-guan-li-yuan-zui-zhong-cai-ding -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## “派生状态机”是什么意思？

<!-- canonical_key: resume.kindvideo.pai-sheng-zhuang-tai-ji-shi-shen-me-yi-si -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 和显式 status 字段有什么区别？

<!-- canonical_key: resume.kindvideo.he-xian-shi-status-zi-duan-you-shen-me-qu-bie -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 状态有哪些？

<!-- canonical_key: resume.kindvideo.zhuang-tai-you-na-xie -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 状态迁移条件有哪些？

<!-- canonical_key: resume.kindvideo.zhuang-tai-qian-yi-tiao-jian-you-na-xie -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么采用可配置票数阈值？

<!-- canonical_key: resume.kindvideo.wei-shen-me-cai-yong-ke-pei-zhi-piao-shu-yu-zhi -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 两个人同时提交最后一票，会发生什么？

<!-- canonical_key: resume.kindvideo.liang-ge-ren-tong-shi-ti-jiao-zui-hou-yi-piao-hui-fa-sheng-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么需要乐观锁？

<!-- canonical_key: resume.kindvideo.wei-shen-me-xu-yao-le-guan-suo -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 乐观锁原理是什么？

<!-- canonical_key: resume.kindvideo.le-guan-suo-yuan-li-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 乐观锁和悲观锁区别是什么？

<!-- canonical_key: resume.kindvideo.le-guan-suo-he-bei-guan-suo-qu-bie-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 你的乐观锁具体怎么实现？

<!-- canonical_key: resume.kindvideo.ni-de-le-guan-suo-ju-ti-zen-me-shi-xian -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## version 字段更新失败以后怎么办？

<!-- canonical_key: resume.kindvideo.version-zi-duan-geng-xin-shi-bai-yi-hou-zen-me-ban -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 是否重试？

<!-- canonical_key: resume.kindvideo.shi-fou-zhong-shi -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 重试多少次？

<!-- canonical_key: resume.kindvideo.zhong-shi-duo-shao-ci -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 如果高并发下大量冲突怎么办？

<!-- canonical_key: resume.kindvideo.ru-guo-gao-bing-fa-xia-da-liang-chong-tu-zen-me-ban -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 什么情况下应该改用悲观锁？

<!-- canonical_key: resume.kindvideo.shen-me-qing-kuang-xia-ying-gai-gai-yong-bei-guan-suo -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 探针池“硬排除”是什么意思？

<!-- canonical_key: resume.kindvideo.tan-zhen-chi-ying-pai-chu-shi-shen-me-yi-si -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 为什么不能只在业务代码里过滤？

<!-- canonical_key: resume.kindvideo.wei-shen-me-bu-neng-zhi-zai-ye-wu-dai-ma-li-guo-lu -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 投票之后自动切下一题怎么实现？

<!-- canonical_key: resume.kindvideo.tou-piao-zhi-hou-zi-dong-qie-xia-yi-ti-zen-me-shi-xian -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 用户连续双击提交会不会重复投票？

<!-- canonical_key: resume.kindvideo.yong-hu-lian-xu-shuang-ji-ti-jiao-hui-bu-hui-zhong-fu-tou-piao -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 如何保证一个人只能投一次？

<!-- canonical_key: resume.kindvideo.ru-he-bao-zheng-yi-ge-ren-zhi-neng-tou-yi-ci -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 完成周期缩短 35% 是怎么算出来的？

<!-- canonical_key: resume.kindvideo.wan-cheng-zhou-qi-suo-duan-35-shi-zen-me-suan-chu-lai-de -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 这个指标有没有其他因素影响？

<!-- canonical_key: resume.kindvideo.zhe-ge-zhi-biao-you-mei-you-qi-ta-yin-su-ying-xiang -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 探针回流的幂等 key 到底是什么？

<!-- canonical_key: resume.kindvideo.tan-zhen-hui-liu-de-mi-deng-key-dao-di-shi-shen-me -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 困难样本状态是存储状态还是运行时派生？

<!-- canonical_key: resume.kindvideo.kun-nan-yang-ben-zhuang-tai-shi-cun-chu-zhuang-tai-hai-shi-yun-xing-shi-pai-sheng -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-

## 乐观锁的 version 字段具体在哪张表？

<!-- canonical_key: resume.kindvideo.le-guan-suo-de-version-zi-duan-ju-ti-zai-na-zhang-biao -->

### 考察点

- 真实实现、个人贡献、证据位置、设计取舍与已知边界

### 回答框架

- 只依据真实代码、日志和本人贡献回答；按“背景—选择—实现—结果—边界”组织，尚未核实的细节不补造。

### 易追问

-

### 关联笔记

-
