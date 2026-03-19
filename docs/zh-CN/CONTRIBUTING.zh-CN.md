> 本文档为 [CONTRIBUTING.md](../../CONTRIBUTING.md) 的中文翻译版本。

# 为 gstack 贡献代码

感谢你愿意让 gstack 变得更好。无论是修复 skill（技能）提示词中的一个错别字，还是构建全新的工作流，本指南都能帮你快速上手。

## 快速开始

gstack skill 是 Markdown 文件，Claude Code 会从 `skills/` 目录中自动发现它们。通常这些文件位于 `~/.claude/skills/gstack/`（你的全局安装目录）。但在开发 gstack 本身时，你希望 Claude Code 使用*工作树（working tree）中的 skill*——这样编辑后无需复制或部署即可立即生效。

这正是开发者模式（dev mode）的用途。它会将你的代码库以符号链接（symlink）的形式接入本地的 `.claude/skills/` 目录，让 Claude Code 直接从你的 checkout 中读取 skill。

```bash
git clone <repo> && cd gstack
bun install                    # 安装依赖
bin/dev-setup                  # 激活开发者模式
```

现在编辑任意 `SKILL.md`，在 Claude Code 中调用它（例如 `/review`），即可实时看到变更效果。开发完成后执行：

```bash
bin/dev-teardown               # 停用开发者模式，恢复全局安装
```

## 贡献者模式

贡献者模式（contributor mode）将 gstack 变成一个自我改进的工具。启用后，Claude Code 会在每个主要工作流步骤结束时周期性地反思其 gstack 使用体验——打出 0-10 的评分。当某步骤没能达到满分 10 时，它会思考原因，并将发生的事情、复现步骤以及改进建议以报告的形式写入 `~/.gstack/contributor-logs/`。

```bash
~/.claude/skills/gstack/bin/gstack-config set gstack_contributor true
```

这些日志是**给你看的**。当某个问题烦扰你到想动手修复时，报告早已写好。Fork gstack，将你的 fork 以符号链接接入遇到问题的项目，修复它，然后提交 PR。

### 贡献者工作流

1. **正常使用 gstack** —— 贡献者模式会自动反思并记录问题
2. **查看你的日志：** `ls ~/.gstack/contributor-logs/`
3. **Fork 并 clone gstack**（如果你还没有的话）
4. **将你的 fork 以符号链接接入遇到 bug 的项目：**
   ```bash
   # 在你的核心项目中（也就是让你对 gstack 感到不满的那个）
   ln -sfn /path/to/your/gstack-fork .claude/skills/gstack
   cd .claude/skills/gstack && bun install && bun run build
   ```
5. **修复问题** —— 你的改动在该项目中立即生效
6. **通过实际使用 gstack 来测试** —— 重现让你不满的操作，验证问题已被修复
7. **从你的 fork 提交 PR**

这是最佳的贡献方式：在做实际工作的同时修复 gstack，就在你真正感受到痛点的那个项目里。

### Session（会话）感知

当你同时打开 3 个及以上 gstack session 时，每个问题都会告诉你当前处于哪个项目、哪个分支、正在发生什么。不再盯着一个问题发呆思考"等等，这是哪个窗口？"全部 15 个 skill 的格式都保持一致。

## 在 gstack 仓库内部开发 gstack

当你正在编辑 gstack skill 并想通过在同一仓库中实际使用 gstack 来测试时，`bin/dev-setup` 会帮你完成配置。它会创建指向你工作树的 `.claude/skills/` 符号链接（已被 gitignore），使 Claude Code 使用本地编辑内容而非全局安装。

