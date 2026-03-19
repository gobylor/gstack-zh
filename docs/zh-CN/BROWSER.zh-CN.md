> 本文档为 [BROWSER.md](../../BROWSER.md) 的中文翻译版本。

# Browser（浏览器）— 技术细节

本文档涵盖 gstack 无头浏览器的命令参考和内部实现。

## 命令参考

| 分类 | 命令 | 用途 |
|------|------|------|
| 导航 | `goto`, `back`, `forward`, `reload`, `url` | 跳转到某个页面 |
| 读取 | `text`, `html`, `links`, `forms`, `accessibility` | 提取页面内容 |
| 快照（Snapshot） | `snapshot [-i] [-c] [-d N] [-s sel] [-D] [-a] [-o] [-C]` | 获取引用（ref）、对比差异、添加注释 |
| 交互 | `click`, `fill`, `select`, `hover`, `type`, `press`, `scroll`, `wait`, `viewport`, `upload` | 操作页面 |
| 检查 | `js`, `eval`, `css`, `attrs`, `is`, `console`, `network`, `dialog`, `cookies`, `storage`, `perf` | 调试与验证 |
| 可视化 | `screenshot [--viewport] [--clip x,y,w,h] [sel\|@ref] [path]`, `pdf`, `responsive` | 查看 Claude 所看到的内容 |
| 对比 | `diff <url1> <url2>` | 发现不同环境之间的差异 |
| 对话框（Dialog） | `dialog-accept [text]`, `dialog-dismiss` | 控制 alert/confirm/prompt 的处理方式 |
| 标签页 | `tabs`, `tab`, `newtab`, `closetab` | 多页面工作流 |
| Cookie | `cookie-import`, `cookie-import-browser` | 从文件或真实浏览器导入 Cookie |
| 批量执行 | `chain`（从 stdin 读取 JSON） | 在一次调用中批量执行命令 |
| 移交（Handoff） | `handoff [reason]`, `resume` | 切换到可见的 Chrome 供用户接管 |

所有选择器参数均支持 CSS 选择器、`snapshot` 后产生的 `@e` 引用，以及 `snapshot -C` 后产生的 `@c` 引用。共 50+ 条命令，另外支持 Cookie 导入。

## 工作原理

