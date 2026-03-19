> 本文档为 [ARCHITECTURE.md](../../ARCHITECTURE.md) 的中文翻译版本。

# 架构

本文档解释 gstack **为何**如此设计。关于安装和命令，请参见 CLAUDE.md；关于贡献指南，请参见 CONTRIBUTING.md。

## 核心思想

gstack 为 Claude Code 提供了一个持久化浏览器和一套有主见的工作流技能（skill）。浏览器是难点——其余部分不过是 Markdown。

关键洞察：AI 智能体（agent）与浏览器交互时，需要**亚秒级延迟**和**持久状态**。若每条命令都冷启动一个浏览器，每次工具调用（tool call）就要等待 3-5 秒。若命令之间浏览器会关闭，Cookie、标签页和登录会话就会全部丢失。因此 gstack 运行一个长生命周期的 Chromium 守护进程（daemon），CLI 通过本地 HTTP 与其通信。

```
Claude Code                     gstack
─────────                      ──────
                               ┌──────────────────────┐
  Tool call: $B snapshot -i    │  CLI (compiled binary)│
  ─────────────────────────→   │  • reads state file   │
                               │  • POST /command      │
                               │    to localhost:PORT   │
                               └──────────┬───────────┘
                                          │ HTTP
                               ┌──────────▼───────────┐
                               │  Server (Bun.serve)   │
                               │  • dispatches command  │
                               │  • talks to Chromium   │
                               │  • returns plain text  │
                               └──────────┬───────────┘
                                          │ CDP
                               ┌──────────▼───────────┐
                               │  Chromium (headless)   │
                               │  • persistent tabs     │
                               │  • cookies carry over  │
                               │  • 30min idle timeout  │
                               └───────────────────────┘
```

第一次调用会启动所有组件（约 3 秒）。之后每次调用约 100-200 毫秒。

## 为什么选择 Bun

Node.js 也可以用，但 Bun 在这里有三点优势：

1. **编译二进制。** `bun build --compile` 生成单个约 58MB 的可执行文件。运行时无需 `node_modules`、无需 `npx`、无需配置 PATH。二进制文件直接运行。这很重要，因为 gstack 安装在 `~/.claude/skills/` 下，用户不希望在那里管理 Node.js 项目。

2. **原生 SQLite。** Cookie 解密需要直接读取 Chromium 的 SQLite Cookie 数据库。Bun 内置了 `new Database()` —— 无需 `better-sqlite3`，无需编译原生插件，无需 gyp。减少了一个在不同机器上容易出问题的依赖。

3. **原生 TypeScript。** 开发时服务器以 `bun run server.ts` 运行，无需编译步骤、无需 `ts-node`、无需调试 source map。编译后的二进制用于部署，源文件用于开发。

4. **内置 HTTP 服务器。** `Bun.serve()` 快速、简洁，无需 Express 或 Fastify。服务器共处理约 10 条路由，引入框架纯属开销。

瓶颈始终是 Chromium，而非 CLI 或服务器。Bun 的启动速度（编译后二进制约 1ms，Node.js 约 100ms）是加分项，但不是选择它的原因。编译二进制和原生 SQLite 才是。

## 守护进程（daemon）模型

### 为什么不为每条命令启动一个浏览器？

Playwright 启动 Chromium 约需 2-3 秒。对单次截图来说可以接受，但对包含 20+ 条命令的 QA 会话来说，就是 40+ 秒的浏览器启动开销。更糟的是：命令之间所有状态都会丢失——Cookie、localStorage、登录会话、已打开的标签页，全部清空。

守护进程模型带来的好处：

- **持久状态。** 登录一次，保持登录。打开标签页，保持打开。localStorage 跨命令持久化。
- **亚秒级命令。** 第一次调用后，每条命令只是一次 HTTP POST，含 Chromium 处理的往返时间约 100-200ms。
- **自动生命周期管理。** 服务器在首次使用时自动启动，空闲 30 分钟后自动关闭，无需手动管理进程。

### 状态文件

服务器写入 `.gstack/browse.json`（通过临时文件 + 重命名实现原子写入，权限 0o600）：

```json
{ "pid": 12345, "port": 34567, "token": "uuid-v4", "startedAt": "...", "binaryVersion": "abc123" }
```

CLI 读取此文件来定位服务器。若文件不存在、已过期，或 PID 对应进程已死，CLI 会启动一个新服务器。