```
gstack/                          <- 你的工作树
├── .claude/skills/              <- 由 dev-setup 创建（已被 gitignore）
│   ├── gstack -> ../../         <- 符号链接指向仓库根目录
│   ├── review -> gstack/review
│   ├── ship -> gstack/ship
│   └── ...                      <- 每个 skill 各一个符号链接
├── review/
│   └── SKILL.md                 <- 编辑此文件，用 /review 测试
├── ship/
│   └── SKILL.md
├── browse/
│   ├── src/                     <- TypeScript 源码
│   └── dist/                    <- 编译后的二进制文件（已被 gitignore）
└── ...
```

## 日常工作流

```bash
# 1. 进入开发者模式
bin/dev-setup

# 2. 编辑一个 skill
vim review/SKILL.md

# 3. 在 Claude Code 中测试——改动立即生效
#    > /review

# 4. 修改了 browse 源码？重新构建二进制文件
bun run build

# 5. 今天的工作结束了？撤销开发者模式
bin/dev-teardown
```

## 测试与评估（Evals）

### 环境配置

```bash
# 1. 复制 .env.example 并填入你的 API Key
cp .env.example .env
# 编辑 .env → 设置 ANTHROPIC_API_KEY=sk-ant-...

# 2. 安装依赖（如果还没安装）
bun install
```

Bun 会自动加载 `.env`，无需额外配置。Conductor（并行工作区工具）工作区会自动从主工作树继承 `.env`（详见下方"Conductor 工作区"章节）。

### 测试层级

| 层级 | 命令 | 费用 | 测试内容 |
|------|---------|------|---------------|
| 1 — 静态验证 | `bun test` | 免费 | 命令验证、snapshot flags（快照标志）、SKILL.md 正确性、TODOS-format.md 引用、可观测性单元测试 |
| 2 — E2E | `bun run test:e2e` | ~$3.85 | 通过 `claude -p` 子进程完整执行 skill |
| 3 — LLM 评估 | `bun run test:evals` | ~$0.15（单独运行） | 对生成的 SKILL.md 文档进行 LLM-as-judge（以 LLM 为评判者）评分 |
| 2+3 | `bun run test:evals` | ~$4（合并运行） | E2E + LLM-as-judge（同时运行两者） |

```bash
bun test                     # 仅第 1 层（每次提交时运行，<5s）
bun run test:e2e             # 第 2 层：仅 E2E（需要 EVALS=1，不能在 Claude Code 内部运行）
bun run test:evals           # 第 2 + 3 层合并（~$4/次）
```

### 第 1 层：静态验证（免费）

随 `bun test` 自动运行，无需 API Key。

- **Skill 解析器测试**（`test/skill-parser.test.ts`）—— 从 SKILL.md bash 代码块中提取所有 `$B` 命令，并对照 `browse/src/commands.ts` 中的命令注册表进行验证。可捕获错别字、已删除的命令和无效的 snapshot flags。
- **Skill 验证测试**（`test/skill-validation.test.ts`）—— 验证 SKILL.md 文件仅引用了真实存在的命令和标志，以及命令描述是否达到质量标准。
- **生成器测试**（`test/gen-skill-docs.test.ts`）—— 测试模板系统：验证占位符能否正确解析、输出是否包含标志的值提示（如 `-d <N>` 而非仅 `-d`）、关键命令是否有丰富描述（如 `is` 列出有效状态，`press` 列出按键示例）。

### 第 2 层：通过 `claude -p` 的 E2E 测试（~$3.85/次）

将 `claude -p` 作为子进程启动，使用 `--output-format stream-json --verbose`，以 NDJSON 格式流式传输实时进度，并扫描 browse 错误。这是最接近"这个 skill 端到端能否正常工作"的测试方式。

```bash
# 必须从普通终端运行——不能嵌套在 Claude Code 或 Conductor 内部
EVALS=1 bun test test/skill-e2e.test.ts
```

