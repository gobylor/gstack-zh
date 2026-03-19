> 本文档为 [skills.md](../skills.md) 的中文翻译版本。

# 技能深度指南

gstack 每项技能的详细说明——设计理念、工作流程与使用示例。

| 技能 | 你的专家 | 他们做什么 |
|------|---------|-----------|
| [`/office-hours`](#office-hours) | **YC Office Hours（创业导师）** | 从这里开始。六个强迫性问题，在你写代码之前重新审视你的产品。挑战你的假设前提，提出实现方案的替代思路。产生的设计文档将输入到所有下游技能中。 |
| [`/plan-ceo-review`](#plan-ceo-review) | **CEO / 创始人** | 重新思考问题。找到藏在需求背后的"10 星级产品"。四种模式：扩展（Expansion）、选择性扩展（Selective Expansion）、守住范围（Hold Scope）、缩减（Reduction）。 |
| [`/plan-eng-review`](#plan-eng-review) | **工程经理** | 确定架构、数据流、图表、边界情况和测试。把隐藏的假设逼出来。 |
| [`/plan-design-review`](#plan-design-review) | **高级设计师** | 交互式计划模式设计审查。对每个维度评分 0-10，解释 10 分是什么样子，然后修正计划。在计划模式下工作。 |
| [`/design-consultation`](#design-consultation) | **设计合伙人** | 从零搭建完整的设计系统。了解行业格局，提出创造性风险点，生成逼真的产品原型。将设计置于所有阶段的核心。 |
| [`/review`](#review) | **Staff 工程师** | 找出能通过 CI 但会在生产环境炸锅的 bug。自动修复显而易见的问题。标记完整性缺口。 |
| [`/investigate`](#investigate) | **调试器** | 系统性根因调试。铁律：没有调查就没有修复。追踪数据流，逐一验证假设，连续 3 次修复失败后停止。 |
| [`/design-review`](#design-review) | **会写代码的设计师** | 线上站点视觉审查 + 修复循环。80 条审查项，找到问题后逐一修复。原子提交，前后对比截图。 |
| [`/qa`](#qa) | **QA 负责人** | 测试你的应用，发现 bug，用原子提交修复，再次验证。为每次修复自动生成回归测试。 |
| [`/qa-only`](#qa) | **QA 报告员** | 与 /qa 方法论相同，但只输出报告。当你只需要纯粹的 bug 报告而不想改代码时使用。 |
| [`/ship`](#ship) | **发布工程师** | 同步 main 分支，运行测试，审查覆盖率，推送，创建 PR。如果你没有测试框架，会自动帮你搭建。一个命令搞定。 |
| [`/document-release`](#document-release) | **技术文档工程师** | 更新所有项目文档以匹配你刚发布的内容。自动捕获过时的 README。 |
| [`/retro`](#retro) | **工程经理** | 团队感知的每周回顾。按人员细分，统计发布连续天数、测试健康趋势、成长机会。 |
| [`/browse`](#browse) | **QA 工程师** | 给 Agent 一双眼睛。真实的 Chromium 浏览器，真实的点击，真实的截图。每条命令约 100ms。 |
| [`/setup-browser-cookies`](#setup-browser-cookies) | **会话管理器** | 从你的真实浏览器（Chrome、Arc、Brave、Edge）导入 Cookie 到无头会话。测试需要登录的页面。 |
| | | |
| **Multi-AI（多模型协作）** | | |
| [`/codex`](#codex) | **第二意见** | 来自 OpenAI Codex CLI 的独立审查。三种模式：代码审查（通过/失败门控）、对抗性挑战、带会话延续的开放咨询。当 `/review` 和 `/codex` 都运行过时，进行跨模型分析。 |
| | | |
| **安全与实用工具** | | |
| [`/careful`](#safety--guardrails) | **安全护栏** | 在执行破坏性命令前发出警告（rm -rf、DROP TABLE、force-push、git reset --hard）。可以覆盖任何警告。常见的构建清理操作已加入白名单。 |
| [`/freeze`](#safety--guardrails) | **编辑锁** | 将所有文件编辑限制在单个目录内。阻止在边界之外使用 Edit 和 Write。调试时的意外防护。 |
| [`/guard`](#safety--guardrails) | **完整安全模式** | 一条命令同时启用 /careful + /freeze。生产环境工作时的最高安全级别。 |
| [`/unfreeze`](#safety--guardrails) | **解锁** | 移除 /freeze 边界，重新允许在任意位置编辑。 |
| [`/gstack-upgrade`](#gstack-upgrade) | **自动升级** | 将 gstack 升级到最新版本。检测全局安装还是 vendored 安装，同步两份副本，显示变更内容。 |

---

## `/office-hours`

这是每个项目都应该开始的地方。

在你制定计划之前，在你做代码审查之前，在你写代码之前——先坐下来，和一位 YC 风格的合伙人一起想清楚你究竟在构建什么。不是你以为自己在构建的东西。而是你*真正*在构建的东西。

### 重新定义框架

以下是一个真实项目中发生的事情。用户说："我想为我的日历构建一个每日简报应用。"听起来合理。然后导师问到了痛点——具体的例子，不是假设性的。他们描述了助手遗漏事项、跨多个 Google 账户的日历信息过时、AI 生成的准备文档质量低劣、活动地点错误需要花很长时间才能追踪到正确位置等问题。

导师回应道：*"我要对这个框架提出质疑，因为我认为你已经超出了它的边界。你说的是'多 Google 日历管理的每日简报应用'。但你实际描述的是一个个人首席执行助理 AI。"*

然后它提炼出用户自己都没意识到正在描述的五种能力：

1. **监控你的日历**——跨所有账户，检测过时信息、缺失地点、权限缺口
2. **生成真正的准备工作**——不是物流摘要，而是为董事会会议、播客、融资路演做准备的*智力工作*
3. **管理你的 CRM**——你见了谁，关系如何，对方想要什么，历史背景是什么
4. **优先排序你的时间**——标记何时需要提前开始准备，主动阻断时间，按重要性对活动排序
5. **用钱换杠杆**——主动寻找委托或自动化的机会

这个重新定义改变了整个项目的走向。他们本来要构建一个日历应用。现在他们在构建一个价值高出十倍的东西——因为技能倾听了他们的痛苦，而不是他们的功能需求。

### 前提质疑

重新定义之后，它会呈现一些前提让你验证。不是"这听起来好吗？"——而是关于产品的可证伪性断言：

1. 日历是锚定数据源，但价值在于其上面的智能层
2. 助手不会被替代——他们会获得超能力
3. 最窄的切入点是一个真正有效的每日简报
4. CRM 集成是必须有的，而不是可有可无的

你同意、不同意或调整。你接受的每一个前提都会成为设计文档中的承重柱。

### 实现方案替代选择

然后它会生成 2-3 个具体的实现方案，附带诚实的工作量估算：

- **方案 A：每日简报优先** ——最窄的切入点，明天就能上线，M 级工作量（人工团队：约 3 周 / CC：约 2 天）
- **方案 B：CRM 优先** ——先构建关系图谱，L 级工作量（人工团队：约 6 周 / CC：约 4 天）
- **方案 C：完整愿景** ——一次性全部实现，XL 级工作量（人工团队：约 3 个月 / CC：约 1.5 周）

推荐方案 A，因为你能从真实使用中学习。CRM 数据会在第二周自然涌现。

### 两种模式

**创业模式** ——适合创始人和企业内部创业者，构建一门生意。你会得到六个源自 YC 合伙人评估产品方式的强迫性问题：需求现实、现状、迫切的具体性、最窄切入点、观察与惊喜、未来适配性。这些问题有意让人不舒服。如果你无法说出一个具体的、需要你产品的真实的人，这是你在写任何代码之前最重要的事情。

**构建者模式** ——适合黑客马拉松、副业项目、开源、学习和乐趣。你会得到一个热情的协作者，帮助你找到创意最酷的版本。什么会让人说"哇"？最快能分享出去的路径是什么？这些问题是发散性的，而不是审问性的。

### 设计文档

两种模式都以写入 `~/.gstack/projects/` 的设计文档结束——该文档直接输入到 `/plan-ceo-review` 和 `/plan-eng-review`。完整的生命周期现在是：`office-hours → plan → implement → review → QA → ship → retro`。

设计文档获批后，`/office-hours` 会反思它在交流中观察到的你的思维方式——不是泛泛的赞美，而是对你在会话中说过的具体内容的回调。这些观察也会出现在设计文档中，让你在日后重读时再次邂逅它们。

---

## `/plan-ceo-review`

这是我的**创始人模式**。

在这里，我希望模型以品味、野心、用户同理心和长远视角来思考。我不希望它字面上理解需求。我希望它首先问一个更重要的问题：

**这个产品究竟是为了什么？**

我把这称为 **Brian Chesky 模式**。

重点不是实现显而易见的工单。重点是从用户角度重新思考问题，找到那个感觉不可避免、令人愉悦、甚至有点神奇的版本。

### 示例

假设我在构建一个 Craigslist 风格的挂牌应用，我说：

> "让卖家为商品上传一张照片。"

一个弱助手会添加一个文件选择器并保存图片。

那不是真正的产品。

在 `/plan-ceo-review` 中，我希望模型质疑"照片上传"是否真的是那个功能。也许真正的功能是帮助某人创建一个真正能卖出去的挂牌信息。

如果那才是真正的工作，整个计划就会改变。

现在模型应该问：

* 我们能从照片中识别产品吗？
* 我们能推断出 SKU 或型号吗？
* 我们能搜索网络并自动起草标题和描述吗？
* 我们能拉取规格、品类和定价对比吗？
* 我们能建议哪张照片最适合作为主图吗？
* 我们能检测上传的照片是否丑陋、昏暗、杂乱或缺乏信任感吗？
* 我们能让整个体验感觉高档，而不是像 2007 年的一个死板表单吗？

这就是 `/plan-ceo-review` 为我做的事。

它不只是问："我如何添加这个功能？"
它问的是，**"这个需求背后隐藏着什么样的 10 星级产品？"**

### 四种模式

- **SCOPE EXPANSION（范围扩展）** ——大胆梦想。Agent 提出雄心勃勃的版本。每一项扩展都作为你可以选择加入的独立决策呈现。热情地推荐。
- **SELECTIVE EXPANSION（选择性扩展）** ——以你当前的范围为基准，但看看还有什么可能。Agent 逐一浮出机会，附带中立建议——你挑选值得做的那些。
- **HOLD SCOPE（守住范围）** ——对现有计划进行最严格的审查。不提出任何扩展。
- **SCOPE REDUCTION（范围缩减）** ——找到最小可行版本。砍掉其他所有东西。

愿景和决策持久化到 `~/.gstack/projects/`，超越对话存活。卓越的愿景可以提升到你仓库的 `docs/designs/` 供团队查阅。

---

## `/plan-eng-review`

这是我的**工程经理模式**。

一旦产品方向正确，我需要的是完全不同类型的智慧。我不想要更多发散的构想。我不想要更多"如果……那该多酷"。我希望模型成为我最好的技术负责人。

这个模式应该搞定：

* 架构（architecture）
* 系统边界（system boundaries）
* 数据流（data flow）
* 状态转换（state transitions）
* 故障模式（failure modes）
* 边界情况（edge cases）
* 信任边界（trust boundaries）
* 测试覆盖（test coverage）

还有一个对我来说意外的重大突破：**图表**。

当你强迫 LLM 画出系统时，它会变得完整得多。时序图、状态图、组件图、数据流图，甚至测试矩阵。图表把隐藏的假设逼出来。它们让含糊其辞的规划难以为继。

所以 `/plan-eng-review` 是我希望模型构建能够承载产品愿景的技术骨架的地方。

### 示例

用同样的挂牌应用示例。

假设 `/plan-ceo-review` 已经完成了它的工作。我们决定真正的功能不只是照片上传。它是一个智能挂牌流程，能够：

* 上传照片
* 识别产品
* 从网络丰富挂牌信息
* 起草有力的标题和描述
* 建议最佳主图

现在 `/plan-eng-review` 接管。

现在我希望模型回答这样的问题：

* 上传、分类、丰富和草稿生成的架构是什么？
* 哪些步骤同步进行，哪些进入后台任务？
* 应用服务器、对象存储、视觉模型、搜索/丰富 API 和挂牌数据库之间的边界在哪里？
* 如果上传成功但丰富失败，会怎样？
* 如果产品识别置信度低，会怎样？
* 重试如何工作？
* 如何防止重复任务？
* 什么时候持久化什么，什么可以安全地重新计算？

这就是我想要图表的地方——架构图、状态模型、数据流图、测试矩阵。图表把隐藏的假设逼出来。它们让含糊其辞的规划难以为继。

这就是 `/plan-eng-review`。

不是"让想法变小"。
**而是让想法变得可以构建。**

### 审查就绪仪表板（Review Readiness Dashboard）

每次审查（CEO、Eng、Design）都会记录其结果。每次审查结束时，你会看到一个仪表板：

```
+====================================================================+
|                    REVIEW READINESS DASHBOARD                       |
+====================================================================+
| Review          | Runs | Last Run            | Status    | Required |
|-----------------|------|---------------------|-----------|----------|
| Eng Review      |  1   | 2026-03-16 15:00    | CLEAR     | YES      |
| CEO Review      |  1   | 2026-03-16 14:30    | CLEAR     | no       |
| Design Review   |  0   | —                   | —         | no       |
+--------------------------------------------------------------------+
| VERDICT: CLEARED — Eng Review passed                                |
+====================================================================+
```

Eng Review 是唯一必须通过的门控（可通过 `gstack-config set skip_eng_review true` 禁用）。CEO 和 Design 是信息性的——分别推荐用于产品变更和 UI 变更。

### 计划到 QA 的流程

当 `/plan-eng-review` 完成测试审查部分时，它会将测试计划制品写入 `~/.gstack/projects/`。当你之后运行 `/qa` 时，它会自动获取该测试计划——你的工程审查会直接输入到 QA 测试中，无需手动复制粘贴。

---

## `/plan-design-review`

这是我的**高级设计师审查你的计划**——在你写一行代码之前。

大多数计划描述后端做什么，但从不规定用户实际看到什么。空状态？错误状态？加载状态？移动端布局？AI 千篇一律的风险？这些决定被推迟到"实现时再想"——然后工程师发布了"No items found."作为空状态，因为没有人规定更好的内容。

`/plan-design-review` 在规划阶段捕获所有这些问题，此时修复的代价还很低。

它的工作方式类似于 `/plan-ceo-review` 和 `/plan-eng-review`——交互式，一次一个问题，使用 **STOP + AskUserQuestion** 模式。它对每个设计维度评分 0-10，解释什么是 10 分，然后编辑计划以达到那个水平。评分驱动工作：评分低 = 大量修复，评分高 = 快速通过。

七轮计划审查：信息架构、交互状态覆盖、用户旅程、AI 千篇一律风险、设计系统对齐、响应式/无障碍，以及未解决的设计决策。对于每一轮，它找到缺口并直接修复它们（明显的问题）或请你做出设计选择（真正的权衡）。

### 示例

```
You:   /plan-design-review

Claude: Initial Design Rating: 4/10

        "This plan describes a user dashboard but never specifies
         what the user sees first. It says 'cards with icons' —
         which looks like every SaaS template. It mentions zero
         loading states, zero empty states, and no mobile behavior."

        Pass 1 (Info Architecture): 3/10
        "A 10 would define primary/secondary/tertiary content
         hierarchy for every screen."
        → Added information hierarchy section to plan

        Pass 2 (Interaction States): 2/10
        "The plan has 4 UI features but specifies 0 out of 20
         interaction states (4 features × 5 states each)."
        → Added interaction state table to plan

        Pass 4 (AI Slop): 4/10
        "The plan says 'clean, modern UI with cards and icons'
         and 'hero section with gradient'. These are the top 2
         AI-generated-looking patterns."
        → Rewrote UI descriptions with specific, intentional alternatives

        Overall: 4/10 → 8/10 after fixes
        "Plan is design-complete. Run /design-review after
         implementation for visual QA."
```

当你重新运行它时，已经达到 8 分以上的部分会快速通过。低于 8 分的部分会得到完整处理。对于实现后的线上站点视觉审查，使用 `/design-review`。

---

## `/design-consultation`

这是我的**设计合伙人模式**。

`/plan-design-review` 审查已有站点。`/design-consultation` 适用于你什么都没有的情况——没有设计系统，没有字体选择，没有调色板。你从零开始，想让一位高级设计师坐下来和你一起构建整个视觉身份。

这是一场对话，不是一个表单。Agent 会询问你的产品、用户和受众。它会思考你的产品需要传达什么——信任、速度、工艺、温度，或者任何合适的东西——然后从这个出发点反推到具体的选择。然后它会提出一个完整、连贯的设计系统：审美方向、排版（3 种以上字体，各有特定用途）、带十六进制值的调色板、间距比例、布局方式和动效策略。每个建议都有其理由。每个选择都与其他选择相互强化。

但连贯性只是基本要求。每个开发工具仪表板看起来都一样——干净的无衬线字体、柔和的灰色、蓝色强调色。它们都很连贯。它们都很容易被遗忘。让产品"看起来不错"和让人们真正认得出来的区别，在于**刻意的创造性风险**：标题用出人意料的衬线字体，你所在品类里没有人用的大胆强调色，让你的数据感觉权威而非空旷的更紧凑的间距。

这才是 `/design-consultation` 真正关于的事情。它不只是提出一个安全的系统，而是同时提出安全选择和风险选择——并告诉你哪个是哪个。"这些是让你在品类里被读懂的选择。这里是我认为你应该打破惯例的地方，以及原因。"你选择要承担哪些风险。Agent 会检查整个系统无论哪种方式都仍然保持连贯。

如果你愿意，Agent 可以研究你所在领域已有的东西——截取真实网站的截图，分析它们的字体、颜色和间距——这样你在做出选择之前就能看到整个格局。这不是关于复制。而是让你进入正确的范围，知道惯例是什么，然后决定哪些值得打破。

在你同意系统之后，它会生成一个交互式 HTML 预览页面——不只是色板和字体样本，而是逼真的产品页面。如果你在构建仪表板，你会看到一个带侧边栏、数据表和统计卡片的仪表板。如果你在构建营销网站，你会看到一个带真实文案和 CTA 的英雄区域。一切都在你的设计系统中渲染，使用你的产品名称，支持亮色和暗色模式。在写一行生产代码之前，你就能感受到你的产品会是什么样子。

然后它会将 `DESIGN.md` 写入你的仓库根目录——你项目的设计真理来源——并更新 `CLAUDE.md`，让每个未来的 Claude Code 会话都遵守这个系统。从那时起，`/design-review` 可以以此为基准进行审查，任何在你的前端工作的 Agent 都知道规则。

### 示例

```
You:   /design-consultation

Claude: From the README, this looks like a developer analytics dashboard
        for engineering teams. Sound right? Tell me about the feeling
        you want — and would you like me to look at what's out there
        first so we can see the landscape?

You:    Yes, it's for eng managers. Show me what's out there.

Claude: [browses Linear, Vercel, Datadog, Posthog — takes screenshots]

        Here's the landscape. They all converge on clean sans-serifs
        and muted blue-gray palettes. Linear stands out with dark-first
        and tight spacing. The rest feel interchangeable.

        Here's my proposal — and where I'd play it safe vs. take risks:

        AESTHETIC: Industrial/Utilitarian — data-first, every pixel earns its place
        TYPOGRAPHY: Geist (body + UI), Instrument Serif (display), JetBrains Mono (data)
        COLOR: Restrained — slate neutrals, single teal accent (#0D9488)
        SPACING: 4px base, compact density
        LAYOUT: Grid-disciplined, 12-column
        MOTION: Minimal-functional — only transitions that aid comprehension

        SAFE CHOICES (match category expectations):
          Geist for body — your users already know this font from Vercel
          Slate neutrals — warm grays feel right for data-heavy UIs
          Grid-disciplined layout — dashboards need predictable alignment

        RISKS (where you'd stand out):
          Instrument Serif for display headings — nobody in dev tools
            uses a serif. It creates instant hierarchy contrast and
            signals "we care about craft" in a space full of sans-serifs.
          Single teal accent instead of blue — everyone else in this space uses blue.
            Teal is close enough to feel trustworthy, different enough
            to be recognizable.
          4px compact density — tighter than Linear, says "we respect
            your screen real estate."

        The safe choices keep you literate. The risks are where your
        product gets its own face. Which risks appeal to you?

You:    Love the serif risk. Let's see it.

Claude: Preview opened in your browser.
        [/tmp/design-consultation-preview-1710000000.html]

        The page shows a realistic dashboard: sidebar nav, header with
        avatar, stat cards, data table — all in your design system.
        Instrument Serif at hero scale, Geist in body text, JetBrains
        Mono in the data table. Teal accent on buttons and active states.
        Light and dark mode toggle in the corner.

        How does it feel? Want to adjust anything before I write DESIGN.md?

You:    Ship it.

Claude: Wrote DESIGN.md (typography, color, spacing, layout, motion).
        Updated CLAUDE.md with design system guidelines.
        All future sessions will respect these choices.
```

---

## `/design-review`

这是我的**会写代码的设计师模式**。

`/plan-design-review` 在实现前审查你的计划。`/design-review` 在实现后审查并修复线上站点。

它对你的线上站点运行 80 条视觉审查，然后进入修复循环：对于每个设计发现，它定位源文件，做出最小的 CSS/样式更改，以 `style(design): FINDING-NNN` 提交，重新导航验证，并拍摄前后截图。每次修复一个提交，完全可二等分（bisectable）。

自我调节启发式规则针对设计工作进行了调整——纯 CSS 更改可以免费通过（它们本身是安全且可逆的），但对组件 JSX/TSX 文件的更改会计入风险预算。硬上限 30 次修复。如果风险分数超过 20%，它会停下来询问。

### 示例

```
You:   /design-review https://myapp.com

Claude: [Runs full 80-item visual audit on the live site]
        Design Score: C  |  AI Slop Score: D
        12 findings (4 high, 5 medium, 3 polish)

        Fixing 9 design issues...

        style(design): FINDING-001 — replace 3-column icon grid with asymmetric layout
        style(design): FINDING-002 — add heading scale 48/32/24/18/16
        style(design): FINDING-003 — remove gradient hero, use bold typography
        style(design): FINDING-004 — add second font for headings
        style(design): FINDING-005 — vary border-radius by element role
        style(design): FINDING-006 — left-align body text, reserve center for headings
        style(design): FINDING-007 — add hover/focus states to all interactive elements
        style(design): FINDING-008 — add prefers-reduced-motion media query
        style(design): FINDING-009 — set max content width to 680px for body text

        Final audit:
        Design Score: C → B+  |  AI Slop Score: D → A
        9 fixes applied (8 verified, 1 best-effort). 3 deferred.

        [Report with before/after screenshots saved to .gstack/design-reports/]
```

九个提交，每个只涉及一个问题。AI 千篇一律评分从 D 升到 A，因为三个最典型的模式（渐变英雄区、3 列网格、统一圆角）都已消失。

---

## `/review`

这是我的**偏执 Staff 工程师模式**。

测试通过并不意味着分支是安全的。

`/review` 存在是因为有一整类 bug 能通过 CI 却仍然会在生产环境中给你一记重拳。这个模式不是关于梦想更大。它不是关于让计划更漂亮。它是关于问：

**什么还可能出问题？**

这是一次结构性审计，不是风格挑剔。我希望模型寻找以下类型的问题：

* N+1 查询（N+1 queries）
* 过期读（stale reads）
* 竞争条件（race conditions）
* 错误的信任边界（bad trust boundaries）
* 缺失索引（missing indexes）
* 转义漏洞（escaping bugs）
* 破坏的不变式（broken invariants）
* 错误的重试逻辑（bad retry logic）
* 通过了但错过真实失败模式的测试
* 遗漏的枚举处理——添加新的状态或类型常量，`/review` 会在你整个代码库中追踪它经过的每一个 switch 语句和白名单，而不仅仅是你更改过的文件

### 先修复

发现的问题会得到处理，而不只是被列出来。明显的机械修复（死代码、过期注释、N+1 查询）会自动应用——你会看到每个问题的 `[AUTO-FIXED] 文件:行号 问题 → 做了什么`。真正模糊的问题（安全性、竞争条件、设计决策）会浮出来由你决定。

### 完整性缺口

`/review` 现在会标记那些完整版本所需时间不超过 30 分钟 CC 时间的快捷实现。如果你选择了 80% 的解决方案，而 100% 的解决方案是个"湖泊"而不是"海洋"，审查会指出来。

### 示例

假设智能挂牌流程已经实现，测试已经通过。

`/review` 仍然应该问：

* 在渲染挂牌照片或草稿建议时，我是否引入了 N+1 查询？
* 我是否在信任客户端提供的文件元数据，而不是验证实际文件？
* 两个标签页能否竞争并覆盖封面照片选择或商品详情？
* 失败的上传是否会在存储中留下孤立文件？
* "恰好一张主图"规则能否在并发下被破坏？
* 如果丰富 API 部分失败，我是优雅降级还是保存垃圾数据？
* 通过将网络数据拉入草稿生成，我是否意外创建了提示注入（prompt injection）或信任边界问题？

这就是 `/review` 的意义所在。

我不想要这里的奉承。
我希望模型在生产事故发生之前就能想象到它。

---

## `/investigate`

当某些东西坏了但你不知道为什么，`/investigate` 是你的系统性调试器。它遵循铁律：**没有根因调查，就没有修复。**

它不是猜测和打补丁，而是追踪数据流，与已知 bug 模式对比，并逐一验证假设。如果三次修复尝试失败，它会停下来质疑架构，而不是继续乱试。这防止了浪费数小时的"让我再试一件事"漩涡。

---

## `/qa`

这是我的 **QA 负责人模式**。

`/browse` 给 Agent 眼睛。`/qa` 给它测试方法论。

最常见的用例：你在功能分支上，刚完成编码，想验证一切正常。只需说 `/qa`——它读取你的 git diff，识别你的更改影响了哪些页面和路由，启动浏览器，逐一测试。不需要 URL，不需要手动测试计划。

四种模式：

- **Diff-aware（差异感知）**（在功能分支上自动启用）——读取 `git diff main`，识别受影响的页面，专门测试它们
- **Full（完整）**——系统性地探索整个应用。5-15 分钟。记录 5-10 个有充分证据的问题。
- **Quick（快速）**（`--quick`）——30 秒冒烟测试。首页 + 前 5 个导航目标。
- **Regression（回归）**（`--regression baseline.json`）——运行完整模式，然后与之前的基准进行对比。

### 自动回归测试

当 `/qa` 修复一个 bug 并验证它时，它会自动生成一个回归测试，捕获导致崩溃的确切场景。测试包括完整的归因追踪，回溯到 QA 报告。

### 示例

```
You:   /qa https://staging.myapp.com

Claude: [Explores 12 pages, fills 3 forms, tests 2 flows]

        QA Report: staging.myapp.com — Health Score: 72/100

        Top 3 Issues:
        1. CRITICAL: Checkout form submits with empty required fields
        2. HIGH: Mobile nav menu doesn't close after selecting an item
        3. MEDIUM: Dashboard chart overlaps sidebar below 1024px

        [Full report with screenshots saved to .gstack/qa-reports/]
```

**测试需要身份验证的页面：** 先使用 `/setup-browser-cookies` 导入你真实的浏览器会话，然后 `/qa` 就能测试登录后的页面。

---

## `/ship`

这是我的**发布机器模式**。

一旦我决定了要构建什么，确定了技术计划，并进行了认真的审查，我不想再说话了。我想要执行。

`/ship` 用于最后一英里。它适用于已准备好的分支，而不是决定构建什么。

这是模型应该停止表现得像头脑风暴伙伴，开始表现得像一个有纪律的发布工程师的地方：与 main 同步，运行正确的测试，确保分支状态正常，如果仓库需要则更新 changelog 或版本，推送，并创建或更新 PR。

### 测试框架引导

如果你的项目没有测试框架，`/ship` 会帮你搭建——检测你的运行时，研究最佳框架，安装它，为你的实际代码编写 3-5 个真实测试，设置 CI/CD（GitHub Actions），并创建 TESTING.md。100% 测试覆盖率是目标——测试让 vibe 编码变得安全，而不是 yolo 编码。

### 覆盖率审计

每次 `/ship` 运行都会从你的差异构建代码路径图，搜索对应的测试，并生成带质量星级的 ASCII 覆盖率图。缺口会自动生成测试。你的 PR 正文显示覆盖率：`Tests: 42 → 47 (+5 new)`。

### 审查门控

`/ship` 在创建 PR 之前检查[审查就绪仪表板](#审查就绪仪表板-review-readiness-dashboard)。如果缺少 Eng Review，它会询问——但不会阻止你。决定按分支保存，所以你不会被重复询问。

很多分支在有趣的工作完成后死掉，只剩下无聊的发布工作。人类会拖延那部分。AI 不应该。

---

## `/document-release`

这是我的**技术文档工程师模式**。

在 `/ship` 创建 PR 之后但合并之前，`/document-release` 读取项目中的每个文档文件，并将其与差异进行交叉对比。它更新文件路径、命令列表、项目结构树以及任何其他偏离的内容。有风险或主观的更改会作为问题浮出——其他所有内容都会自动处理。

```
You:   /document-release

Claude: Analyzing 21 files changed across 3 commits. Found 8 documentation files.

        README.md: updated skill count from 9 to 10, added new skill to table
        CLAUDE.md: added new directory to project structure
        CONTRIBUTING.md: current — no changes needed
        TODOS.md: marked 2 items complete, added 1 new item

        All docs updated and committed. PR body updated with doc diff.
```

它还会润色 CHANGELOG 的措辞（不覆盖已有条目），清理已完成的 TODOS，检查跨文档一致性，并仅在适当时询问 VERSION 版本号。

---

## `/retro`

这是我的**工程经理模式**。

在周末我想知道实际发生了什么。不是感觉——是数据。`/retro` 分析提交历史、工作模式和发布速度，写出一份坦率的回顾。

它是团队感知的。它识别谁在运行该命令，对你自己的工作给出最深入的处理，然后对每个贡献者进行细分，提供具体的表扬和成长机会。它计算提交数、代码行数（LOC）、测试比率、PR 大小和修复比率等指标。它从提交时间戳检测编码会话，找到热点文件，跟踪发布连续天数，识别本周最大的发布。

它还跟踪测试健康状况：总测试文件数、本期新增测试数、回归测试提交数和趋势变化量。如果测试比率降至 20% 以下，它会将其标记为成长领域。

### 示例

```
You:   /retro

Claude: Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs, peak: 10pm | Streak: 47d

        ## Your Week
        32 commits, +2.4k LOC, 41% tests. Peak hours: 9-11pm.
        Biggest ship: cookie import system (browser decryption + picker UI).
        What you did well: shipped a complete feature with encryption, UI, and
        18 unit tests in one focused push...

        ## Team Breakdown

        ### Alice
        12 commits focused on app/services/. Every PR under 200 LOC — disciplined.
        Opportunity: test ratio at 12% — worth investing before payment gets more complex.

        ### Bob
        3 commits — fixed the N+1 query on dashboard. Small but high-impact.
        Opportunity: only 1 active day this week — check if blocked on anything.

        [Top 3 team wins, 3 things to improve, 3 habits for next week]
```

它将 JSON 快照保存到 `.context/retros/`，这样下次运行时就能显示趋势。

---

## `/browse`

这是我的 **QA 工程师模式**。

`/browse` 是完成闭环的技能。在此之前，Agent 可以思考和编码，但仍然半盲。它不得不猜测 UI 状态、身份验证流程、重定向、控制台错误、空状态和破坏的布局。现在它可以直接去看。

它是一个与持久化 Chromium 守护进程通信的编译二进制文件——基于微软的 [Playwright](https://playwright.dev/) 构建。第一次调用启动浏览器（约 3 秒）。之后每次调用：约 100-200 毫秒。浏览器在命令之间保持运行，所以 Cookie、标签页和 localStorage 会延续。

### 示例

```
You:   /browse staging.myapp.com — log in, test the signup flow, and check
       every page I changed in this branch

Claude: [18 tool calls, ~60 seconds]

        > browse goto https://staging.myapp.com/signup
        > browse snapshot -i
        > browse fill @e2 "test@example.com"
        > browse fill @e3 "password123"
        > browse click @e5                    (Submit)
        > browse screenshot /tmp/signup.png
        > Read /tmp/signup.png

        Signup works. Redirected to onboarding. Now checking changed pages.

        > browse goto https://staging.myapp.com/dashboard
        > browse screenshot /tmp/dashboard.png
        > Read /tmp/dashboard.png
        > browse console

        Dashboard loads. No console errors. Charts render with sample data.

        All 4 pages load correctly. No console errors. No broken layouts.
        Signup → onboarding → dashboard flow works end to end.
```

18 次工具调用，大约一分钟。完整的 QA 通过。没有打开浏览器窗口。

### 浏览器移交（Browser handoff）

当无头浏览器卡住时——CAPTCHA、多因素认证、复杂的身份验证——将控制权移交给用户：

```
Claude: I'm stuck on a CAPTCHA at the login page. Opening a visible
        Chrome so you can solve it.

        > browse handoff "Stuck on CAPTCHA at login page"

        Chrome opened at https://app.example.com/login with all your
        cookies and tabs intact. Solve the CAPTCHA and tell me when
        you're done.

You:    done

Claude: > browse resume

        Got a fresh snapshot. Logged in successfully. Continuing QA.
```

浏览器在移交过程中保留所有状态（Cookie、localStorage、标签页）。`resume` 之后，Agent 获取你离开时所在位置的新快照。如果 browse 工具连续失败 3 次，它会自动建议使用 `handoff`。

**安全说明：** `/browse` 运行一个持久化的 Chromium 会话。Cookie、localStorage 和会话状态在命令之间延续。除非你有意为之，否则不要对敏感的生产环境使用它——它是带有真实状态的真实浏览器。会话在空闲 30 分钟后自动关闭。

完整的命令参考，请参见 [BROWSER.md](../BROWSER.md)。

---

## `/setup-browser-cookies`

这是我的**会话管理器模式**。

在 `/qa` 或 `/browse` 能够测试需要身份验证的页面之前，它们需要 Cookie。每次都通过无头浏览器手动登录，而不是直接从你的日常浏览器导入真实会话——那将是在浪费时间。`/setup-browser-cookies` 就是解决这个问题的。

它自动检测已安装的 Chromium 浏览器（Comet、Chrome、Arc、Brave、Edge），通过 macOS Keychain 解密 Cookie，并将它们加载到 Playwright 会话中。一个交互式选择器 UI 让你选择要导入的确切域名——Cookie 值从不显示。

```
You:   /setup-browser-cookies

Claude: Cookie picker opened — select the domains you want to import
        in your browser, then tell me when you're done.

        [You pick github.com, myapp.com in the browser UI]

You:    done

Claude: Imported 2 domains (47 cookies). Session is ready.
```

或者完全跳过 UI：

```
You:   /setup-browser-cookies github.com

Claude: Imported 12 cookies for github.com from Comet.
```

---

## `/codex`

这是我的**第二意见模式**。

当 `/review` 从 Claude 的视角捕获 bug 时，`/codex` 带来一个完全不同的 AI——OpenAI 的 Codex CLI——来审查同一个差异。不同的训练，不同的盲点，不同的优势。重叠部分告诉你什么是确定真实的。每个模型独有的发现是你找到任何一个单独都无法捕获的 bug 的地方。

### 三种模式

**Review（审查）** ——对当前差异运行 `codex review`。Codex 读取每个更改的文件，按严重性分类发现（P1 严重、P2 高危、P3 中危），并返回通过/失败裁决。任何 P1 发现 = 失败。审查是完全独立的——Codex 看不到 Claude 的审查。

**Challenge（挑战）** ——对抗模式。Codex 主动尝试破坏你的代码。它寻找边界情况、竞争条件、安全漏洞和在负载下会失败的假设。使用最大推理努力（`xhigh`）。把它想象成对你的逻辑进行渗透测试。

**Consult（咨询）** ——带会话延续的开放对话。询问 Codex 关于代码库的任何问题。后续问题重用同一个会话，所以上下文延续。非常适合"我这样思考对吗？"的时刻。

### 跨模型分析

当 `/review`（Claude）和 `/codex`（OpenAI）都审查了同一个分支，你会得到一个跨模型比较：哪些发现重叠（高置信度），哪些是 Codex 独有的（不同视角），哪些是 Claude 独有的。这是对代码审查采用"两个医生，同一个病人"的方法。

```
You:   /codex review

Claude: Running independent Codex review...

        CODEX REVIEW: PASS (3 findings)
        [P2] Race condition in payment handler — concurrent charges
             can double-debit without advisory lock
        [P3] Missing null check on user.email before downcase
        [P3] Token comparison not using constant-time compare

        Cross-model analysis (vs /review):
        OVERLAP: Race condition in payment handler (both caught it)
        UNIQUE TO CODEX: Token comparison timing attack
        UNIQUE TO CLAUDE: N+1 query in listing photos
```

---

## 安全与护栏（Safety & Guardrails）

四个技能，为任何 Claude Code 会话添加安全护栏。它们通过 Claude Code 的 PreToolUse 钩子工作——透明、会话范围、无需配置文件。

### `/careful`

当你在生产环境附近工作、运行破坏性命令或只是想要一个安全网时，说"be careful"或运行 `/careful`。每个 Bash 命令都会针对已知危险模式进行检查：

- `rm -rf` / `rm -r` ——递归删除
- `DROP TABLE` / `DROP DATABASE` / `TRUNCATE` ——数据丢失
- `git push --force` / `git push -f` ——历史重写
- `git reset --hard` ——丢弃提交
- `git checkout .` / `git restore .` ——丢弃未提交的工作
- `kubectl delete` ——生产资源删除
- `docker rm -f` / `docker system prune` ——容器/镜像丢失

常见的构建产物清理（`rm -rf node_modules`、`dist`、`.next`、`__pycache__`、`build`、`coverage`）已加入白名单——日常操作不会产生误报。

你可以覆盖任何警告。护栏是意外防护，不是访问控制。

### `/freeze`

将所有文件编辑限制在单个目录内。当你在调试计费 bug 时，你不希望 Claude 意外地"修复" `src/auth/` 中不相关的代码。`/freeze src/billing` 会阻止该路径之外的所有 Edit 和 Write 操作。

`/investigate` 会自动激活它——它检测正在调试的模块，并将编辑冻结到该目录。

```
You:   /freeze src/billing

Claude: Edits restricted to src/billing/. Run /unfreeze to remove.

        [Later, Claude tries to edit src/auth/middleware.ts]

Claude: BLOCKED — Edit outside freeze boundary (src/billing/).
        Skipping this change.
```

注意：这只阻止 Edit 和 Write 工具。像 `sed` 这样的 Bash 命令仍然可以修改边界之外的文件——这是意外防护，不是安全沙箱。

### `/guard`

完整安全模式——一条命令同时启用 `/careful` + `/freeze`。破坏性命令警告加上目录范围编辑。在触及生产或调试线上系统时使用。

### `/unfreeze`

移除 `/freeze` 边界，重新允许在任意位置编辑。钩子在会话中保持注册——它们只是允许所有操作。再次运行 `/freeze` 以设置新边界。

---

## `/gstack-upgrade`

一条命令保持 gstack 最新。它检测你的安装类型（全局安装在 `~/.claude/skills/gstack` 还是 vendored 安装在项目的 `.claude/skills/gstack`），运行升级，如果你有双重安装则同步两份副本，并显示更改内容。

```
You:   /gstack-upgrade

Claude: Current version: 0.7.4
        Latest version: 0.8.2

        What's new:
        - Browse handoff for CAPTCHAs and auth walls
        - /codex multi-AI second opinion
        - /qa always uses browser now
        - Safety skills: /careful, /freeze, /guard
        - Proactive skill suggestions

        Upgraded to 0.8.2. Both global and project installs synced.
```

在 `~/.gstack/config.yaml` 中设置 `auto_upgrade: true` 以完全跳过提示——当新版本可用时，gstack 在每个会话开始时静默升级。

---

## Greptile 集成

[Greptile](https://greptile.com) 是一家自动审查你的 PR 的 YC 公司。它捕获真实的 bug——竞争条件、安全问题、通过 CI 却在生产环境炸锅的事情。它确实不止一次救了我。我很喜欢这些人。

### 设置

在 [greptile.com](https://greptile.com) 上将 Greptile 安装到你的 GitHub 仓库——大约需要 30 秒。一旦它开始审查你的 PR，gstack 就会自动获取其评论。无需额外配置。

### 工作原理

任何自动化审查工具的问题在于分类（triage）。Greptile 很好，但不是每条评论都是真实的问题。有些是误报。有些标记了你三个提交前已经修复的东西。如果没有分类层，评论就会堆积，你开始忽视它们——这就违背了目的。

gstack 解决了这个问题。`/review` 和 `/ship` 现在是 Greptile 感知的。它们读取 Greptile 的评论，对每条进行分类，并采取行动：

- **有效问题**被添加到关键发现中，并在发布前修复
- **已修复的问题**获得自动回复，确认捕获
- **误报**被驳回——你确认后，一条回复解释为什么它是错的

结果是两层审查：Greptile 在 PR 上异步捕获问题，然后 `/review` 和 `/ship` 将这些发现作为正常工作流程的一部分进行分类。没有任何东西会从缝隙中漏掉。

### 从历史中学习

你确认的每一个误报都会保存到 `~/.gstack/greptile-history.md`。未来的运行会自动跳过你的代码库中已知的误报模式。`/retro` 会随时间追踪 Greptile 的命中率——这样你就能看到信噪比是否在改善。

### 示例

```
You:   /ship

Claude: [syncs main, runs tests, pre-landing review...]

        Greptile found 3 comments on this PR:

        [VALID] app/services/payment_service.rb:47 — Race condition:
        concurrent charges can double-debit. Recommend DB-level advisory
        lock around the charge block.

        [ALREADY FIXED] app/models/user.rb:12 — Missing null check
        on email before downcase.
        → Already fixed in commit abc1234. Auto-replying.

        [FALSE POSITIVE] lib/auth.rb:88 — "Token comparison should
        use constant-time comparison." We already use
        ActiveSupport::SecurityUtils.secure_compare here.

        Fixing the race condition in payment_service.rb...

        The auth token comment is a false positive.
        Options: A) Reply to Greptile explaining  B) Fix anyway  C) Ignore

You:    A

Claude: Replied to Greptile. All tests pass.
        PR: github.com/you/app/pull/42
```

三条 Greptile 评论。一个真实修复。一个自动确认。一个误报被附带回复驳回。额外花费的时间：大约 30 秒。