### 端口选择

在 10000-60000 之间随机选取端口（冲突时最多重试 5 次）。这意味着 10 个 Conductor 工作区可以各自运行独立的 browse 守护进程，无需任何配置，端口零冲突。旧方案（扫描 9400-9409）在多工作区环境下频繁出问题。

### 版本自动重启

构建时会将 `git rev-parse HEAD` 写入 `browse/dist/.version`。每次 CLI 调用时，若二进制的版本与正在运行的服务器的 `binaryVersion` 不匹配，CLI 会杀掉旧服务器并启动新服务器。这从根本上消除了"二进制文件过期"这类 bug —— 重新构建二进制后，下一条命令会自动切换到新版本。

## 安全模型

### 仅限本地访问

HTTP 服务器绑定到 `localhost`，而非 `0.0.0.0`，无法从网络访问。

### Bearer Token 认证

每次服务器会话都会生成一个随机 UUID token，写入权限为 0o600（仅所有者可读）的状态文件。每个 HTTP 请求都必须携带 `Authorization: Bearer <token>`。若 token 不匹配，服务器返回 401。

这防止了同一机器上的其他进程与你的 browse 服务器通信。Cookie 选择器 UI（`/cookie-picker`）和健康检查（`/health`）豁免于此——它们仅限本地访问且不执行命令。

### Cookie 安全

Cookie 是 gstack 处理的最敏感数据，设计如下：

1. **访问 Keychain（密钥链）需要用户授权。** 每个浏览器第一次导入 Cookie 时会触发 macOS Keychain 对话框，用户必须点击"允许"或"始终允许"。gstack 从不静默访问凭据。

2. **解密在进程内完成。** Cookie 值在内存中解密（PBKDF2 + AES-128-CBC），加载到 Playwright 上下文，绝不以明文形式写入磁盘。Cookie 选择器 UI 从不显示 Cookie 值——只显示域名和数量。

3. **数据库以只读方式访问。** gstack 将 Chromium Cookie 数据库复制到临时文件（以避免与正在运行的浏览器产生 SQLite 锁冲突），并以只读方式打开。它永远不会修改真实浏览器的 Cookie 数据库。

4. **密钥缓存仅限当次会话。** Keychain 密码和派生的 AES 密钥在内存中缓存，仅在服务器生命周期内有效。服务器关闭（空闲超时或显式停止）后，缓存即清除。

5. **日志中不含 Cookie 值。** 控制台、网络和对话框日志中从不包含 Cookie 值。`cookies` 命令输出 Cookie 元数据（域名、名称、过期时间），但值会被截断。

### Shell 注入防护

浏览器注册表（Comet、Chrome、Arc、Brave、Edge）是硬编码的。数据库路径由已知常量构造，从不来自用户输入。Keychain 访问使用 `Bun.spawn()` 并传入显式参数数组，而非 shell 字符串拼接。

## ref（引用）系统

ref（`@e1`、`@e2`、`@c1`）是智能体寻址页面元素的方式，无需编写 CSS 选择器或 XPath。

### 工作原理

```
1. 智能体运行：$B snapshot -i
2. 服务器调用 Playwright 的 page.accessibility.snapshot()
3. 解析器遍历 ARIA 树，按顺序分配 ref：@e1, @e2, @e3...
4. 为每个 ref 构建 Playwright Locator：getByRole(role, { name }).nth(index)
5. 在 BrowserManager 实例上存储 Map<string, RefEntry>（role + name + Locator）
6. 将带注释的树以纯文本形式返回

之后：
7. 智能体运行：$B click @e3
8. 服务器解析 @e3 → Locator → locator.click()
```

### 为什么用 Locator，而非 DOM 注入

最直接的方案是向 DOM 注入 `data-ref="@e1"` 属性。但这在以下情况会失效：

- **CSP（内容安全策略）。** 许多生产站点会阻止脚本修改 DOM。
- **React/Vue/Svelte 水合（hydration）。** 框架协调（reconciliation）可能会剥离注入的属性。
- **Shadow DOM。** 无法从外部访问 shadow root 内部。

Playwright Locator 外置于 DOM，使用无障碍树（Chromium 在内部维护）和 `getByRole()` 查询，无需 DOM 注入，无 CSP 问题，无框架冲突。

### ref 生命周期