gstack 的浏览器是一个编译好的 CLI 二进制文件，通过 HTTP 与持久运行的本地 Chromium 守护进程通信。CLI 是一个轻量客户端 — 它读取状态文件、发送命令、并将响应打印到 stdout。服务端通过 [Playwright](https://playwright.dev/) 完成实际工作。

```
┌─────────────────────────────────────────────────────────────────┐
│  Claude Code                                                    │
│                                                                 │
│  "browse goto https://staging.myapp.com"                        │
│       │                                                         │
│       ▼                                                         │
│  ┌──────────┐    HTTP POST     ┌──────────────┐                 │
│  │ browse   │ ──────────────── │ Bun HTTP     │                 │
│  │ CLI      │  localhost:rand  │ server       │                 │
│  │          │  Bearer token    │              │                 │
│  │ compiled │ ◄──────────────  │  Playwright  │──── Chromium    │
│  │ binary   │  plain text      │  API calls   │    (headless)   │
│  └──────────┘                  └──────────────┘                 │
│   ~1ms startup                  persistent daemon               │
│                                 auto-starts on first call       │
│                                 auto-stops after 30 min idle    │
└─────────────────────────────────────────────────────────────────┘
```

### 生命周期

1. **首次调用**：CLI 在项目根目录检查 `.gstack/browse.json`，查找是否有正在运行的服务器。若无，则在后台启动 `bun run browse/src/server.ts`。服务器通过 Playwright 启动无头 Chromium，随机选取一个端口（10000-60000），生成 bearer token（认证令牌），写入状态文件，并开始接受 HTTP 请求。此过程约需 3 秒。

2. **后续调用**：CLI 读取状态文件，携带 bearer token 发送 HTTP POST 请求，打印响应。往返时间约 100-200ms。

3. **空闲关闭**：若 30 分钟内无任何命令，服务器将自动关闭并清理状态文件。下次调用时会自动重启。

4. **崩溃恢复**：若 Chromium 崩溃，服务器立即退出（不自愈 — 不隐藏故障）。CLI 在下次调用时检测到服务器已停止，并自动启动一个新实例。

### 核心组件

```
browse/
├── src/
│   ├── cli.ts              # 轻量客户端 — 读取状态文件，发送 HTTP，打印响应
│   ├── server.ts           # Bun.serve HTTP 服务器 — 将命令路由到 Playwright 处理函数
│   ├── browser-manager.ts  # Chromium 生命周期 — 启动、标签页管理、ref 映射、崩溃处理
│   ├── snapshot.ts         # 无障碍树 → @ref 分配 → Locator 映射 + diff/annotate/-C
│   ├── read-commands.ts    # 非变更命令（text、html、links、js、css、is、dialog 等）
│   ├── write-commands.ts   # 变更命令（click、fill、select、upload、dialog-accept 等）
│   ├── meta-commands.ts    # 服务器管理、chain、diff、snapshot 路由
│   ├── cookie-import-browser.ts  # 从真实 Chromium 浏览器解密并导入 Cookie
│   ├── cookie-picker-routes.ts   # 交互式 Cookie 选择器 UI 的 HTTP 路由
│   ├── cookie-picker-ui.ts       # Cookie 选择器的自包含 HTML/CSS/JS
│   └── buffers.ts          # CircularBuffer<T> + console/network/dialog 捕获
├── test/                   # 集成测试 + HTML 测试固件（fixture）
└── dist/
    └── browse              # 编译后的二进制文件（约 58MB，使用 Bun --compile）
```

### 快照（Snapshot）系统

浏览器的核心创新是基于 ref（引用）的元素选择，构建在 Playwright 的无障碍树（accessibility tree）API 之上：

1. `page.locator(scope).ariaSnapshot()` 返回类 YAML 格式的无障碍树
2. 快照解析器为每个元素分配 ref（`@e1`、`@e2`、……）
3. 针对每个 ref，使用 `getByRole` + nth-child 构建一个 Playwright `Locator`
4. ref 到 Locator 的映射存储在 `BrowserManager` 上
5. 后续命令如 `click @e3` 查找对应 Locator 并调用 `locator.click()`

无 DOM 变更。无注入脚本。仅使用 Playwright 的原生无障碍 API。

**Ref 过期检测：** 单页应用（SPA）可能在不触发页面导航的情况下修改 DOM（如 React 路由切换、tab 切换、弹窗）。此时，从之前 `snapshot` 收集的 ref 可能指向已不存在的元素。为此，`resolveRef()` 在使用任何 ref 之前会先执行异步 `count()` 检查 — 若元素数量为 0，立即抛出错误并提示 agent 重新执行 `snapshot`。这种快速失败机制耗时约 5ms，远优于等待 Playwright 30 秒的操作超时。

**扩展快照功能：**
- `--diff`（`-D`）：将每次快照存为基线（baseline）。下次带 `-D` 调用时，返回显示变更内容的统一差异（unified diff）。用于验证某个操作（click、fill 等）是否真正生效。
- `--annotate`（`-a`）：在每个 ref 的边界框处注入临时覆盖层 div，截图时显示 ref 标签，完成后移除覆盖层。使用 `-o <path>` 指定输出路径。
- `--cursor-interactive`（`-C`）：通过 `page.evaluate` 扫描非 ARIA 的可交互元素（带有 `cursor:pointer`、`onclick`、`tabindex>=0` 的 div），分配 `@c1`、`@c2`……等 ref，使用确定性的 `nth-child` CSS 选择器。这些是 ARIA 树遗漏但用户仍可点击的元素。

### 截图（Screenshot）模式

`screenshot` 命令支持四种模式：

| 模式 | 语法 | Playwright API |
|------|------|----------------|
| 全页（默认） | `screenshot [path]` | `page.screenshot({ fullPage: true })` |
| 仅视口 | `screenshot --viewport [path]` | `page.screenshot({ fullPage: false })` |
| 元素裁剪 | `screenshot "#sel" [path]` 或 `screenshot @e3 [path]` | `locator.screenshot()` |
| 区域裁剪 | `screenshot --clip x,y,w,h [path]` | `page.screenshot({ clip })` |

元素裁剪支持 CSS 选择器（`.class`、`#id`、`[attr]`）或 `snapshot` 产生的 `@e`/`@c` ref。自动识别规则：`@e`/`@c` 前缀 = ref，`.`/`#`/`[` 前缀 = CSS 选择器，`--` 前缀 = 标志参数，其他 = 输出路径。

互斥限制：`--clip` + 选择器，以及 `--viewport` + `--clip` 同时使用均会报错。未知标志（如 `--bogus`）同样报错。

### 认证（Authentication）

每个服务器会话生成一个随机 UUID 作为 bearer token。该 token 以 chmod 600 权限写入状态文件（`.gstack/browse.json`）。每个 HTTP 请求必须携带 `Authorization: Bearer <token>`，以防止机器上的其他进程控制浏览器。

### Console、网络与对话框捕获

服务器监听 Playwright 的 `page.on('console')`、`page.on('response')` 和 `page.on('dialog')` 事件。所有条目保存在 O(1) 环形缓冲区（循环缓冲区，容量各 50,000 条）中，并通过 `Bun.write()` 异步刷入磁盘：

- Console 日志：`.gstack/browse-console.log`
- 网络日志：`.gstack/browse-network.log`
- 对话框日志：`.gstack/browse-dialog.log`

`console`、`network`、`dialog` 命令从内存缓冲区读取数据，而非磁盘文件。

### 用户移交（User Handoff）

当无头浏览器无法继续（如遇到验证码 CAPTCHA、多因素认证 MFA、复杂登录流程），`handoff` 命令会打开一个可见的 Chrome 窗口，保留所有 Cookie、localStorage 和标签页，供用户手动操作。用户处理完毕后，`resume` 命令将控制权交还给 agent，并提供最新快照。

```bash
$B handoff "Stuck on CAPTCHA at login page"   # 打开可见的 Chrome 窗口
# 用户完成验证码...
$B resume                                       # 切换回无头模式并获取最新快照
```

连续失败 3 次后，浏览器会自动建议执行 `handoff`。切换过程中状态完整保留，无需重新登录。

### 对话框（Dialog）处理

对话框（alert、confirm、prompt）默认自动接受，以防止浏览器卡死。`dialog-accept` 和 `dialog-dismiss` 命令可控制此行为。对于 prompt 类型，`dialog-accept <text>` 提供响应文本。所有对话框均记录到 dialog 缓冲区，包含类型、消息内容和处理动作。

### JavaScript 执行（`js` 与 `eval`）

`js` 执行单个表达式，`eval` 执行一个 JS 文件。两者均支持 `await` — 包含 `await` 的表达式会自动包裹在异步上下文中：

```bash
$B js "await fetch('/api/data').then(r => r.json())"  # 正常工作
$B js "document.title"                                  # 同样正常（无需包裹）
$B eval my-script.js                                    # 含 await 的文件同样可用
```

对于 `eval` 文件，单行文件直接返回表达式的值。多行文件在使用 `await` 时需要显式 `return`。注释中含有 "await" 不会触发异步包裹。

### 多工作区支持

每个工作区（workspace）拥有独立的浏览器实例，包含各自的 Chromium 进程、标签页、Cookie 和日志。状态存储在项目根目录（通过 `git rev-parse --show-toplevel` 检测）下的 `.gstack/` 中。

| 工作区 | 状态文件 | 端口 |
|--------|----------|------|
| `/code/project-a` | `/code/project-a/.gstack/browse.json` | 随机（10000-60000） |
| `/code/project-b` | `/code/project-b/.gstack/browse.json` | 随机（10000-60000） |

无端口冲突。无共享状态。每个项目完全隔离。

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `BROWSE_PORT` | 0（随机 10000-60000） | HTTP 服务器的固定端口（调试用覆盖选项） |
| `BROWSE_IDLE_TIMEOUT` | 1800000（30 分钟） | 空闲关闭超时时间（毫秒） |
| `BROWSE_STATE_FILE` | `.gstack/browse.json` | 状态文件路径（CLI 传给服务器） |
| `BROWSE_SERVER_SCRIPT` | 自动检测 | server.ts 的路径 |

### 性能

| 工具 | 首次调用 | 后续调用 | 每次调用的上下文开销 |
|------|---------|---------|---------------------|
| Chrome MCP | ~5s | ~2-5s | ~2000 tokens（schema + 协议） |
| Playwright MCP | ~3s | ~1-3s | ~1500 tokens（schema + 协议） |
| **gstack browse** | **~3s** | **~100-200ms** | **0 tokens**（纯文本 stdout） |

上下文开销的差距会快速累积。在一个包含 20 条命令的浏览器会话中，MCP 工具仅协议封装就会消耗 30,000-40,000 tokens，而 gstack 消耗为零。

### 为什么选择 CLI 而非 MCP？

MCP（Model Context Protocol，模型上下文协议）在远程服务场景下运行良好，但对于本地浏览器自动化而言只会带来纯粹的额外开销：

- **上下文膨胀**：每次 MCP 调用都包含完整的 JSON schema 和协议封装。一个简单的"获取页面文本"操作消耗的上下文 token 数是应有量的 10 倍。
- **连接脆弱性**：持久的 WebSocket/stdio 连接容易断开且难以重连。
- **不必要的抽象层**：Claude Code 已内置 Bash 工具。打印到 stdout 的 CLI 是最简单的接口形式。

gstack 跳过了所有这些。编译后的二进制文件。纯文本输入，纯文本输出。无协议。无 schema。无连接管理。

## 致谢

浏览器自动化层基于微软的 [Playwright](https://playwright.dev/) 构建。Playwright 的无障碍树 API、Locator 系统以及无头 Chromium 管理，使基于 ref 的交互成为可能。快照系统 — 为无障碍树节点分配 `@ref` 标签并将其映射回 Playwright Locator — 完全构建于 Playwright 的原语之上。感谢 Playwright 团队构建了如此坚实的基础。

## 开发

### 前置条件

- [Bun](https://bun.sh/) v1.0+
- Playwright 的 Chromium（运行 `bun install` 时自动安装）

### 快速开始

```bash
bun install              # 安装依赖 + Playwright Chromium
bun test                 # 运行集成测试（约 3 秒）
bun run dev <cmd>        # 从源码运行 CLI（无需编译）
bun run build            # 编译到 browse/dist/browse
```

### 开发模式与编译二进制

开发期间请使用 `bun run dev` 而非编译后的二进制文件。它直接用 Bun 运行 `browse/src/cli.ts`，无需编译即可立即看到效果：

```bash
bun run dev goto https://example.com
bun run dev text
bun run dev snapshot -i
bun run dev click @e3
```

编译后的二进制文件（`bun run build`）仅用于发布分发。使用 Bun 的 `--compile` 标志，在 `browse/dist/browse` 生成约 58MB 的单一可执行文件。

### 运行测试

```bash
bun test                         # 运行所有测试
bun test browse/test/commands              # 仅运行命令集成测试
bun test browse/test/snapshot              # 仅运行快照测试
bun test browse/test/cookie-import-browser # 仅运行 Cookie 导入单元测试
```

测试会启动一个本地 HTTP 服务器（`browse/test/test-server.ts`），提供来自 `browse/test/fixtures/` 的 HTML 测试固件，然后对这些页面执行 CLI 命令。共 3 个文件，203 条测试，总耗时约 15 秒。

### 源码结构

| 文件 | 职责 |
|------|------|
| `browse/src/cli.ts` | 入口点。读取 `.gstack/browse.json`，向服务器发送 HTTP，打印响应。 |
| `browse/src/server.ts` | Bun HTTP 服务器。将命令路由到对应处理函数，管理空闲超时。 |
| `browse/src/browser-manager.ts` | Chromium 生命周期 — 启动、标签页管理、ref 映射、崩溃检测。 |
| `browse/src/snapshot.ts` | 解析无障碍树，分配 `@e`/`@c` ref，构建 Locator 映射。处理 `--diff`、`--annotate`、`-C`。 |
| `browse/src/read-commands.ts` | 非变更命令：`text`、`html`、`links`、`js`、`css`、`is`、`dialog`、`forms` 等。导出 `getCleanText()`。 |
| `browse/src/write-commands.ts` | 变更命令：`goto`、`click`、`fill`、`upload`、`dialog-accept`、`useragent`（含上下文重建）等。 |
| `browse/src/meta-commands.ts` | 服务器管理、chain 路由、diff（通过 `getCleanText` 复用）、snapshot 委托。 |
| `browse/src/cookie-import-browser.ts` | 通过 macOS Keychain + PBKDF2/AES-128-CBC 解密 Chromium Cookie，自动检测已安装的浏览器。 |
| `browse/src/cookie-picker-routes.ts` | `/cookie-picker/*` 的 HTTP 路由 — 浏览器列表、域名搜索、导入、移除。 |
| `browse/src/cookie-picker-ui.ts` | 交互式 Cookie 选择器的自包含 HTML 生成器（深色主题，无框架依赖）。 |
| `browse/src/buffers.ts` | `CircularBuffer<T>`（O(1) 环形缓冲区）+ console/network/dialog 捕获，支持异步磁盘刷写。 |

### 部署到活跃技能（Active Skill）

活跃技能位于 `~/.claude/skills/gstack/`。修改后执行以下步骤：

1. 推送你的分支
2. 在技能目录中拉取更新：`cd ~/.claude/skills/gstack && git pull`
3. 重新构建：`cd ~/.claude/skills/gstack && bun run build`

或直接复制二进制文件：`cp browse/dist/browse ~/.claude/skills/gstack/browse/dist/browse`

### 添加新命令

1. 在 `read-commands.ts`（非变更命令）或 `write-commands.ts`（变更命令）中添加处理函数
2. 在 `server.ts` 中注册路由
3. 如需 HTML 测试固件，在 `browse/test/commands.test.ts` 中添加测试用例
4. 运行 `bun test` 验证
5. 运行 `bun run build` 编译