- 由 `EVALS=1` 环境变量控制（防止意外触发高费用运行）
- 如果在 Claude Code 内部运行则自动跳过（`claude -p` 不能嵌套）
- API 连通性预检——在耗费预算之前遇到 ConnectionRefused 时快速失败
- 实时进度输出到 stderr：`[Ns] turn T tool #C: Name(...)`
- 保存完整的 NDJSON 转录文件和失败 JSON，便于调试
- 测试位于 `test/skill-e2e.test.ts`，运行器逻辑位于 `test/helpers/session-runner.ts`

### E2E 可观测性

E2E 测试运行时，会在 `~/.gstack-dev/` 中生成机器可读的产物（artifact）：

| 产物 | 路径 | 用途 |
|----------|------|---------|
| 心跳（Heartbeat） | `e2e-live.json` | 当前测试状态（每次工具调用后更新） |
| 部分结果 | `evals/_partial-e2e.json` | 已完成的测试（进程被杀死后仍可保留） |
| 进度日志 | `e2e-runs/{runId}/progress.log` | 仅追加的文本日志 |
| NDJSON 转录 | `e2e-runs/{runId}/{test}.ndjson` | 每个测试的原始 `claude -p` 输出 |
| 失败 JSON | `e2e-runs/{runId}/{test}-failure.json` | 失败时的诊断数据 |

**实时仪表盘：** 在第二个终端中运行 `bun run eval:watch`，可查看显示已完成测试、当前运行中的测试和费用的实时仪表盘。使用 `--tail` 参数还可同时显示 progress.log 的最后 10 行。

**评估历史工具：**

```bash
bun run eval:list            # 列出所有评估运行（每次运行的轮次、时长、费用）
bun run eval:compare         # 对比两次运行——显示每个测试的差异 + Takeaway 评注
bun run eval:summary         # 跨运行的聚合统计 + 每个测试的效率平均值
```

**评估对比评注：** `eval:compare` 会生成自然语言 Takeaway 章节，解读两次运行之间的变化——标记回归、记录改进、指出效率提升（更少轮次、更快、更便宜），并给出整体总结。这由 `eval-store.ts` 中的 `generateCommentary()` 驱动。

产物不会被自动清理——它们会在 `~/.gstack-dev/` 中持续积累，用于事后调试和趋势分析。

### 第 3 层：LLM-as-judge（~$0.15/次）

使用 Claude Sonnet 从三个维度对生成的 SKILL.md 文档进行评分：

- **清晰度（Clarity）** —— AI agent 能否在没有歧义的情况下理解这些指令？
- **完整性（Completeness）** —— 所有命令、标志和使用模式是否都已记录？
- **可执行性（Actionability）** —— agent 能否仅凭文档中的信息完成任务？

每个维度打分 1-5 分。阈值：每个维度必须达到 **≥ 4 分**。此外还有一项回归测试，将生成的文档与 `origin/main` 中手动维护的基准进行比较——生成的文档必须得分等于或高于基准。

```bash
# 需要 .env 中有 ANTHROPIC_API_KEY——已包含在 bun run test:evals 中
```

- 使用 `claude-sonnet-4-6` 以保证评分稳定性
- 测试位于 `test/skill-llm-eval.test.ts`
- 直接调用 Anthropic API（而非 `claude -p`），因此可在任何地方运行，包括 Claude Code 内部

### CI（持续集成）

一个 GitHub Action（`.github/workflows/skill-docs.yml`）在每次 push 和 PR 时运行 `bun run gen:skill-docs --dry-run`。如果生成的 SKILL.md 文件与已提交的内容不同，CI 将失败。这可在文档过期合并之前提前捕获问题。

测试直接针对 browse 二进制文件运行——不需要开发者模式。

## 编辑 SKILL.md 文件

SKILL.md 文件是从 `.tmpl` 模板**生成**的。不要直接编辑 `.md` 文件——你的修改会在下次构建时被覆盖。

```bash
# 1. 编辑模板
vim SKILL.md.tmpl              # 或 browse/SKILL.md.tmpl

# 2. 重新生成
bun run gen:skill-docs

# 3. 检查健康状态
bun run skill:check

# 或使用 watch 模式——保存时自动重新生成
bun run dev:skill
```