ref 在导航时清除（主框架的 `framenavigated` 事件）。这是正确的——导航后所有 locator 都已过期。智能体必须重新运行 `snapshot` 以获取新 ref。这是有意为之：过期的 ref 应该立即报错，而不是点击错误的元素。

### ref 过期检测

SPA（单页应用）可能在不触发 `framenavigated` 的情况下变更 DOM（例如 React Router 路由切换、标签页切换、模态框打开）。这会导致 ref 过期，即使页面 URL 没有变化。为捕获这种情况，`resolveRef()` 在使用任何 ref 前会执行异步 `count()` 检查：

```
resolveRef(@e3) → entry = refMap.get("e3")
                → count = await entry.locator.count()
                → if count === 0: throw "Ref @e3 is stale — element no longer exists. Run 'snapshot' to get fresh refs."
                → if count > 0: return { locator }
```

这会快速失败（约 5ms 开销），而非等待 Playwright 30 秒的操作超时。`RefEntry` 在 Locator 旁边存储了 `role` 和 `name` 元数据，让错误信息能告知智能体该元素原本是什么。

### 游标交互 ref（@c）

`-C` 标志用于查找可点击但不在 ARIA 树中的元素——那些设置了 `cursor: pointer` 样式、带有 `onclick` 属性或自定义 `tabindex` 的元素。这些元素在独立命名空间中获得 `@c1`、`@c2` 等 ref。这能捕获框架渲染为 `<div>` 但实际上是按钮的自定义组件。

## 日志架构

三个环形缓冲区（ring buffer）（每个 50,000 条，O(1) 写入）：

```
Browser events → CircularBuffer (in-memory) → Async flush to .gstack/*.log
```

控制台消息、网络请求和对话框事件各有独立的缓冲区。每 1 秒刷新一次——服务器只追加自上次刷新以来的新条目。这意味着：

- HTTP 请求处理从不被磁盘 I/O 阻塞
- 日志在服务器崩溃后仍可保存（最多丢失 1 秒的数据）
- 内存占用有界（50K 条 × 3 个缓冲区）
- 磁盘文件仅追加，可被外部工具读取

`console`、`network` 和 `dialog` 命令从内存缓冲区读取，而非磁盘。磁盘文件用于事后（post-mortem）调试。

## SKILL.md 模板系统

### 问题

SKILL.md 文件告诉 Claude 如何使用 browse 命令。若文档列出了不存在的标志，或遗漏了新增的命令，智能体就会报错。手动维护的文档总会与代码产生偏差。

### 解决方案

```
SKILL.md.tmpl          （人工编写的正文 + 占位符）
       ↓
gen-skill-docs.ts      （读取源码元数据）
       ↓
SKILL.md               （已提交，自动生成的部分）
```

模板包含需要人工判断的工作流、技巧和示例，占位符在构建时从源码填充：

| 占位符 | 来源 | 生成内容 |
|--------|------|----------|
| `{{COMMAND_REFERENCE}}` | `commands.ts` | 分类命令表 |
| `{{SNAPSHOT_FLAGS}}` | `snapshot.ts` | 带示例的标志参考 |
| `{{PREAMBLE}}` | `gen-skill-docs.ts` | 启动块：更新检查、会话追踪、贡献者模式、AskUserQuestion 格式 |
| `{{BROWSE_SETUP}}` | `gen-skill-docs.ts` | 二进制发现 + 安装说明 |
| `{{BASE_BRANCH_DETECT}}` | `gen-skill-docs.ts` | 面向 PR 的技能（ship、review、qa、plan-ceo-review）的动态基础分支检测 |
| `{{QA_METHODOLOGY}}` | `gen-skill-docs.ts` | /qa 和 /qa-only 共享的 QA 方法论块 |
| `{{DESIGN_METHODOLOGY}}` | `gen-skill-docs.ts` | /plan-design-review 和 /design-review 共享的设计审查方法论 |
| `{{REVIEW_DASHBOARD}}` | `gen-skill-docs.ts` | /ship 预检的 Review Readiness Dashboard |
| `{{TEST_BOOTSTRAP}}` | `gen-skill-docs.ts` | /qa、/ship、/design-review 的测试框架检测、引导和 CI/CD 设置 |

这在结构上是可靠的——代码中存在的命令才会出现在文档中，不存在就不可能出现。

### preamble（前言）

每个技能都以 `{{PREAMBLE}}` 块开始，在技能自身逻辑之前运行。它通过单条 bash 命令处理四件事：

