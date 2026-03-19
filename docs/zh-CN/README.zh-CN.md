> 本文档为 [README.md](../../README.md) 的中文翻译版本。

# gstack

你好，我是 [Garry Tan](https://x.com/garrytan)，[Y Combinator](https://www.ycombinator.com/) 的总裁兼 CEO。我与数千家初创公司合作过，包括 Coinbase、Instacart 和 Rippling——当时这些公司的创始人还只是一两个人在车库里工作，如今已是价值数百亿美元的企业。在 YC 之前，我设计了 Palantir 的 Logo，并曾是那里最早的工程经理/PM/设计师之一。我联合创办了 Posterous（一个博客平台，后被 Twitter 收购）。2013 年，我搭建了 Bookface，YC 的内部社交网络。作为设计师、PM 和工程经理，我做产品已经很多年了。

而此刻，我正身处一个感觉全然崭新的时代。

在过去 60 天里，我写了**超过 60 万行生产代码**——其中 35% 是测试——同时作为 YC CEO 履行所有职责，每天只用部分时间就能产出 **1 万到 2 万行可用代码**。这不是笔误。我最近一次 `/retro`（过去 7 天的开发者统计）横跨 3 个项目：**新增 140,751 行，362 次提交，约 11.5 万行净增**。模型每周都在大幅提升。我们正处于某种真实事物的黎明——一个人能以过去需要二十人团队才能达到的规模交付产品。

**2026 年——1,237 次贡献，持续增长中：**

![GitHub 贡献记录 2026 — 1,237 次贡献，1-3 月大幅加速](docs/images/github-2026.png)

**2013 年——我在 YC 构建 Bookface 时（772 次贡献）：**

![GitHub 贡献记录 2013 — 772 次贡献，在 YC 构建 Bookface](docs/images/github-2013.png)

同一个人。不同的时代。区别在于工具。

**gstack 就是我的方法。** 这是我的开源软件工厂。它把 Claude Code 变成一支你真正能管理的虚拟工程团队——一位重新思考产品的 CEO，一位锁定架构的工程经理，一位发现 AI 粗制滥造的设计师，一位挑剔的审查者专门找生产 bug，一位 QA（质量保证）负责人打开真实浏览器点遍你的应用，以及一位发布工程师负责提交 PR。十五位专家加六个强力工具，全部是斜杠命令（Slash Command），全部是 Markdown，**全部免费，MIT 许可证，现在就能用。**

我正在学习如何触及 2026 年 3 月智能体系统的能力边界，这是我的实时实验。我把它分享出来，因为我希望全世界都和我一起踏上这段旅程。

Fork（分叉）它。改进它。让它成为你的。别嫉妒，去欣赏。

**适合谁：**
- **创始人和 CEO**——尤其是那些仍然想亲自交付的技术型创始人。这是你以二十人团队规模构建产品的方式。
- **Claude Code 新手**——gstack 是最好的起点。有了结构化角色，告别空白提示。
- **技术负责人和资深工程师**——为每个 PR 带来严格的审查、QA 和发布自动化。

## 快速上手：前 10 分钟

1. 安装 gstack（30 秒——见下文）
2. 运行 `/office-hours`——描述你在构建什么。它会在你写一行代码之前重新定义问题。
3. 对任何功能想法运行 `/plan-ceo-review`
4. 对任何有改动的分支运行 `/review`
5. 对你的预发布（staging）URL 运行 `/qa`
6. 到此为止。你会知道这是否适合你。

在任何已有测试的仓库中，5 分钟内即可得到第一次有用的运行结果。

**如果你只再读一个章节，读这个。**

## 安装——只需 30 秒

**依赖：** [Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[Git](https://git-scm.com/)、[Bun](https://bun.sh/) v1.0+

### 第一步：安装到你的机器

打开 Claude Code，粘贴以下内容。Claude 会处理剩下的事。

> Install gstack: run **`git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`** then add a "gstack" section to CLAUDE.md that says to use the /browse skill from gstack for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, and lists the available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /browse, /qa, /qa-only, /design-review, /setup-browser-cookies, /retro, /investigate, /document-release, /codex, /careful, /freeze, /guard, /unfreeze, /gstack-upgrade. Then ask the user if they also want to add gstack to the current project so teammates get it.

### 第二步：添加到你的仓库让队友也能用（可选）

> Add gstack to this project: run **`cp -Rf ~/.claude/skills/gstack .claude/skills/gstack && rm -rf .claude/skills/gstack/.git && cd .claude/skills/gstack && ./setup`** then add a "gstack" section to this project's CLAUDE.md that says to use the /browse skill from gstack for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, lists the available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /browse, /qa, /qa-only, /design-review, /setup-browser-cookies, /retro, /investigate, /document-release, /codex, /careful, /freeze, /guard, /unfreeze, /gstack-upgrade, and tells Claude that if gstack skills aren't working, run `cd .claude/skills/gstack && ./setup` to build the binary and register skills.

真实文件会被提交到你的仓库（不是 submodule 子模块），所以 `git clone` 开箱即用。所有内容都在 `.claude/` 目录内，不会触碰你的 PATH，也不会在后台运行任何东西。

## 看它如何工作

```
你:     我想构建一个日历的每日简报应用。
你:     /office-hours
Claude: [询问痛点——具体例子，而非假设情景]

你:     多个 Google 日历，事件信息陈旧，地点错误。
        准备工作永远占用太多时间，而且效果不够好……

Claude: 我要反驳你的框架。你说的是"每日简报应用"。
        但你实际描述的是一个个人 AI 参谋长。
        [提取了 5 个你自己没意识到在描述的能力]
        [挑战了 4 个前提——你同意、不同意或调整]
        [生成了 3 种实现方案及工作量估算]
        建议：明天就交付最窄的楔形方案，从真实使用中学习。
        完整愿景是个 3 个月的项目——先从真正能用的每日简报开始。
        [编写设计文档 → 自动流入下游技能]

你:     /plan-ceo-review
        [读取设计文档，挑战范围，进行 10 项评审]

你:     /plan-eng-review
        [数据流、状态机、错误路径的 ASCII 图表]
        [测试矩阵，故障模式，安全关注点]

你:     批准计划。退出计划模式。
        [在 11 个文件中写了 2,400 行。约 8 分钟。]

你:     /review
        [自动修复] 2 个问题。[待确认] 竞态条件 → 你批准修复。

你:     /qa https://staging.myapp.com
        [打开真实浏览器，点遍流程，找到并修复一个 bug]

你:     /ship
        测试：42 → 51（+9 个新测试）。PR: github.com/you/app/pull/42
```

你说"每日简报应用"。智能体说"你在构建一个 AI 参谋长"——因为它倾听的是你的痛点，而不是你的功能需求。然后它挑战你的前提，生成三种方案，推荐最窄的楔形方案，并写了一份设计文档流入每个下游技能。八条命令。这不是一个副驾驶（Copilot）。这是一支团队。

## 冲刺流程（Sprint）

gstack 是一套流程，不是工具的堆砌。这些技能（Skill）按照冲刺流程的顺序排列：

**思考 → 规划 → 构建 → 审查 → 测试 → 发布 → 复盘**

每个技能都流入下一个。`/office-hours` 写的设计文档被 `/plan-ceo-review` 读取。`/plan-eng-review` 写的测试计划被 `/qa` 拾取。`/review` 发现的 bug 被 `/ship` 验证已修复。没有什么会漏网，因为每个步骤都知道前一步做了什么。

一次冲刺，一个人，一个功能——用 gstack 大约需要 30 分钟。但真正改变一切的是：你可以并行运行 10-15 次这样的冲刺。不同功能，不同分支，不同智能体——同时进行。这就是我每天在做好本职工作的同时交付 1 万行以上生产代码的方式。

| 技能 | 你的专家 | 他们做什么 |
|-------|----------------|--------------|
| `/office-hours` | **YC Office Hours（创业咨询时间）** | 从这里开始。六个强制性问题，在你写代码之前重新定义你的产品。反驳你的框架，挑战前提，生成多种实现方案。设计文档自动流入所有下游技能。 |
| `/plan-ceo-review` | **CEO / 创始人** | 重新思考问题。找到隐藏在需求中的 10 星产品。四种模式：扩展、选择性扩展、维持范围、缩减。 |
| `/plan-eng-review` | **工程经理** | 锁定架构、数据流、图表、边界情况和测试。将隐藏的假设逼出水面。 |
| `/plan-design-review` | **资深设计师** | 对每个设计维度评分 0-10，解释 10 分是什么样的，然后编辑计划以达到那个水准。AI 粗制滥造检测。交互式——每个设计选择一个 AskUserQuestion。 |
| `/design-consultation` | **设计合伙人** | 从零构建完整的设计系统。了解行业现状，提出创意冒险，生成真实的产品模型。让设计成为所有阶段的核心。 |
| `/review` | **资深工程师（Staff Engineer）** | 找到那些通过 CI 但会在生产环境爆炸的 bug。自动修复明显问题。标记完整性缺口。 |
| `/investigate` | **调试专家** | 系统性根因调试。铁律：没有调查就没有修复。追踪数据流，测试假设，3 次修复失败后停止。 |
| `/design-review` | **会写代码的设计师** | 与 /plan-design-review 相同的审计，然后修复发现的问题。原子提交，修复前后截图对比。 |
| `/qa` | **QA 负责人** | 测试你的应用，找出 bug，用原子提交修复，重新验证。为每个修复自动生成回归测试。 |
| `/qa-only` | **QA 报告员** | 与 /qa 方法相同但只出报告。当你想要纯粹的 bug 报告而不改动代码时使用。 |
| `/ship` | **发布工程师** | 同步主干，运行测试，审计覆盖率，推送，开 PR。如果你没有测试框架，会自动搭建一个。一条命令搞定。 |
| `/document-release` | **技术写作员** | 更新所有项目文档以匹配你刚发布的内容。自动发现陈旧的 README。 |
| `/retro` | **工程经理** | 团队感知的每周回顾。按人员拆分，发布连胜记录，测试健康趋势，成长机会。 |
| `/browse` | **QA 工程师** | 给智能体一双眼睛。真实的 Chromium 浏览器，真实的点击，真实的截图。约 100 毫秒每条命令。 |
| `/setup-browser-cookies` | **会话管理员** | 从你的真实浏览器（Chrome、Arc、Brave、Edge）导入 Cookie 到无头会话。测试需要登录的页面。 |

### 强力工具

| 技能 | 功能 |
|-------|-------------|
| `/codex` | **第二意见** — 来自 OpenAI Codex CLI 的独立代码审查。三种模式：审查（通过/拒绝门控）、对抗性挑战、开放咨询。当 `/review` 和 `/codex` 都运行后提供跨模型分析。 |
| `/careful` | **安全护栏** — 在破坏性命令（rm -rf、DROP TABLE、force-push）执行前发出警告。说"be careful"激活。可覆盖任何警告。 |
| `/freeze` | **编辑锁定** — 将文件编辑限制在一个目录内。调试时防止意外修改范围外的内容。 |
| `/guard` | **完全安全** — 一条命令同时激活 `/careful` 和 `/freeze`。生产工作的最大安全保障。 |
| `/unfreeze` | **解锁** — 移除 `/freeze` 边界。 |
| `/gstack-upgrade` | **自我更新** — 将 gstack 升级到最新版本。检测全局安装还是 vendored（内嵌）安装，同步两者，显示变更内容。 |

**[每个技能的深度解析，含示例和设计哲学 →](docs/skills.md)**

## 新特性及其意义

**`/office-hours` 在你写代码之前重新定义你的产品。** 你说"每日简报应用"。它倾听你真实的痛点，反驳框架，告诉你其实你在构建一个个人 AI 参谋长，挑战你的前提，并生成三种带工作量估算的实现方案。它写的设计文档直接流入 `/plan-ceo-review` 和 `/plan-eng-review`——所以每个下游技能都从真正的清晰认知出发，而不是模糊的功能需求。

**设计处于核心位置。** `/design-consultation` 不只是挑字体。它研究你所在领域的现有产品，提出稳妥的选择和创意冒险，生成你实际产品的真实模型，并写入 `DESIGN.md`——然后 `/design-review` 和 `/plan-eng-review` 读取你的选择。设计决策流贯整个系统。

**`/qa` 是一次重大突破。** 它让我从 6 个并行工作者扩展到 12 个。Claude Code 说*"我看到问题了"*然后真正修复它，生成回归测试，验证修复——这改变了我的工作方式。智能体现在有眼睛了。

**智能审查路由。** 就像在运作良好的初创公司：CEO 不需要看基础设施 bug 修复，后端改动不需要设计审查。gstack 追踪已运行的审查，判断什么是合适的，然后做出明智的决定。审查就绪仪表板在发布前告诉你当前状态。

**测试一切。** 如果你的项目没有测试框架，`/ship` 会从头搭建一个。每次 `/ship` 运行都会产生覆盖率审计。每次 `/qa` bug 修复都会生成回归测试。100% 测试覆盖率是目标——测试让氛围驱动编码（vibe coding）变得安全，而不是鲁莽编码（yolo coding）。

**`/document-release` 是你从未有过的工程师。** 它读取项目中的每个文档文件，与 diff 交叉对比，更新所有偏离的内容。README、ARCHITECTURE、CONTRIBUTING、CLAUDE.md、TODOS——全部自动保持最新。现在 `/ship` 会自动调用它——无需额外命令，文档始终保持同步。

**AI 卡住时的浏览器交接。** 遇到 CAPTCHA（验证码）、认证墙或 MFA（多因素认证）提示？`$B handoff` 会在同一页面打开一个可见的 Chrome 窗口，携带所有 Cookie 和标签页。你解决问题后告诉 Claude 已完成，`$B resume` 从原处继续。连续 3 次失败后，智能体甚至会自动建议你这样做。

**多 AI 第二意见。** `/codex` 从 OpenAI 的 Codex CLI 获取独立审查——一个完全不同的 AI 审视同一个 diff。三种模式：带通过/拒绝门控的代码审查、主动尝试破坏你代码的对抗性挑战，以及具有会话连续性的开放咨询。当 `/review`（Claude）和 `/codex`（OpenAI）都审查了同一分支，你会得到一个跨模型分析，显示哪些发现重叠，哪些是各自独有的。

**按需安全护栏。** 说"be careful"，`/careful` 会在任何破坏性命令前发出警告——rm -rf、DROP TABLE、force-push、git reset --hard。`/freeze` 在调试时将编辑锁定到一个目录，防止 Claude 意外"修复"无关代码。`/guard` 同时激活两者。`/investigate` 会自动冻结到被调查的模块。

**主动技能建议。** gstack 会注意你所处的阶段——头脑风暴、代码审查、调试、测试——并建议合适的技能。不喜欢？说"stop suggesting"，它会在会话间记住。

## 10-15 个并行冲刺

一个冲刺时 gstack 很强大。十个同时运行时它是变革性的。

[Conductor](https://conductor.build) 可以并行运行多个 Claude Code 会话——每个都在各自隔离的工作空间中。一个会话对新想法运行 `/office-hours`，另一个对 PR 做 `/review`，第三个在实现功能，第四个对预发布环境运行 `/qa`，还有六个在其他分支上。同时进行。我经常运行 10-15 个并行冲刺——这是目前的实际上限。

冲刺结构正是让并行化得以运作的关键。没有流程，十个智能体就是十个混乱来源。有了流程——思考、规划、构建、审查、测试、发布——每个智能体都清楚地知道该做什么以及何时停止。你像 CEO 管理团队一样管理它们：关注重要的决策，让其他的自行运转。

---

## 一起踏上这段旅程

这是**免费、MIT 许可、开源、现在就能用的。** 没有付费层。没有等待名单。没有附加条件。

我开源了我的开发方式，并在这里积极升级我自己的软件工厂。你可以 fork 它并让它成为你自己的。这就是整个意义所在。我希望每个人都踏上这段旅程。

同样的工具，不同的结果——因为 gstack 给你的是结构化角色和审查关卡，而不是混乱的通用智能体。这种治理就是快速交付和鲁莽交付之间的区别。

模型在快速变好。那些现在就学会如何与之合作的人——真正合作，而不只是浅尝——将拥有巨大的优势。这就是那个窗口。我们走吧。

十五位专家和六个强力工具。全部斜杠命令。全部 Markdown。全部免费。**[github.com/garrytan/gstack](https://github.com/garrytan/gstack)** — MIT 许可证

> **我们正在招聘。** 想每天交付 1 万行以上代码并帮助强化 gstack 吗？
> 来 YC 工作——[ycombinator.com/software](https://ycombinator.com/software)
> 极具竞争力的薪资和期权。旧金山，Dogpatch 区。

## 文档

| 文档 | 内容 |
|-----|---------------|
| [技能深度解析](docs/skills.md) | 每个技能的设计哲学、示例和工作流（含 Greptile 集成） |
| [架构](ARCHITECTURE.md) | 设计决策和系统内部机制 |
| [浏览器参考](BROWSER.md) | `/browse` 的完整命令参考 |
| [贡献指南](CONTRIBUTING.md) | 开发环境设置、测试、贡献者模式和开发模式 |
| [更新日志](CHANGELOG.md) | 每个版本的新内容 |

## 故障排除

**技能没有出现？** `cd ~/.claude/skills/gstack && ./setup`

**`/browse` 失败？** `cd ~/.claude/skills/gstack && bun install && bun run build`

**安装陈旧？** 运行 `/gstack-upgrade`——或在 `~/.gstack/config.yaml` 中设置 `auto_upgrade: true`

**Claude 说看不到技能？** 确保你项目的 `CLAUDE.md` 有一个 gstack 章节。添加以下内容：

```
## gstack
Use /browse from gstack for all web browsing. Never use mcp__claude-in-chrome__* tools.
Available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review,
/design-consultation, /review, /ship, /browse, /qa, /qa-only, /design-review,
/setup-browser-cookies, /retro, /investigate, /document-release, /codex, /careful,
/freeze, /guard, /unfreeze, /gstack-upgrade.
```

## 许可证

MIT。永久免费。去构建点什么吧。
