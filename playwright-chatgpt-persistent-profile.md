# Playwright 控制 ChatGPT：持久化浏览器 Profile 与跨设备复现

## 概述

- Playwright 的 `launchPersistentContext(userDataDir, options)` 会把 Cookie、Local Storage 等浏览器状态写入指定 Profile，因此在同一台电脑上重复使用同一个 `userDataDir` 时，通常不需要每次重新登录 ChatGPT。
- 使用 `nodeRepl.tmpDir/myvault-review-profile` 只能保证当前临时环境中的复用；Codex 重启、临时目录清理或运行环境更换后，登录状态可能消失。
- 长期方案是把 Profile 放在仓库外的固定用户目录，并让每次 Playwright 启动都解析到同一路径。
- Git 只管理启动约定、操作步骤和安全规则，绝不管理浏览器 Profile、Cookie、密码或其他认证材料。
- 换电脑后需要在新电脑的独立 Profile 中手动登录一次；之后可以在该电脑上持续复用，但不能依靠 Git 跨设备复制登录状态。

## 仓库内容与本机状态的边界

- 应进入 Git：本文档、固定 Profile 的路径约定、Playwright 启动方式、排查步骤和安全规则。
- 不应进入 Git：`userDataDir` 的实际目录、Cookie、Local Storage、浏览器缓存、下载记录、密码和站点会话令牌。
- Profile 必须位于 MyVault 仓库外，避免误执行 `git add` 时泄漏登录凭据。
- 不通过 iCloud、网盘或 Git 同步 Profile；浏览器数据包含认证令牌，而且跨操作系统、Chrome 版本和机器复制后也不保证可用。

## 固定 Profile 路径约定

- Playwright MCP/Node REPL 环境优先从 `nodeRepl.homeDir` 计算用户目录，不硬编码某台电脑的用户名。
- 统一使用以下逻辑路径：

```text
<用户主目录>/.myvault/playwright-profiles/chatgpt
```

- 在支持 Node 路径 API 的 Playwright MCP 中可这样计算：

```javascript
const path = await import("node:path");
const profileDir = path.join(
  nodeRepl.homeDir,
  ".myvault",
  "playwright-profiles",
  "chatgpt"
);
```

- 不再使用 `/tmp/...`、`nodeRepl.tmpDir/...` 或某次安装产生的随机目录保存长期 Profile。

## Playwright 启动约定

- 下面代码假设当前 MCP 会话已经取得 Playwright 对象；具体加载方式由所在电脑的 Playwright MCP 服务负责。

```javascript
const context = await playwright.chromium.launchPersistentContext(profileDir, {
  channel: "chrome",
  headless: false,
  viewport: { width: 1440, height: 1000 }
});

const page = context.pages()[0] ?? await context.newPage();
await page.goto("https://chatgpt.com/", {
  waitUntil: "domcontentloaded",
  timeout: 30000
});
```

- `launchPersistentContext` 的第一个参数必须始终是上述固定 `profileDir`；使用普通 `chromium.launch()` 或每次更换目录都会失去持久化效果。
- `channel: "chrome"` 表示使用机器上已安装的 Chrome；如果目标电脑没有 Chrome，需要先安装或改成当地可用的 Playwright 浏览器。
- Playwright 包或浏览器驱动可以按机器安装，不要把一次性模块路径（例如 `/tmp/.../node_modules`）写入仓库配置。
- 同一 Profile 同一时间只交给一个浏览器进程使用，否则可能遇到 Profile Lock、状态覆盖或数据损坏。
- 操作结束时正常关闭 `context`；不要通过强杀进程作为日常退出方式，以便 Chrome 完整写回会话状态。

## 新电脑复现流程

- 克隆 MyVault 仓库并读取本文档。
- 安装 Chrome 和可用的 Playwright MCP 服务；Playwright 依赖安装在本机环境，不提交到 MyVault。
- 按固定路径约定创建或首次启动 Profile，确保没有落到临时目录。
- 打开 `https://chatgpt.com/`，由用户亲自在浏览器窗口中完成登录和可能的二次验证。
- 登录完成后关闭并重新启动一次，确认仍然处于登录状态，才算持久化配置验收通过。
- 后续所有 ChatGPT 网页操作都复用同一个 Profile，并在执行前确认没有另一个实例占用它。
- 每台新电脑原则上只需首次登录一次；ChatGPT 主动撤销会话、密码变更、长时间未使用或安全风控仍可能要求重新登录。

## Agent 操作安全规则

- Agent 可以通过页面可见状态判断“已登录/未登录”，但不得读取、导出、打印或分析 Cookie、Local Storage、密码库和浏览器 Profile 文件。
- 登录、验证码、二次验证和账号恢复必须由用户在浏览器窗口中亲自完成。
- Agent 不把认证信息发送到聊天、日志、命令输出或外部服务。
- Agent 不直接复用日常 Chrome 主 Profile；使用独立自动化 Profile 可以降低锁冲突和误操作个人浏览数据的风险。
- 在发送消息、删除对话或执行其他外部副作用前，仍需以用户当前授权的具体任务为边界；持久化登录不等于获得无限操作授权。

## 常见问题排查

- 每次都要求登录：优先检查启动时的 `profileDir` 是否变化，以及是否错误使用了临时目录或普通非持久化 Context。
- 报 Profile 正在使用：关闭占用同一路径的旧 Playwright/Chrome 实例，再重试；不要复制正在使用中的 Profile。
- 登录一段时间后失效：可能是站点会话过期、密码变更或安全策略触发，不应通过导入 Cookie 绕过，直接由用户重新登录。
- 换电脑后没有登录状态：这是预期行为；Git 只同步方案，不同步认证材料，在新电脑手动登录一次即可。
- Playwright 模块路径失效：重新安装或连接目标电脑的 Playwright MCP 服务，但继续复用固定 `profileDir`，登录状态与模块安装目录相互独立。