1. **更新检查** —— 调用 `gstack-update-check`，若有升级可用则报告。
2. **会话追踪** —— 触碰 `~/.gstack/sessions/$PPID` 并统计活跃会话数（过去 2 小时内修改过的文件）。当 3 个及以上会话同时运行时，所有技能进入"ELI16 模式"——每个问题都会重新交代用户所处上下文，因为此时用户在多个窗口间切换。
3. **贡献者模式** —— 从配置中读取 `gstack_contributor`。为 true 时，智能体会在 gstack 自身出现问题时将非正式的现场报告记录到 `~/.gstack/contributor-logs/`。
4. **AskUserQuestion 格式** —— 通用格式：上下文、问题、`RECOMMENDATION: Choose X because ___`、字母选项。在所有技能中保持一致。

### 为什么提交而非运行时生成？

三个原因：

1. **Claude 在技能加载时读取 SKILL.md。** 用户调用 `/browse` 时没有构建步骤，文件必须已经存在且正确。
2. **CI 可以验证新鲜度。** `gen:skill-docs --dry-run` + `git diff --exit-code` 在合并前捕获过期文档。
3. **git blame 可用。** 你可以看到命令是何时、在哪次提交中添加的。

### 模板测试层级

| 层级 | 内容 | 成本 | 速度 |
|------|------|------|------|
| 1 — 静态验证 | 解析 SKILL.md 中的每个 `$B` 命令，对照注册表验证 | 免费 | <2s |
| 2 — 通过 `claude -p` 进行 E2E 测试 | 启动真实 Claude 会话，运行每个技能，检查错误 | ~$3.85 | ~20min |
| 3 — LLM 评审（LLM-as-judge） | Sonnet 对文档的清晰度、完整性和可操作性打分 | ~$0.15 | ~30s |

第 1 层在每次 `bun test` 时运行。第 2、3 层需要 `EVALS=1` 才会触发。思路是：免费捕获 95% 的问题，仅将 LLM 用于需要判断力的情况。

## 命令分发

命令按副作用分类：

- **READ**（text、html、links、console、cookies 等）：无变更，可安全重试，返回页面状态。
- **WRITE**（goto、click、fill、press 等）：变更页面状态，非幂等。
- **META**（snapshot、screenshot、tabs、chain 等）：服务器级操作，不便归入读/写。

这不只是分类上的意义，服务器在分发时也使用它：

```typescript
if (READ_COMMANDS.has(cmd))  → handleReadCommand(cmd, args, bm)
if (WRITE_COMMANDS.has(cmd)) → handleWriteCommand(cmd, args, bm)
if (META_COMMANDS.has(cmd))  → handleMetaCommand(cmd, args, bm, shutdown)
```

`help` 命令返回全部三组集合，以便智能体自行发现可用命令。

## 错误处理哲学

错误是给 AI 智能体看的，不是给人类看的。每条错误信息必须具有可操作性：

- "Element not found" → "Element not found or not interactable. Run `snapshot -i` to see available elements."
- "Selector matched multiple elements" → "Selector matched multiple elements. Use @refs from `snapshot` instead."
- 超时 → "Navigation timed out after 30s. The page may be slow or the URL may be wrong."

Playwright 的原生错误会通过 `wrapError()` 重写，去除内部调用栈并添加指引。智能体应该能直接读懂错误并知道下一步怎么做，无需人工介入。

### 崩溃恢复

服务器不尝试自我修复。若 Chromium 崩溃（`browser.on('disconnected')`），服务器立即退出。CLI 在下一条命令时检测到服务器已死并自动重启。这比试图重连半死状态的浏览器进程更简单、更可靠。

## E2E 测试基础设施

### Session runner（会话运行器）（`test/helpers/session-runner.ts`）

E2E 测试将 `claude -p` 作为完全独立的子进程启动——而非通过 Agent SDK，因为后者无法嵌套在 Claude Code 会话内部。运行器的工作流：

1. 将提示词写入临时文件（避免 shell 转义问题）
2. 启动 `sh -c 'cat prompt | claude -p --output-format stream-json --verbose'`
3. 从 stdout 流式接收 NDJSON，实时展示进度
4. 与可配置的超时竞速
5. 将完整的 NDJSON 转录解析为结构化结果

