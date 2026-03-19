# gstack 综合学习指南

> 本文档是专为中文用户编写的 gstack 学习指南，帮助你从零开始掌握 gstack 和 Claude Code 的顶级使用技巧。

---

## 目录

- [一、gstack 是什么](#一gstack-是什么)
- [二、核心架构：三层设计](#二核心架构三层设计)
- [三、Skill 系统详解](#三skill-系统详解)
- [四、Browse CLI：给 AI 的浏览器](#四browse-cli给-ai-的浏览器)
- [五、测试体系：三层金字塔](#五测试体系三层金字塔)
- [六、完整的工程工作流](#六完整的工程工作流)
- [七、Claude Code 顶级使用技巧](#七claude-code-顶级使用技巧)
- [八、快速上手路线图](#八快速上手路线图)

---

## 一、gstack 是什么

**一句话总结：** gstack 是一套给 Claude Code 用的「技能包」（Skills）+ 一个无头浏览器 CLI，让 AI agent 能自动完成 QA 测试、Code Review、发布上线、设计审查等完整工程工作流。

它不是一个应用程序，而是一套 **增强 Claude Code 能力的插件系统**。

### 谁创建了 gstack？

gstack 的作者是 Y Combinator 的 CEO Garry Tan。他用 gstack 实现了**每天产出 10,000-20,000 行可用代码**——作为 CEO 日常工作的一部分。他的理念是：一个人 + AI 工具 = 过去需要 20 人团队的产出。

### 核心理念

gstack 把 Claude Code 变成了一个 **虚拟工程团队**：
- **CEO** (`/office-hours`, `/plan-ceo-review`) — 重新思考产品方向
- **工程经理** (`/plan-eng-review`) — 锁定架构和技术方案
- **设计师** (`/plan-design-review`, `/design-review`) — 审查 UX 和视觉质量
- **Staff 工程师** (`/review`) — 找到 CI 通过但生产环境会崩溃的 bug
- **QA 负责人** (`/qa`) — 打开真实浏览器测试你的应用
- **发布工程师** (`/ship`) — 跑测试、创建 PR、一键发布

15 个专家角色 + 6 个工具，全部是 slash 命令，全部是 Markdown，全部 MIT 开源免费。

---

## 二、核心架构：三层设计

```
┌─────────────────────────────────────────────────┐
│  第三层: Skills（技能层）                          │
│  15+ SKILL.md 文件，每个是一段结构化的 prompt       │
│  /ship, /review, /qa, /investigate, /retro ...   │
├─────────────────────────────────────────────────┤
│  第二层: Browse CLI（浏览器工具层）                 │
│  编译为单一二进制文件，~100ms/命令，持久化会话        │
│  goto, snapshot, click, fill, screenshot ...      │
├─────────────────────────────────────────────────┤
│  第一层: 基础设施（模板引擎 + 测试框架）             │
│  gen-skill-docs.ts → 从源码生成文档                │
│  三层测试：静态校验 → E2E → LLM-as-Judge           │
└─────────────────────────────────────────────────┘
```

### 为什么是这样的架构？

1. **Skills 是 Markdown**：Claude Code 的 Skill 就是一个 `.md` 文件，放在 `~/.claude/skills/` 目录下。用户输入 `/skill-name` 时，Claude 把这个 md 文件作为 prompt 注入上下文。所以 SKILL.md 不是文档，而是 **写给 AI 的指令**。

2. **Browse 是编译二进制**：用 Bun 编译成 ~58MB 的单文件可执行程序。不需要 `node_modules`，不需要配置 PATH。直接运行。

3. **模板系统保证代码和文档同步**：SKILL.md 从 `.tmpl` 模板和源码自动生成，命令参考表直接从 `commands.ts` 源码读取，所以文档永远不会过时。

---

## 三、Skill 系统详解

### 3.1 Skill 的本质

在 Claude Code 中，一个 Skill 就是一个 `SKILL.md` 文件。当你输入 `/review` 时：

1. Claude Code 在 `~/.claude/skills/` 目录下找到 `review/SKILL.md`
2. 把文件内容作为 prompt 注入当前对话上下文
3. Claude 按照文件中的指令一步步执行

**关键认知：SKILL.md 中的每一行都在控制 Claude 的行为。**

### 3.2 Skill 的结构

每个 SKILL.md 包含：

```yaml
---
name: ship                    # 技能名称
version: 1.0.0               # 版本号
description: "发布工作流"       # 描述
allowed-tools: Bash, Read, Edit, Write, AskUserQuestion
---
```

**前置元数据（frontmatter）** 告诉 Claude：
- 这个技能叫什么、干什么
- 允许使用哪些工具（**这是安全约束**）

然后是分阶段的自然语言指令：

```markdown
Step 0: 预检
- 读取 CLAUDE.md 获取测试命令
- 如果找不到测试命令，用 AskUserQuestion 问用户

Step 1: 运行测试
- 执行 Step 0 中找到的测试命令
- 如果测试失败，停止并报告

Step 2: 创建 PR
...
```

### 3.3 模板驱动的生成系统

gstack 的精妙设计：**永远不直接编辑 SKILL.md，只编辑 `.tmpl` 模板文件**。

```
SKILL.md.tmpl  ──(gen-skill-docs.ts)──→  SKILL.md
```

模板中使用 `{{PLACEHOLDER}}` 占位符：

| 占位符 | 来源 | 作用 |
|--------|------|------|
| `{{PREAMBLE}}` | 硬编码模板函数 | 通用技能头部（更新检查、会话追踪、分析埋点） |
| `{{COMMAND_REFERENCE}}` | `browse/src/commands.ts` | 从源码自动生成命令参考表 |
| `{{SNAPSHOT_FLAGS}}` | `browse/src/snapshot.ts` | 从源码自动生成快照标志表 |
| `{{BROWSE_SETUP}}` | 模板函数 | 浏览器二进制定位逻辑 |
| `{{QA_METHODOLOGY}}` | 500 行模板函数 | QA 测试方法论 |

**为什么要这样做？** 因为 browse CLI 的命令会变化。如果手动维护文档，命令改了文档忘了改，AI 就会用错误的命令。从源码生成 = **文档永远和代码同步**。

### 3.4 Skill 的关键设计原则

#### a) 平台无关（Platform-agnostic）

Skills 从不硬编码 `npm test` 或 `rails test`。它们：
1. 先读 CLAUDE.md 找项目配置
2. 找不到就用 AskUserQuestion 问用户
3. 把答案 **持久化到 CLAUDE.md**，下次不再问

这意味着 gstack 可以用于任何技术栈：Rails、Node、Python、Go 等。

#### b) Bash 块互相独立

SKILL.md 中的每个 bash 代码块在 **独立 shell** 中执行，变量不共享。所以用自然语言传递状态：

```markdown
Step 1 中检测到的基础分支是 main。

Step 2: 基于 Step 1 检测到的基础分支，运行 git diff...
```

#### c) Fix-First 模式

Review 类技能不只报告问题，而是分两类：
- **AUTO-FIX**: 机械性问题（拼写错误、格式问题），直接自动修复
- **ASK**: 需要判断的（架构决策、逻辑变更），询问用户

### 3.5 所有 Skills 一览

| 技能 | 角色 | 功能 |
|------|------|------|
| `/office-hours` | YC Office Hours | 头脑风暴，用 6 个关键问题重新定义你的产品 |
| `/plan-ceo-review` | CEO/创始人 | 战略审查，4 种模式：扩展/选择性扩展/保持/缩减 |
| `/plan-eng-review` | 工程经理 | 架构审查，数据流图、边界情况、测试计划 |
| `/plan-design-review` | 高级设计师 | 设计审查，0-10 评分，解释怎样才是 10 分 |
| `/design-consultation` | 设计合伙人 | 从零构建完整设计系统 |
| `/review` | Staff 工程师 | Code Review，自动修复机械问题 |
| `/investigate` | 调试专家 | 系统性根因调试，铁律：没有调查就没有修复 |
| `/design-review` | 会写代码的设计师 | 视觉审查 + 修复循环 |
| `/qa` | QA 负责人 | 浏览器自动化测试 + bug 修复 + 回归测试 |
| `/qa-only` | QA 报告员 | 纯报告，不修改代码 |
| `/ship` | 发布工程师 | 跑测试 → 版本号 → 创建 PR |
| `/document-release` | 技术写手 | 自动更新项目文档 |
| `/retro` | 工程经理 | 周回顾：代码指标 + 趋势分析 |
| `/browse` | QA 工程师 | 无头浏览器操作 |
| `/codex` | 第二意见 | 用 OpenAI Codex 独立审查 |
| `/careful` | 安全护栏 | 在破坏性命令前发出警告 |
| `/freeze` | 编辑锁 | 限制文件编辑范围 |
| `/guard` | 完整安全模式 | `/careful` + `/freeze` |

---

## 四、Browse CLI：给 AI 的浏览器

### 4.1 为什么需要它？

AI agent 操作浏览器需要 **亚秒级延迟** 和 **持久化状态**。如果每个命令都冷启动浏览器，需要 3-5 秒。如果浏览器在命令之间关闭，Cookie、标签页、登录状态全部丢失。

### 4.2 架构

```
CLI (cli.ts)  ──HTTP──→  Server (server.ts)  ──Playwright──→  Chromium
   薄客户端              持久化守护进程                 无头浏览器
   ~1ms 启动             自动启动/30分钟空闲停止         Cookie 持久化
```

- **首次调用**：~3 秒（启动 Chromium）
- **后续调用**：~100-200ms（只是 HTTP POST）
- Cookie、localStorage、标签页 **跨命令持久化**

### 4.3 Snapshot：核心创新

`$B snapshot` 输出一棵 **可访问性树**（Accessibility Tree），每个交互元素分配一个 ref（引用）：

```
@e1 link "首页"
@e2 button "登录"
@e3 textbox "邮箱"
@e4 textbox "密码"
@e5 button "提交"
```

然后 AI 可以：
```bash
$B fill @e3 "user@example.com"   # 填写邮箱
$B fill @e4 "password123"        # 填写密码
$B click @e5                     # 点击提交
$B snapshot -D                   # 显示差异，确认操作生效
```

**为什么用 ref 而不是 CSS 选择器？**
- 不修改 DOM（不会破坏 CSP、React 水合、Shadow DOM）
- 基于可访问性树，比 CSS 选择器更可靠
- 自动检测过期 ref（SPA 场景下 DOM 变化后会报错提示重新 snapshot）

### 4.4 命令分类

| 类别 | 示例 | 用途 |
|------|------|------|
| 导航 | `goto`, `back`, `forward` | 页面跳转 |
| 读取 | `text`, `html`, `links`, `forms`, `js` | 提取页面数据 |
| 交互 | `click`, `fill`, `select`, `upload` | 操作页面 |
| 检查 | `is visible`, `is enabled`, `console`, `network` | 断言验证 |
| 视觉 | `screenshot`, `snapshot -a`, `responsive`, `diff` | 视觉证据 |
| 会话 | `cookie-import-browser`, `handoff`, `resume` | Cookie 导入、人机切换 |

### 4.5 人机切换（Handoff）

当无头浏览器遇到 CAPTCHA、多因素认证等 AI 无法处理的场景：

```bash
$B handoff "在登录页遇到了 CAPTCHA"   # 打开可见的 Chrome 窗口
# 用户手动解决 CAPTCHA...
$B resume                              # 回到无头模式，状态完全保留
```

连续失败 3 次后会自动建议 handoff。

---

## 五、测试体系：三层金字塔

```
          ┌──────────────┐
          │  LLM-as-Judge │  ~$0.15/次，质量评分
          │  第三层        │
          ├──────────────┤
          │  E2E 测试     │  ~$3.85/次，真实调用 claude -p
          │  第二层        │
          ├──────────────┤
          │  静态校验      │  免费 <1s，命令合法性
          │  第一层        │
          └──────────────┘
```

### 第一层：静态校验（`bun test`）

- 扫描所有 SKILL.md 中的 `$B` 命令
- 验证每个命令是否存在于 `commands.ts` 注册表中
- **零成本、<1 秒、每次提交前运行**

### 第二层：E2E 测试（`bun run test:e2e`）

- 启动 `claude -p`（Claude CLI 的 prompt 模式）作为子进程
- 喂入测试 prompt，流式读取 NDJSON 输出
- 验证 Claude 是否正确调用了 browse 命令
- **~$3.85/次，Diff-based 只跑受影响的测试**

### 第三层：LLM-as-Judge（`bun run test:evals`）

- 用 Claude Sonnet 作为评委
- 对生成的文档质量打分（清晰度/完整度/可操作性，1-5 分）
- 对 QA 报告评分（是否发现预埋 bug、是否有误报）
- **~$0.15/次**

### Diff-based 智能选择

每个测试声明自己依赖的文件（touchfiles）。运行时根据 `git diff` 只跑被影响的测试。改了全局文件（如 gen-skill-docs.ts）则跑全部。

---

## 六、完整的工程工作流

gstack 展示了一个完整的 AI 辅助开发循环：

```
想法
 ↓
/office-hours          ← 头脑风暴、需求澄清
 ↓
/plan-ceo-review       ← 战略审查：范围是否合理？
/plan-eng-review       ← 架构审查：技术方案是否可行？
/plan-design-review    ← 设计审查：UX 是否合理？
 ↓
实现代码
 ↓
/investigate           ← 遇到 bug 时系统调试
 ↓
/qa                    ← QA 测试（用 browse 自动化）
 ↓
/review                ← Code Review
 ↓
/ship                  ← 跑测试 → 版本号 → 创建 PR
 ↓
/document-release      ← 自动更新文档
 ↓
/retro                 ← 周回顾：指标 + 趋势
```

**每一步都是一个 Skill，每一步 Claude 都知道该做什么、用什么工具、怎么验证结果。**

### 并行冲刺

这个流程的强大之处在于可以 **并行运行 10-15 个冲刺**：
- 一个窗口在 `/office-hours` 讨论新想法
- 另一个窗口在 `/review` 审查 PR
- 第三个窗口在实现功能
- 第四个窗口在 `/qa` 测试 staging
- 还有六个窗口在其他分支上工作

有了结构化流程，10 个 agent 不是 10 个混乱源。每个 agent 知道做什么、什么时候停。

---

## 七、Claude Code 顶级使用技巧

以下是从 gstack 项目中提炼的最佳实践：

### 技巧 1：用 CLAUDE.md 作为项目的"大脑"

```markdown
# CLAUDE.md 中应该包含：
- 构建命令（bun test, bun run build）
- 项目结构说明
- 提交规范
- 测试策略
- 关键架构决策
```

Claude Code **每次对话都会读 CLAUDE.md**。把项目知识沉淀在这里，新会话不需要重新解释。

### 技巧 2：写 Skills 自动化重复工作流

任何重复执行的工作流（review、ship、debug），都应该写成 Skill。一个 SKILL.md 就是一段 **编排 AI 行为的 prompt**。

### 技巧 3：用模板系统保持文档和代码同步

gstack 用 `{{PLACEHOLDER}}` + 代码解析器实现了 **代码即文档**。你可以用类似思路给自己的项目生成文档。

### 技巧 4：分层测试 AI 输出

- 静态检查（免费）：格式、命令合法性
- E2E 测试（付费）：真实执行验证
- LLM-as-Judge（付费）：语义质量评分

### 技巧 5：Diff-based 选择性测试

不要每次跑全量测试。声明依赖关系，根据 git diff 只跑受影响的测试，节省成本。

### 技巧 6：用 `allowed-tools` 限制 Skill 的权限

SKILL.md 的 frontmatter 中声明 `allowed-tools`，Claude 在执行这个 skill 时只能使用指定工具，防止意外操作。

### 技巧 7：Preamble 统一行为

所有 skill 共享一个 `{{PREAMBLE}}`，统一处理更新检查、会话追踪、使用分析、完成状态协议。

### 技巧 8：平台无关设计

永远不在 skill 中硬编码框架命令。读 CLAUDE.md → 问用户 → 持久化答案。

### 技巧 9：Bisect Commits

每个 commit 是一个独立的逻辑变更。重命名和行为变更分开、模板变更和生成文件分开。

### 技巧 10：错误信息写给 AI 看

gstack 的错误信息不是给人看的，是给 AI agent 看的。每个错误消息都附带下一步操作建议：
- "Element not found" → "Element not found. Run `snapshot -i` to see available elements."
- 超时 → "Navigation timed out after 30s. The page may be slow or the URL may be wrong."

---

## 八、快速上手路线图

### 第一步：安装（30 秒）

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

### 第二步：配置 CLAUDE.md

在你的项目 CLAUDE.md 中添加：

```markdown
## gstack
Use /browse from gstack for all web browsing.
Available skills: /office-hours, /plan-ceo-review, /plan-eng-review,
/plan-design-review, /review, /ship, /browse, /qa, /investigate, /retro
```

### 第三步：试用技能

1. **`/office-hours`** — 描述你在做什么，体验 AI 如何重新定义你的产品
2. **`/review`** — 在有改动的分支上运行，体验自动化 code review
3. **`/qa`** — 对一个本地网站运行，体验浏览器自动化测试
4. **`/ship`** — 体验一键发布

### 第四步：深入学习

1. 读 `ship/SKILL.md.tmpl` — 学习 skill 的写法
2. 读 `scripts/gen-skill-docs.ts` — 理解模板系统
3. 跑 `bun test` — 体验静态校验
4. 尝试修改一个 `.tmpl` 文件 → 跑 `bun run gen:skill-docs` → 看生成的 `.md` 变化

### 第五步：写自己的 Skill

从一个简单的重复工作流开始，参考现有 skill 的结构编写你自己的 SKILL.md。

---

## 进一步阅读

| 文档 | 内容 |
|------|------|
| [README 中文版](README.zh-CN.md) | 项目介绍、安装方法、功能概览 |
| [架构文档中文版](ARCHITECTURE.zh-CN.md) | 设计决策和系统内部原理 |
| [浏览器参考中文版](BROWSER.zh-CN.md) | Browse CLI 完整命令参考 |
| [贡献指南中文版](CONTRIBUTING.zh-CN.md) | 开发环境搭建、测试、贡献方式 |
| [技能深度解析中文版](skills.zh-CN.md) | 每个技能的哲学、工作流、示例 |
| [更新日志中文版](CHANGELOG.zh-CN.md) | 每个版本的新功能 |