模板编写最佳实践（使用自然语言而非 bash 风格、动态分支检测、`{{BASE_BRANCH_DETECT}}` 的用法），请参见 CLAUDE.md 中的"Writing SKILL templates"章节。

要添加 browse 命令，请将其加入 `browse/src/commands.ts`。要添加 snapshot flag，请将其加入 `browse/src/snapshot.ts` 中的 `SNAPSHOT_FLAGS`。然后重新构建。

## Conductor 工作区

如果你使用 [Conductor](https://conductor.build) 并行运行多个 Claude Code session，`conductor.json` 会自动配置工作区生命周期：

| Hook | 脚本 | 功能 |
|------|--------|-------------|
| `setup` | `bin/dev-setup` | 从主工作树复制 `.env`，安装依赖，符号链接 skill |
| `archive` | `bin/dev-teardown` | 移除 skill 符号链接，清理 `.claude/` 目录 |

当 Conductor 创建新工作区时，`bin/dev-setup` 会自动运行。它会检测主工作树（通过 `git worktree list`），复制你的 `.env` 以便 API Key 随之传递，并设置开发者模式——无需手动操作。

**首次配置：** 在主仓库的 `.env` 中填入你的 `ANTHROPIC_API_KEY`（参见 `.env.example`）。每个 Conductor 工作区都会自动继承它。

## 注意事项

- **SKILL.md 文件是生成的。** 编辑 `.tmpl` 模板，而非 `.md` 文件。运行 `bun run gen:skill-docs` 重新生成。
- **TODOS.md 是统一的待办事项列表（backlog）。** 按 skill/组件组织，优先级为 P0-P4。`/ship` 会自动检测已完成的条目。所有规划/审查/复盘 skill 都会读取它以获取上下文。
- **修改 browse 源码后需要重新构建。** 如果你修改了 `browse/src/*.ts`，请运行 `bun run build`。
- **开发者模式会遮蔽你的全局安装。** 项目本地的 skill 优先于 `~/.claude/skills/gstack`。`bin/dev-teardown` 会恢复全局安装。
- **Conductor 工作区相互独立。** 每个工作区都是独立的 git worktree。`bin/dev-setup` 会通过 `conductor.json` 自动运行。
- **`.env` 在工作树间传播。** 在主仓库中设置一次，所有 Conductor 工作区都会获得它。
- **`.claude/skills/` 已被 gitignore。** 这些符号链接不会被提交。

## 在真实项目中测试你的改动

**这是开发 gstack 的推荐方式。** 将你的 gstack checkout 以符号链接接入你实际使用它的项目，这样在进行真实工作的同时改动就已生效：

```bash
# 在你的核心项目中
ln -sfn /path/to/your/gstack-checkout .claude/skills/gstack
cd .claude/skills/gstack && bun install && bun run build
```

现在该项目中每次调用 gstack skill 都会使用你的工作树。编辑模板，运行 `bun run gen:skill-docs`，下次调用 `/review` 或 `/qa` 时就会立即生效。

**恢复到稳定的全局安装**，只需移除符号链接：

```bash
rm .claude/skills/gstack
```

Claude Code 会自动回退到 `~/.claude/skills/gstack/`。

### 备选方案：将全局安装指向某个分支

如果你不想为每个项目创建符号链接，可以切换全局安装指向的版本：

```bash
cd ~/.claude/skills/gstack
git fetch origin
git checkout origin/<branch>
bun install && bun run build
```

这会影响所有项目。恢复时执行：`git checkout main && git pull && bun run build`。

## 发布你的改动

当你对 skill 编辑满意后：

```bash
/ship
```

这会运行测试、审查 diff、处理 Greptile 评论（两级升级机制）、管理 TODOS.md、更新版本号，并开启一个 PR。完整工作流请参见 `ship/SKILL.md`。