`parseNDJSON()` 函数是纯函数——无 I/O，无副作用，可独立测试。

### 可观测性数据流

```
  skill-e2e.test.ts
        │
        │ 生成 runId，将 testName + runId 传递给每次调用
        │
  ┌─────┼──────────────────────────────┐
  │     │                              │
  │  runSkillTest()              evalCollector
  │  (session-runner.ts)         (eval-store.ts)
  │     │                              │
  │  每次工具调用：              每次 addTest()：
  │  ┌──┼──────────┐              savePartial()
  │  │  │          │                   │
  │  ▼  ▼          ▼                   ▼
  │ [HB] [PL]    [NJ]          _partial-e2e.json
  │  │    │        │             （原子覆盖写入）
  │  │    │        │
  │  ▼    ▼        ▼
  │ e2e-  prog-  {name}
  │ live  ress   .ndjson
  │ .json .log
  │
  │  失败时：
  │  {name}-failure.json
  │
  │  所有文件位于 ~/.gstack-dev/
  │  运行目录：e2e-runs/{runId}/
  │
  │         eval-watch.ts
  │              │
  │        ┌─────┴─────┐
  │     读取 HB     读取 partial
  │        └─────┬─────┘
  │              ▼
  │        渲染看板
  │        （超过 10 分钟无更新则警告）
  │
```

**职责分离：** session-runner 负责心跳（当前测试状态），eval-store 负责部分结果（已完成测试状态）。观测者同时读取两者，两个组件互不感知——它们仅通过文件系统共享数据。

**全部非致命：** 所有可观测性 I/O 都包裹在 try/catch 中。写入失败绝不导致测试失败。测试本身是事实来源，可观测性是尽力而为。

**机器可读的诊断信息：** 每个测试结果包含 `exit_reason`（success、timeout、error_max_turns、error_api、exit_code_N）、`timeout_at_turn` 和 `last_tool_call`。这支持如下 `jq` 查询：
```bash
jq '.tests[] | select(.exit_reason == "timeout") | .last_tool_call' ~/.gstack-dev/evals/_partial-e2e.json
```

### Eval 持久化（`test/helpers/eval-store.ts`）

`EvalCollector` 累积测试结果，以两种方式写入：

1. **增量写入：** 每次测试后 `savePartial()` 写入 `_partial-e2e.json`（原子写入：先写 `.tmp`，再 `fs.renameSync`），可在进程被杀死后存活。
2. **最终写入：** `finalize()` 写入带时间戳的 eval 文件（如 `e2e-20260314-143022.json`）。partial 文件不会被清理——它与最终文件并存，供可观测性使用。

`eval:compare` 对比两次 eval 运行的差异。`eval:summary` 汇总 `~/.gstack-dev/evals/` 下所有运行的统计数据。

### 测试层级

| 层级 | 内容 | 成本 | 速度 |
|------|------|------|------|
| 1 — 静态验证 | 解析 `$B` 命令，对照注册表验证，可观测性单元测试 | 免费 | <5s |
| 2 — 通过 `claude -p` 进行 E2E 测试 | 启动真实 Claude 会话，运行每个技能，扫描错误 | ~$3.85 | ~20min |
| 3 — LLM 评审（LLM-as-judge） | Sonnet 对文档的清晰度、完整性和可操作性打分 | ~$0.15 | ~30s |

第 1 层在每次 `bun test` 时运行。第 2、3 层需要 `EVALS=1` 才会触发。思路是：免费捕获 95% 的问题，仅将 LLM 用于需要判断力的情况和集成测试。

## 有意不包含的内容

- **没有 WebSocket 流式传输。** HTTP 请求/响应更简单，可用 curl 调试，速度也足够。流式传输会为边际收益引入额外复杂度。
- **没有 MCP 协议。** MCP 每次请求都有 JSON Schema 开销，还需要持久连接。纯 HTTP + 纯文本输出对 token 更友好，也更易调试。
- **没有多用户支持。** 每个工作区一个服务器，一个用户。Token 认证是纵深防御，而非多租户。
- **没有 Windows/Linux Cookie 解密。** 目前仅支持 macOS Keychain。Linux（GNOME Keyring/kwallet）和 Windows（DPAPI）在架构上可以实现，但尚未支持。
- **没有 iframe 支持。** Playwright 可以处理 iframe，但 ref 系统目前尚不支持跨 frame 边界。这是呼声最高的缺失功能。
