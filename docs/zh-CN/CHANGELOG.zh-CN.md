> 本文档为 [CHANGELOG.md](../../CHANGELOG.md) 的中文翻译版本。

# 更新日志

## [0.8.5] - 2026-03-19

### 修复

- **`/retro` 现在按完整自然日计算时间。** 深夜运行 retro 不再悄悄丢失当天早些时候的提交。Git 处理裸日期时，若你在晚上 11 点运行，会将 `--since="2026-03-11"` 解释为"3 月 11 日晚上 11 点"——现在改为传入 `--since="2026-03-11T00:00:00"`，始终从午夜起算。对比模式的时间窗口也做了同样的修复。
- **review log 不再因含 `/` 的分支名而出错。** 类似 `garrytan/design-system` 的分支名会导致 review log 写入失败，因为 Claude Code 将多行 bash 块作为独立 shell 调用执行，变量在命令之间会丢失。新增的 `gstack-review-log` 和 `gstack-review-read` 原子助手将整个操作封装在单条命令中。
- **所有技能模板现在与平台无关。** 从 `/ship`、`/review`、`/plan-ceo-review` 和 `/plan-eng-review` 中移除了 Rails 特定的模式（`bin/test-lane`、`RAILS_ENV`、`.includes()`、`rescue StandardError` 等）。review 清单现在并排展示 Rails、Node、Python 和 Django 的示例。
- **`/ship` 通过读取 CLAUDE.md 来发现测试命令**，而不再硬编码 `bin/test-lane` 和 `npm run test`。如果未找到测试命令，会向用户询问并将答案持久化到 CLAUDE.md。

### 新增

- **平台无关设计原则**已写入 CLAUDE.md——技能必须读取项目配置，绝不硬编码框架命令。
- **`## Testing` 章节**已添加到 CLAUDE.md，供 `/ship` 发现测试命令使用。

## [0.8.4] - 2026-03-19

### 新增

- **`/ship` 现在自动同步你的文档。** 创建 PR 后，`/ship` 将在步骤 8.5 运行 `/document-release`——README、ARCHITECTURE、CONTRIBUTING 和 CLAUDE.md 无需额外命令即可保持最新状态。再也不会在发布后留下过时的文档了。
- **六个新技能已写入文档。** README、docs/skills.md 和 BROWSER.md 现在涵盖 `/codex`（多 AI 第二意见）、`/careful`（危险命令警告）、`/freeze`（目录级编辑锁定）、`/guard`（完整安全模式）、`/unfreeze` 和 `/gstack-upgrade`。冲刺技能表保留了 15 位专家；新的"强力工具"章节涵盖其余内容。
- **Browse 交接已在各处记录。** BROWSER.md 命令表、docs/skills.md 深度介绍以及 README 的"新功能"章节均说明了 `$B handoff` 和 `$B resume`，用于处理 CAPTCHA/MFA/认证障碍。
- **主动建议功能现已了解所有技能。** 根 SKILL.md.tmpl 现在在合适的工作流节点建议 `/codex`、`/careful`、`/freeze`、`/guard`、`/unfreeze` 和 `/gstack-upgrade`。

## [0.8.3] - 2026-03-19

### 新增

- **计划审查现在会引导你进行下一步。** 运行 `/plan-ceo-review`、`/plan-eng-review` 或 `/plan-design-review` 后，你会收到下一步操作的建议——工程审查始终作为必要的发布门槛，检测到 UI 变更时建议设计审查，大型产品变更时会温和提示 CEO 审查。再也不需要自己记住工作流了。
- **审查现在知道自己是否过期。** 每次审查都会记录其运行时的提交哈希。仪表盘将其与当前 HEAD 进行比较，并告知确切经过了多少次提交——显示"eng review 可能已过期——自审查以来已有 13 次提交"，而不是靠猜测。
- **`skip_eng_review` 在各处均受遵循。** 如果你已全局选择退出工程审查，链式建议不会再为此烦扰你。
- **设计审查简版现在也追踪提交。** 在 `/review` 和 `/ship` 中运行的轻量级设计检查获得了与完整审查相同的过期追踪功能。

### 修复

- **Browse 不再跳转到危险 URL。** `goto`、`diff` 和 `newtab` 现在会拦截 `file://`、`javascript:`、`data:` 协议以及云元数据端点（`169.254.169.254`、`metadata.google.internal`）。本地 QA 测试仍允许访问 localhost 和私有 IP。（关闭 #17）
- **setup 脚本现在告知你缺少什么。** 在未安装 `bun` 的情况下运行 `./setup`，现在会显示清晰的错误信息和安装说明，而不是神秘的"command not found"。（关闭 #147）
- **`/debug` 重命名为 `/investigate`。** Claude Code 有内置的 `/debug` 命令，会遮蔽 gstack 技能。系统化根因调试工作流现在位于 `/investigate`。（关闭 #190）
- **Shell 注入攻击面已消除。** 所有技能模板现在使用 `source <(gstack-slug)` 代替 `eval $(gstack-slug)`。行为相同，无需 `eval`。（关闭 #133）
- **新增 25 个安全测试。** URL 验证（16 个测试）和路径遍历验证（14 个测试）现在有专用单元测试套件，涵盖协议拦截、元数据 IP 拦截、目录逃逸和前缀碰撞边界情况。

## [0.8.2] - 2026-03-19

### 新增

- **当无头浏览器卡住时，可以切换到真实 Chrome。** 遇到 CAPTCHA、认证障碍或 MFA 提示？运行 `$B handoff "原因"`，一个可见的 Chrome 会在相同页面打开，保留所有 cookie 和标签页。解决问题后告诉 Claude 你已完成，`$B resume` 将从原处继续并获取新快照。
- **连续失败 3 次后自动提示切换。** 如果 browse 工具连续失败 3 次，它会建议使用 `handoff`——避免浪费时间看 AI 不断重试 CAPTCHA。
- **新增 15 个 handoff 功能测试。** 包含状态保存/恢复、失败追踪、边界情况的单元测试，以及完整的无头到有头流程（含 cookie 和标签页保留）的集成测试。

### 变更

- `recreateContext()` 重构为使用共享的 `saveState()`/`restoreState()` 助手——行为相同，代码更少，为未来的状态持久化功能做好准备。
- `browser.close()` 现在有 5 秒超时，防止在 macOS 上关闭有头浏览器时挂起。

## [0.8.1] - 2026-03-19

### 修复

- **`/qa` 不再拒绝在纯后端变更上使用浏览器。** 此前，如果你的分支只修改了提示模板、配置文件或服务逻辑，`/qa` 会分析差异，得出"没有 UI 需要测试"的结论，并建议运行 eval。现在它始终打开浏览器——当无法从差异中识别出具体页面时，退化为快速模式冒烟测试（主页 + 前 5 个导航目标）。

## [0.8.0] - 2026-03-19 — 多 AI 第二意见

**`/codex` — 从完全不同的 AI 获取独立的第二意见。**

三种模式。`/codex review` 对你的差异运行 OpenAI 的 Codex CLI 并给出通过/失败判定——如果 Codex 发现关键问题（`[P1]`），则失败。`/codex challenge` 采取对抗模式：它试图找出你的代码在生产环境中失败的方式，像攻击者和混沌工程师一样思考。`/codex <任何内容>` 打开与 Codex 关于你代码库的对话，具有会话连续性，后续问题可记住上下文。

当 `/review`（Claude）和 `/codex review` 都运行后，你会得到一份跨模型分析，显示哪些发现相互重叠、哪些是各 AI 独有的——帮助你建立对何时信任哪个系统的直觉。

**随处集成。** `/review` 完成后，它会提供 Codex 第二意见。在 `/ship` 期间，你可以在推送前将 Codex 审查作为可选门槛。在 `/plan-eng-review` 中，Codex 可以在工程审查开始前独立批评你的计划。所有 Codex 结果都显示在 Review Readiness Dashboard 中。

**本次发布还包含：** 主动技能建议——gstack 现在会注意你处于开发的哪个阶段并建议合适的技能。不喜欢？说"停止建议"，它会跨会话记住。

## [0.7.4] - 2026-03-18

### 变更

- **`/qa` 和 `/design-review` 现在会询问如何处理未提交的变更**，而不是拒绝启动。当你的工作区有未提交内容时，你会看到一个带有三个选项的交互提示：提交变更、暂存（stash）它们，或中止。不再出现"ERROR: Working tree is dirty"后跟一大段文字的情况。

## [0.7.3] - 2026-03-18

### 新增

- **一条命令即可开启安全护栏。** 说"be careful"或"safety mode"，`/careful` 会在任何危险命令前警告你——`rm -rf`、`DROP TABLE`、强制推送、`kubectl delete` 等等。每条警告都可以覆盖。常见的构建产物清理（`rm -rf node_modules`、`dist`、`.next`）已列入白名单。
- **用 `/freeze` 将编辑锁定到一个文件夹。** 调试某个问题，不想让 Claude 去"修复"不相关的代码？`/freeze` 会阻止在你选择的目录之外进行任何文件编辑。这是硬性拦截，不只是警告。运行 `/unfreeze` 可以取消限制而不需要结束会话。
- **`/guard` 同时激活两者。** 一条命令实现最大安全性，适用于接触生产环境或线上系统时——危险命令警告加上目录级编辑限制。
- **`/debug` 现在会自动冻结对被调试模块的编辑。** 形成根因假设后，`/debug` 会将编辑锁定在受影响的最小目录范围内。调试过程中不再意外"修复"不相关的代码。
- **你现在可以看到自己使用了哪些技能以及频率。** 每次技能调用都会本地记录到 `~/.gstack/analytics/skill-usage.jsonl`。运行 `bun run analytics` 查看你最常用的技能、按仓库细分以及安全钩子实际拦截的频率。数据保留在你的机器上。
- **每周 retro 现在包含技能使用情况。** `/retro` 会在通常的提交分析和指标旁边展示你在 retro 周期内使用了哪些技能。

## [0.7.2] - 2026-03-18

### 修复

- `/retro` 日期范围现在对齐到午夜，而不是当前时间。晚上 9 点运行 `/retro` 不再悄悄丢失开始日期当天上午的内容——你会得到完整的自然日。
- `/retro` 时间戳现在使用你的本地时区，而不是硬编码的太平洋时间。美国西海岸以外的用户在直方图、会话检测和连续天数追踪中可以看到正确的本地时间。

## [0.7.1] - 2026-03-19

### 新增

- **gstack 现在在自然时机建议技能。** 你不需要知道斜杠命令——只需谈论你正在做的事情。在头脑风暴？gstack 建议 `/office-hours`。什么东西坏了？它建议 `/debug`。准备部署？它建议 `/ship`。每个工作流技能现在都有在适当时机触发的主动触发器。
- **生命周期地图。** gstack 的根技能描述现在包含一份开发者工作流指南，将 12 个阶段（头脑风暴 → 计划 → 审查 → 编码 → 调试 → 测试 → 发布 → 文档 → 复盘）映射到合适的技能。Claude 在每次会话中都能看到这些内容。
- **用自然语言选择退出。** 如果主动建议感觉太频繁，只需说"stop suggesting things"——gstack 会跨会话记住。说"be proactive again"重新启用。
- **11 个旅程阶段 E2E 测试。** 每个测试模拟开发者生命周期中的真实时刻，具有真实的项目上下文（plan.md、错误日志、git 历史、代码），并验证合适的技能能否仅通过自然语言触发。11/11 通过。
- **触发词验证。** 静态测试验证每个工作流技能都有"Use when"和"Proactively suggest"短语——免费捕获回归。

### 修复

- `/debug` 和 `/office-hours` 对自然语言完全不可见——根本没有触发词。现在两者都有完整的响应式 + 主动式触发器。

## [0.7.0] - 2026-03-18 — YC Office Hours

**`/office-hours` — 在写一行代码之前，先和 YC 合伙人坐下来谈谈。**

两种模式。如果你在创业，你会得到六个从 YC 评估产品方式中提炼出来的强制性问题：需求现实、现状评估、迫切的具体性、最窄的切入点、观察与惊喜，以及未来适配性。如果你在做副项目、学习编程或参加黑客马拉松，你会得到一个充满热情的头脑风暴伙伴，帮你找到创意最酷的版本。

两种模式都会写出一份设计文档，直接输入到 `/plan-ceo-review` 和 `/plan-eng-review`。会话结束后，技能会反馈它对你思维方式的具体观察——是具体的发现，不是泛泛的称赞。

**`/debug` — 找到根因，而不是症状。**

当某个东西坏了但你不知道为什么，`/debug` 是你的系统化调试器。它遵循铁律：没有根因调查，不能修复。追踪数据流，与已知 bug 模式匹配（竞态条件、nil 传播、缓存过期、配置漂移），逐一验证假设。如果 3 次修复都失败，它会停下来质疑架构，而不是无休止地尝试。

## [0.6.4.1] - 2026-03-18

### 新增

- **技能现在可以通过自然语言发现。** 之前缺少明确触发词的 12 个技能现在都有了——说"deploy this"，Claude 会找到 `/ship`；说"check my diff"，它会找到 `/review`。遵循 Anthropic 的最佳实践："描述字段不是摘要——它是触发时机。"

## [0.6.4.0] - 2026-03-17

### 新增

- **`/plan-design-review` 现在是交互式的——评分 0-10，修复计划。** 设计师不再只是产出一份带字母评级的报告，现在的工作方式与 CEO 和工程审查相同：对每个设计维度评分 0-10，解释 10 分是什么样子，然后编辑计划以达到目标。每个设计选择一个 AskUserQuestion。输出是更好的计划，而不是关于计划的文档。
- **CEO 审查现在会邀请设计师参与。** 当 `/plan-ceo-review` 在计划中检测到 UI 范围时，它会激活设计与 UX 章节（第 11 节），涵盖信息架构、交互状态覆盖、AI 糟糕设计风险和响应式意图。对于深度设计工作，它建议使用 `/plan-design-review`。
- **15 个技能中的 14 个现在有完整测试覆盖（E2E + LLM 评判 + 验证）。** 为之前缺少 LLM 评判质量 eval 的 10 个技能添加了评判：ship、retro、qa-only、plan-ceo-review、plan-eng-review、plan-design-review、design-review、design-consultation、document-release、gstack-upgrade。为 gstack-upgrade 添加了真实 E2E 测试（之前是 `.todo`）。将 design-consultation 添加到命令验证中。
- **二分提交风格。** CLAUDE.md 现在要求每次提交都是单一逻辑变更——重命名与重写分开，测试基础设施与测试实现分开。

### 变更

- `/qa-design-review` 重命名为 `/design-review`——现在有了 `/plan-design-review` 计划模式，"qa-" 前缀会引起混淆。已在所有 22 个文件中更新。

## [0.6.3.0] - 2026-03-17

### 新增

- **每个涉及前端代码的 PR 现在都会自动获得设计审查。** `/review` 和 `/ship` 会对修改过的 CSS、HTML、JSX 和视图文件应用 20 条设计清单。能捕获 AI 糟糕设计模式（紫色渐变、3 列图标网格、通用英雄区文案）、排版问题（正文字号 < 16px、黑名单字体）、无障碍缺陷（`outline: none`）和 `!important` 滥用。机械性 CSS 修复会自动应用；需要设计判断的问题会先征询你的意见。
- **`gstack-diff-scope` 对分支中的变更进行分类。** 运行 `source <(gstack-diff-scope main)` 并获得 `SCOPE_FRONTEND=true/false`、`SCOPE_BACKEND`、`SCOPE_PROMPTS`、`SCOPE_TESTS`、`SCOPE_DOCS`、`SCOPE_CONFIG`。设计审查使用它在纯后端 PR 上静默跳过。发布前检查使用它在触及前端文件时建议设计审查。
- **设计审查出现在 Review Readiness Dashboard 中。** 仪表盘现在区分"LITE"（代码级，在 /review 和 /ship 中自动运行）和"FULL"（通过 /plan-design-review 进行的可视化审计，使用 browse 二进制）。两者都以设计审查条目形式显示。
- **设计审查检测的 E2E eval。** 植入了包含 7 个已知反模式的 CSS/HTML 固件（Papyrus 字体、14px 正文、`outline: none`、`!important`、紫色渐变、通用英雄区文案、3 列功能网格）。eval 验证 `/review` 能捕获其中至少 4 个。

## [0.6.2.0] - 2026-03-17

### 新增

- **计划审查现在像业内最优秀的人那样思考。** `/plan-ceo-review` 应用来自贝佐斯（单向门、第一天代理人怀疑主义）、格鲁夫（偏执扫描）、芒格（逆向思维）、霍洛维茨（战时意识）、切斯基/格雷厄姆（创始人模式）和奥特曼（杠杆强迫症）的 14 种认知模式。`/plan-eng-review` 应用来自拉尔森（团队状态诊断）、麦金利（默认保持无聊）、布鲁克斯（本质vs偶然复杂性）、贝克（先让变更简单）、梅杰斯（拥有生产中的代码）和 Google SRE（错误预算）的 15 种模式。`/plan-design-review` 应用来自拉姆斯（减法默认值）、诺曼（时间维度设计）、卓（有原则的品味）、盖比亚（为信任而设计，为旅程绘制故事板）和艾夫（关怀是可见的）的 12 种模式。
- **激活潜在空间，而非清单。** 认知模式通过点名具体的框架和人物，让 LLM 调用其对这些人实际思维方式的深层知识。指令是"内化这些，不要列举它们"——使每次审查都成为真正的视角转变，而不是更长的清单。

## [0.6.1.0] - 2026-03-17

### 新增

- **E2E 和 LLM 评判测试现在只运行你修改过的内容。** 每个测试声明它依赖哪些源文件。运行 `bun run test:e2e` 时，它会检查你的差异并跳过依赖文件未被触及的测试。只修改 `/retro` 的分支现在运行 2 个测试，而不是 31 个。使用 `bun run test:e2e:all` 强制运行所有测试。
- **`bun run eval:select` 预览哪些测试会运行。** 在花费 API 额度之前，精确查看你的差异会触发哪些测试。支持 `--json` 用于脚本化，支持 `--base <branch>` 覆盖基础分支。
- **完整性护栏捕获遗忘的测试条目。** 一个免费单元测试验证 E2E 和 LLM 评判测试文件中的每个 `testName` 在 TOUCHFILES 映射中都有对应条目。没有条目的新测试会立即让 `bun test` 失败——不再有静默的始终运行退化。

### 变更

- `test:evals` 和 `test:e2e` 现在基于差异自动选择（之前是：全部或不运行）
- 新增 `test:evals:all` 和 `test:e2e:all` 脚本用于显式全量运行

## 0.6.1 — 2026-03-17 — 煮沸整个湖

每个 gstack 技能现在都遵循**完整性原则**：当 AI 使边际成本接近于零时，始终推荐完整实现。当选项 A 只需多 70 行代码时，不再建议"选 B，因为它提供了 90% 的价值"。

阅读理念：https://garryslist.org/posts/boil-the-ocean

- **完整性评分**：每个 AskUserQuestion 选项现在显示完整性评分（1-10），偏向完整解决方案
- **双重时间估算**：工作量估算同时显示人工团队和 CC+gstack 时间（例如，"人工：约 2 周 / CC：约 1 小时"），并附有任务类型压缩参考表
- **反模式示例**：preamble 中具体的"不要这样做"示例库，使原则不再抽象
- **首次入门引导**：新用户会看到一次性介绍，链接到该文章，并提供在浏览器中打开的选项
- **审查完整性缺口**：`/review` 现在会标记捷径实现，当完整版本的 CC 时间成本 < 30 分钟时
- **Lake Score**：CEO 和工程审查完成摘要显示有多少建议选择了完整选项 vs 捷径
- **CEO + 工程审查双重时间**：时间询问、工作量估算和令人愉悦的机会都显示人工和 CC 两种时间尺度

## 0.6.0.1 — 2026-03-17

- **`/gstack-upgrade` 现在自动捕获过期的本地副本。** 如果你的全局 gstack 是最新的，但项目中的本地副本落后，`/gstack-upgrade` 会检测到不匹配并同步它。不再需要手动询问"我们是否有本地副本？"——它直接告诉你并提供更新。
- **升级同步更安全。** 如果同步本地副本时 `./setup` 失败，gstack 会从备份恢复之前的版本，而不是留下一个损坏的安装。

### 贡献者相关

- `gstack-upgrade/SKILL.md.tmpl` 中的独立使用章节现在引用步骤 2 和 4.5（DRY），而不是重复检测/同步 bash 块。新增了一个版本比较 bash 块。
- 独立模式的更新检查回退现在与 preamble 模式匹配（全局路径 → 本地路径 → `|| true`）。

## 0.6.0 — 2026-03-17

- **100% 测试覆盖是优质氛围编码的关键。** gstack 现在会从零开始引导测试框架，当你的项目没有测试框架时。检测你的运行时，研究最佳框架，请你选择，安装它，为你的实际代码编写 3-5 个真实测试，设置 CI/CD（GitHub Actions），创建 TESTING.md，并向 CLAUDE.md 添加测试文化指导。之后每次 Claude Code 会话都会自然地编写测试。
- **每个 bug 修复现在都会获得回归测试。** 当 `/qa` 修复一个 bug 并验证后，阶段 8e.5 会自动生成一个捕获该确切场景的回归测试。测试包含完整的归因追踪，可追溯到 QA 报告。自动递增的文件名防止跨会话冲突。
- **自信发布——覆盖率审计显示什么已测试、什么未测试。** `/ship` 步骤 3.4 从你的差异构建代码路径地图，搜索对应的测试，并生成带质量星级的 ASCII 覆盖率图（★★★ = 边界情况 + 错误处理，★★ = 正常路径，★ = 冒烟测试）。缺口会自动生成测试。PR 正文显示"Tests: 42 → 47 (+5 new)"。
- **你的复盘追踪测试健康状况。** `/retro` 现在显示测试文件总数、本周期新增测试数、回归测试提交数和趋势变化。如果测试比例低于 20%，它会将其标记为增长点。
- **设计审查也会生成回归测试。** `/qa-design-review` 阶段 8e.5 跳过仅 CSS 的修复（这些由重新运行设计审计捕获），但为 JavaScript 行为变更（如损坏的下拉菜单或动画失败）编写测试。

### 贡献者相关

- 向 `gen-skill-docs.ts` 添加了 `generateTestBootstrap()` 解析器（约 155 行）。在 RESOLVERS 映射中注册为 `{{TEST_BOOTSTRAP}}`。插入到 qa、ship（步骤 2.5）和 qa-design-review 模板中。
- 阶段 8e.5 回归测试生成已添加到 `qa/SKILL.md.tmpl`（46 行）和 `qa-design-review/SKILL.md.tmpl` 的 CSS 感知变体（12 行）。规则 13 修订为允许创建新测试文件。
- 步骤 3.4 测试覆盖率审计已添加到 `ship/SKILL.md.tmpl`（88 行），含质量评分标准和 ASCII 图表格式。
- 测试健康追踪已添加到 `retro/SKILL.md.tmpl`：3 个新数据收集命令、指标行、叙述章节、JSON 模式字段。
- `qa-only/SKILL.md.tmpl` 在未检测到测试框架时添加建议说明。
- `qa-report-template.md` 新增带延迟测试规格的回归测试章节。
- ARCHITECTURE.md 占位符表更新了 `{{TEST_BOOTSTRAP}}` 和 `{{REVIEW_DASHBOARD}}`。
- WebSearch 已添加到 qa、ship、qa-design-review 的允许工具列表中。
- 新增 26 个验证测试、2 个新 E2E eval（引导 + 覆盖率审计）。
- 新增 2 个 P3 TODO：非 GitHub 提供商的 CI/CD、自动升级弱测试。

## 0.5.4 — 2026-03-17

- **工程审查现在始终是完整审查。** `/plan-eng-review` 不再要求你在"大变更"和"小变更"模式之间选择。每个计划都会获得完整的交互式审查（架构、代码质量、测试、性能）。只有当复杂度检查实际触发时才会建议缩减范围——而不是作为常驻菜单选项。
- **发布后不会再重复询问已回答的审查问题。** 当 `/ship` 询问缺少审查并且你回答"照常发布"或"不相关"时，该决定会为当前分支保存。不再需要在每次预登陆修复后重新运行 `/ship` 时被反复询问。

### 贡献者相关

- 从 `plan-eng-review/SKILL.md.tmpl` 中删除了 SMALL_CHANGE / BIG_CHANGE / SCOPE_REDUCTION 菜单。范围缩减现在是主动的（由复杂度检查触发），而不是菜单项。
- 向 `ship/SKILL.md.tmpl` 添加了审查门控覆盖持久化——将 `ship-review-override` 条目写入 `$BRANCH-reviews.jsonl`，使后续 `/ship` 运行跳过该门控。
- 更新了 2 个 E2E 测试提示以匹配新流程。

## 0.5.3 — 2026-03-17

- **你始终掌控一切——即使在宏大构想时。** `/plan-ceo-review` 现在将每个范围扩展作为你单独选择加入的决策。EXPANSION 模式热情地推荐，但你对每个想法单独说是或否。不再有"代理自作主张添加了 5 个我没要求的功能"的情况。
- **新模式：SELECTIVE EXPANSION。** 将当前范围作为基准，但看看还有什么可能。代理逐一提出扩展机会，带有中立建议——你挑选值得做的。非常适合迭代现有功能，既想要严格性，又想被相邻改进所吸引。
- **你的 CEO 审查愿景被保存，而不是丢失。** 扩展想法、精选决策和 10x 愿景现在作为结构化设计文档持久化到 `~/.gstack/projects/{repo}/ceo-plans/`。过期计划自动归档。如果某个愿景特别出色，你可以将其提升到仓库的 `docs/designs/` 中供团队使用。

- **更智能的发布门控。** `/ship` 不再在 CEO 和设计审查不相关时烦扰你。工程审查是唯一必需的门控（即使这个也可以用 `gstack-config set skip_eng_review true` 禁用）。大型产品变更推荐 CEO 审查；UI 工作推荐设计审查。仪表盘仍然显示三者——只是不会因为可选项而阻止你。

### 贡献者相关

- 向 `plan-ceo-review/SKILL.md.tmpl` 添加了 SELECTIVE EXPANSION 模式，包含精选仪式、中立建议姿态和 HOLD SCOPE 基准。
- 重写了 EXPANSION 模式的步骤 0D，加入选择加入仪式——将愿景提炼为离散提案，每个作为 AskUserQuestion 呈现。
- 添加了 CEO 计划持久化（0D-POST 步骤）：带 YAML 前置内容的结构化 markdown（`status: ACTIVE/ARCHIVED/PROMOTED`）、范围决策表、归档流程。
- 在 Review Log 后添加了 `docs/designs` 提升步骤。
- 模式快速参考表扩展至 4 列。
- Review Readiness Dashboard：工程审查必需（可通过 `skip_eng_review` 配置覆盖），CEO/设计可选，由代理判断。
- 新测试：CEO 审查模式验证（4 种模式、持久化、提升）、SELECTIVE EXPANSION E2E 测试。

## 0.5.2 — 2026-03-17

- **你的设计顾问现在会承担创意风险。** `/design-consultation` 不只是提出一个安全、连贯的系统——它明确分解 SAFE CHOICES（类别基准）vs. RISKS（你的产品脱颖而出的地方）。你选择打破哪些规则。每个风险都附有为什么有效以及代价是什么的理由。
- **选择之前先了解全局。** 当你选择进行研究时，代理会使用截图和无障碍树分析浏览你所在领域的真实网站——而不只是网络搜索结果。你在做设计决策之前可以看到外面有什么。
- **预览看起来像你产品的页面。** 预览页面现在渲染真实的产品模型——带侧边栏导航和数据表格的仪表盘、带英雄区的营销页面、带表单的设置页面——而不只是字体样本和颜色调色板。

## 0.5.1 — 2026-03-17
- **发布前知道自己站在哪里。** 每个 `/plan-ceo-review`、`/plan-eng-review` 和 `/plan-design-review` 现在都将结果记录到审查追踪器中。每次审查结束时，你会看到一个 **Review Readiness Dashboard**，显示哪些审查已完成、何时运行以及是否通过——并有清晰的"CLEARED TO SHIP"或"NOT READY"判定。
- **`/ship` 在创建 PR 前检查你的审查。** 预检现在读取仪表盘，当审查缺失时询问是否继续。仅供参考——不会阻止你，但你会知道跳过了什么。
- **少复制粘贴一件事。** SLUG 计算（那个从 git remote 计算 `owner-repo` 的神秘 sed 管道）现在是共享的 `bin/gstack-slug` 助手。所有模板中 14 个内联副本已替换为 `source <(gstack-slug)`。如果格式有变，只需修改一处。
- **截图现在在 QA 和 browse 会话中可见。** gstack 截图时，现在会以可点击图片元素的形式显示在输出中——不再有你看不到的不可见 `/tmp/browse-screenshot.png` 路径。适用于 `/qa`、`/qa-only`、`/plan-design-review`、`/qa-design-review`、`/browse` 和 `/gstack`。

### 贡献者相关

- 向 `gen-skill-docs.ts` 添加了 `{{REVIEW_DASHBOARD}}` 解析器——注入到 4 个模板中的共享仪表盘读取器（3 个审查技能 + ship）。
- 添加了 `bin/gstack-slug` 助手（5 行 bash），附带单元测试。输出 `SLUG=` 和 `BRANCH=` 行，将 `/` 转义为 `-`。
- 新 TODO：智能审查相关性检测（P3）、用于审查门控 PR 合并的 `/merge` 技能（P2）。

## 0.5.0 — 2026-03-16

- **你的网站刚刚获得了设计审查。** `/plan-design-review` 打开你的网站，像高级产品设计师一样审查——排版、间距、层级、颜色、响应式、交互和 AI 糟糕设计检测。按类别获得字母评级（A-F），双重标题"Design Score"+"AI Slop Score"，以及不客气的结构化第一印象。
- **它也能修复发现的问题。** `/qa-design-review` 运行相同的设计师视角审计，然后在你的源代码中迭代修复设计问题，使用原子 `style(design):` 提交和前后对比截图。默认 CSS 安全，带有专为样式变更调整的更严格自律启发式规则。
- **了解你实际的设计系统。** 两个技能都通过 JS 提取你线上网站的字体、颜色、标题比例和间距模式——然后提供将推断的系统保存为 `DESIGN.md` 基准的选项。终于知道你实际使用了多少种字体。
- **AI 糟糕设计检测是头条指标。** 每份报告以两个分数开头：Design Score 和 AI Slop Score。AI 糟糕设计清单捕获 10 个最易识别的 AI 生成模式——3 列功能网格、紫色渐变、装饰性斑点、emoji 要点、通用英雄区文案。
- **设计回归追踪。** 报告写入 `design-baseline.json`。下次运行自动比较：按类别评级变化、新发现、已解决发现。随时间观察你的设计分数改善。
- **80 条设计审计清单**，跨 10 个类别：视觉层级、排版、颜色/对比度、间距/布局、交互状态、响应式、动效、内容/微文案、AI 糟糕设计和性能即设计。从 Vercel 的 100+ 条规则、Anthropic 的前端设计技能和其他 6 个设计框架中提炼而来。

### 贡献者相关

- 向 `gen-skill-docs.ts` 添加了 `{{DESIGN_METHODOLOGY}}` 解析器——注入到 `/plan-design-review` 和 `/qa-design-review` 模板中的共享设计审计方法论，遵循 `{{QA_METHODOLOGY}}` 模式。
- 将 `~/.gstack-dev/plans/` 添加为长期愿景文档的本地计划目录（不提交到仓库）。CLAUDE.md 和 TODOS.md 已更新。
- 向 TODOS.md 添加了 `/setup-design-md`（P2），用于从头开始交互式创建 DESIGN.md。

## 0.4.5 — 2026-03-16

- **审查发现现在真正得到修复，而不只是列出来。** `/review` 和 `/ship` 过去会打印信息性发现（死代码、测试缺口、N+1 查询），然后忽略它们。现在每个发现都会得到处理：明显的机械修复自动应用，真正模糊的问题汇总为一个问题而不是 8 个单独提示。每个自动修复你都能看到 `[AUTO-FIXED] file:line 问题 → 所做的事情`。
- **你掌控"直接修复"和"先问我"之间的界限。** 死代码、过期注释、N+1 查询会自动修复。安全问题、竞态条件、设计决策会浮现出来供你决定。分类标准集中在一个地方（`review/checklist.md`），使 `/review` 和 `/ship` 保持同步。

### 修复

- **`$B js "const x = await fetch(...); return x.status"` 现在可以工作了。** `js` 命令过去将所有内容作为表达式包装——所以 `const`、分号和多行代码都会出错。它现在检测语句并使用块包装，就像 `eval` 一样。
- **点击下拉选项不再永久挂起。** 如果代理在快照中看到 `@e3 [option] "Admin"` 并运行 `click @e3`，gstack 现在会自动选择该选项，而不是挂在不可能的 Playwright 点击上。正确的事情就这样发生了。
- **当点击是错误工具时，gstack 会告诉你。** 通过 CSS 选择器点击 `<option>` 过去会因神秘的 Playwright 错误而超时。现在你会得到：`"Use 'browse select' instead of 'click' for dropdown options."`

### 贡献者相关

- 门控分类 → 严重性分类重命名（严重性决定呈现顺序，而不是是否显示提示）。
- 修复优先启发式章节已添加到 `review/checklist.md`——权威的 AUTO-FIX vs ASK 分类标准。
- 新验证测试：`Fix-First Heuristic exists in checklist and is referenced by review + ship`。
- 在 `read-commands.ts` 中提取了 `needsBlockWrapper()` 和 `wrapForEvaluate()` 助手——由 `js` 和 `eval` 命令共享（DRY）。
- 向 `BrowserManager` 添加了 `getRefRole()`——为 ref 选择器暴露 ARIA 角色，而不改变 `resolveRef` 返回类型。
- 点击处理器自动将 `[role=option]` ref 路由到 `selectOption()`（通过父 `<select>`），带有 DOM `tagName` 检查以避免阻止自定义列表框组件。
- 6 个新测试：多行 js、分号、语句关键字、简单表达式、选项自动路由、CSS 选项错误引导。

## 0.4.4 — 2026-03-16

- **不到一小时就能检测到新版本，而不是半天。** 更新检查缓存设置为 12 小时，这意味着新版本发布时你可能整天都停留在旧版本上。现在"你是最新的"在 60 分钟后过期，所以你会在一小时内看到升级。"升级可用"仍然持续提示 12 小时（这就是目的）。
- **`/gstack-upgrade` 始终真实检查。** 直接运行 `/gstack-upgrade` 现在会绕过缓存并对 GitHub 进行新鲜检查。不再出现"你已经是最新版本"而实际上你不是的情况。

### 贡献者相关

- 拆分了 `last-update-check` 缓存 TTL：`UP_TO_DATE` 为 60 分钟，`UPGRADE_AVAILABLE` 为 720 分钟。
- 向 `bin/gstack-update-check` 添加了 `--force` 标志（在检查前删除缓存文件）。
- 3 个新测试：`--force` 清除 UP_TO_DATE 缓存、`--force` 清除 UPGRADE_AVAILABLE 缓存、带 `utimesSync` 的 60 分钟 TTL 边界测试。

## 0.4.3 — 2026-03-16

- **新增 `/document-release` 技能。** 在 `/ship` 后、合并前运行——它读取项目中的每个文档文件，与差异交叉对比，并更新 README、ARCHITECTURE、CONTRIBUTING、CHANGELOG 和 TODOS 以匹配你实际发布的内容。有风险的变更会作为问题浮现；其他一切都是自动的。
- **每次提问现在都清晰明了，每次都是。** 过去需要运行 3+ 次会话才能让 gstack 给你完整的上下文和简明的英文解释。现在每一个问题——即使在单次会话中——都会告诉你项目、分支和正在发生什么，解释得足够简单，即使在上下文切换中途也能理解。不再需要"抱歉，再用更简单的方式解释一下"。
- **分支名总是正确的。** gstack 现在在运行时检测你的当前分支，而不是依赖对话开始时的快照。中途切换分支？gstack 跟得上。

### 贡献者相关

- 将 ELI16 规则合并到基础 AskUserQuestion 格式中——一种格式而不是两种，没有 `_SESSIONS >= 3` 条件。
- 向 preamble bash 块添加了 `_BRANCH` 检测（`git branch --show-current`，带回退）。
- 为分支检测和简化规则添加了回归守卫测试。

## 0.4.2 — 2026-03-16

- **`$B js "await fetch(...)"` 现在直接可用。** `$B js` 或 `$B eval` 中的任何 `await` 表达式都会自动包装在异步上下文中。不再有 `SyntaxError: await is only valid in async functions`。单行 eval 文件直接返回值；多行文件使用显式 `return`。
- **贡献者模式现在反思，而不只是反应。** 贡献者模式不再只在出问题时提交报告，现在还会提示定期反思："给你的 gstack 体验打 0-10 分。不是 10 分？想想为什么。"捕获被动检测遗漏的生活质量问题和摩擦点。报告现在包含 0-10 评分和"什么能让这成为 10 分"，以聚焦于可行的改进。
- **技能现在尊重你的分支目标。** `/ship`、`/review`、`/qa` 和 `/plan-ceo-review` 会检测你的 PR 实际针对的分支，而不是假设是 `main`。堆叠分支、针对功能分支的 Conductor 工作区以及使用 `master` 的仓库现在都能正常工作。
- **`/retro` 适用于任何默认分支。** 使用 `master`、`develop` 或其他默认分支名的仓库会自动检测——不再因为分支名错误而出现空的 retro。
- **新 `{{BASE_BRANCH_DETECT}}` 占位符**，供技能作者使用——在任何模板中加入它，即可免费获得 3 步分支检测（PR 基础 → 仓库默认 → 回退）。
- **3 个新 E2E 冒烟测试**，验证基础分支检测在 ship、review 和 retro 技能中端到端正常工作。

### 贡献者相关

- 添加了带注释剥离的 `hasAwait()` 助手，以避免 eval 文件中 `// await` 的假阳性。
- 智能 eval 包装：单行 → 表达式 `(...)`，多行 → 块 `{...}`，带显式 `return`。
- 6 个新异步包装单元测试，40 个新贡献者模式 preamble 验证测试。
- 校准示例框架为历史性的（"过去会失败"），以避免在修复后暗示存在活跃 bug。
- 向 CLAUDE.md 添加了"编写 SKILL 模板"章节——自然语言优先于 bash 惯用法、动态分支检测、自包含代码块的规则。
- 硬编码 main 回归测试扫描所有 `.tmpl` 文件中带硬编码 `main` 的 git 命令。
- QA 模板清理：删除了 `REPORT_DIR` shell 变量，将端口检测简化为散文。
- gstack-upgrade 模板：为 bash 块之间的变量引用添加了明确的跨步骤散文。

## 0.4.1 — 2026-03-16

- **gstack 现在会注意到自己搞砸了。** 开启贡献者模式（`gstack-config set gstack_contributor true`），gstack 会自动写下出了什么问题——你在做什么、什么坏了、复现步骤。下次某事让你恼火时，bug 报告已经写好了。Fork gstack 自己修复它。
- **同时管理多个会话？gstack 跟得上。** 当你有 3+ 个 gstack 窗口打开时，每个问题现在都会告诉你是哪个项目、哪个分支，以及你在做什么。不再盯着一个问题想"等等，这是哪个窗口？"
- **每个问题现在都附带建议。** 与其把选项堆给你让你思考，gstack 告诉你它会选什么以及为什么。所有技能都使用相同的清晰格式。
- **`/review` 现在捕获遗忘的枚举处理器。** 添加了新的状态、级别或类型常量？`/review` 会在代码库中的每个 switch 语句、允许列表和过滤器中追踪它——不只是你修改过的文件。在发布前捕获"添加了值但忘记处理它"这类 bug。

### 贡献者相关

- 在所有 11 个技能模板中将 `{{UPDATE_CHECK}}` 重命名为 `{{PREAMBLE}}`——一个启动块现在处理更新检查、会话追踪、贡献者模式和问题格式化。
- 将 plan-ceo-review 和 plan-eng-review 的问题格式化简化为引用 preamble 基准，而不是重复规则。
- 向 CLAUDE.md 添加了 CHANGELOG 风格指南和本地副本意识文档。

## 0.4.0 — 2026-03-16

### 新增
- **QA-only 技能**（`/qa-only`）——仅报告的 QA 模式，只查找和记录 bug，不进行修复。向团队交付一份干净的 bug 报告，代理不会触碰你的代码。
- **QA 修复循环**——`/qa` 现在运行查找-修复-验证循环：发现 bug、修复它们、提交、重新导航确认修复已生效。一条命令从损坏到发布。
- **计划到 QA 的制品流**——`/plan-eng-review` 写入测试计划制品，`/qa` 自动拾取。你的工程审查现在直接输入 QA 测试，无需手动复制粘贴。
- **`{{QA_METHODOLOGY}}` DRY 占位符**——注入到 `/qa` 和 `/qa-only` 模板的共享 QA 方法论块。在你更新测试标准时保持两个技能同步。
- **Eval 效率指标**——轮次、时长和费用现在在所有 eval 界面显示，附带自然语言**Takeaway**评论。一目了然地看到你的提示变更是让代理更快还是更慢。
- **`generateCommentary()` 引擎**——解释比较增量，让你无需手动理解：标记回归、注明改进，并生成整体效率摘要。
- **Eval 列表列**——`bun run eval:list` 现在显示每次运行的轮次和时长。立即发现昂贵或缓慢的运行。
- **Eval 摘要每测试效率**——`bun run eval:summary` 显示跨运行的每个测试的平均轮次/时长/费用。识别哪些测试随时间消耗最多。
- **`judgePassed()` 单元测试**——提取并测试了通过/失败判断逻辑。
- **3 个新 E2E 测试**——qa-only 禁止修复守卫、带提交验证的 qa 修复循环、plan-eng-review 测试计划制品。
- **浏览器 ref 过期检测**——`resolveRef()` 现在检查元素数量以检测页面变更后的过期 ref。SPA 导航不再导致缺少元素的 30 秒超时。
- 3 个新的 ref 过期快照测试。

### 变更
- QA 技能提示重构为明确的两阶段工作流（查找 → 修复 → 验证）。
- `formatComparison()` 现在在费用旁边显示每个测试的轮次和时长增量。
- `printSummary()` 显示轮次和时长列。
- `eval-store.test.ts` 修复了预先存在的 `_partial` 文件断言 bug。

### 修复
- 浏览器 ref 过期——页面变更（例如 SPA 导航）前收集的 ref 现在被检测并重新收集。消除了一类动态网站上的不稳定 QA 失败。

## 0.3.9 — 2026-03-15

### 新增
- **`bin/gstack-config` CLI**——`~/.gstack/config.yaml` 的简单 get/set/list 界面。由更新检查和升级技能用于持久化设置（auto_upgrade、update_check）。
- **智能更新检查**——12 小时缓存 TTL（之前是 24 小时），用户拒绝升级时指数退避暂停（24 小时 → 48 小时 → 1 周），`update_check: false` 配置选项可完全禁用检查。发布新版本时暂停重置。
- **自动升级模式**——在配置中设置 `auto_upgrade: true` 或设置 `GSTACK_AUTO_UPGRADE=1` 环境变量，跳过升级提示并自动更新。
- **4 选项升级提示**——"是，立即升级"、"始终保持最新"、"暂时不（暂停）"、"不再询问（禁用）"。
- **本地副本同步**——`/gstack-upgrade` 现在在升级主安装后检测并更新当前项目中的本地本地副本。
- 25 个新测试：gstack-config CLI 11 个，更新检查中暂停/配置路径 14 个。

### 变更
- README 升级/故障排除章节简化，引用 `/gstack-upgrade` 而不是长段粘贴命令。
- 升级技能模板升级到 v1.1.0，带有 `Write` 工具权限用于配置编辑。
- 所有 SKILL.md preamble 更新了新的升级流程描述。

## 0.3.8 — 2026-03-14

### 新增
- **TODOS.md 作为单一事实来源**——将 `TODO.md`（路线图）和 `TODOS.md`（近期）合并为一个文件，按技能/组件组织，P0-P4 优先级排序，并有已完成章节。
- **`/ship` 步骤 5.5：TODOS.md 管理**——从差异中自动检测已完成项目，用版本注释标记为已完成，如果 TODOS.md 缺失或结构混乱，提供创建/重组选项。
- **跨技能 TODOS 感知**——`/plan-ceo-review`、`/plan-eng-review`、`/retro`、`/review` 和 `/qa` 现在读取 TODOS.md 获取项目上下文。`/retro` 添加了积压健康指标（未完成数、P0/P1 项目、波动）。
- **共享 `review/TODOS-format.md`**——被 `/ship` 和 `/plan-ceo-review` 引用的权威 TODO 项格式，防止格式漂移（DRY）。
- **Greptile 两层回复系统**——第一层（友好，内联差异 + 解释）用于首次回复；第二层（坚定，完整证据链 + 重新排名请求）用于 Greptile 在之前回复后重新标记时。
- **Greptile 回复模板**——`greptile-triage.md` 中的结构化模板，用于修复（内联差异）、已修复（所做的事情）和误报（证据 + 建议重新排名）。替代含糊的一行回复。
- **Greptile 升级检测**——明确的算法检测评论线程上的先前 GStack 回复并自动升级到第二层。
- **Greptile 严重性重新排名**——当 Greptile 错误分类问题严重性时，回复现在包含 `**Suggested re-rank:**`。
- `TODOS-format.md` 跨技能引用的静态验证测试。

### 修复
- **`.gitignore` 追加失败被静默吞噬**——`ensureStateDir()` 裸 `catch {}` 替换为仅 ENOENT 静默；非 ENOENT 错误（EACCES、ENOSPC）记录到 `.gstack/browse-server.log`。

### 变更
- `TODO.md` 已删除——所有项目合并到 `TODOS.md`。
- `/ship` 步骤 3.75 和 `/review` 步骤 5 现在引用 `greptile-triage.md` 中的回复模板和升级检测。
- `/ship` 步骤 6 提交顺序在最终提交中包含 TODOS.md（与 VERSION + CHANGELOG 一起）。
- `/ship` 步骤 8 PR 正文包含 TODOS 章节。

## 0.3.7 — 2026-03-14

### 新增
- **截图元素/区域裁剪**——`screenshot` 命令现在支持通过 CSS 选择器或 @ref 进行元素裁剪（`screenshot "#hero" out.png`、`screenshot @e3 out.png`），区域裁剪（`screenshot --clip x,y,w,h out.png`），以及仅视口模式（`screenshot --viewport out.png`）。使用 Playwright 的原生 `locator.screenshot()` 和 `page.screenshot({ clip })`。全页截图仍为默认。
- 10 个新测试覆盖所有截图模式（视口、CSS、@ref、裁剪）和错误路径（未知标志、互斥、无效坐标、路径验证、不存在的选择器）。

## 0.3.6 — 2026-03-14

### 新增
- **E2E 可观测性**——心跳文件（`~/.gstack-dev/e2e-live.json`）、每次运行日志目录（`~/.gstack-dev/e2e-runs/{runId}/`）、progress.log、每个测试的 NDJSON 转录、持久化失败转录。所有 I/O 非致命。
- **`bun run eval:watch`**——实时终端仪表盘每秒读取心跳 + 部分 eval 文件。显示已完成测试、当前测试（含轮次/工具信息）、过期检测（>10 分钟）、`--tail` 用于 progress.log。
- **增量 eval 保存**——`savePartial()` 在每个测试完成后写入 `_partial-e2e.json`。崩溃弹性：部分结果在运行终止时仍然存在。永不清理。
- **机器可读诊断**——eval JSON 中的 `exit_reason`、`timeout_at_turn`、`last_tool_call` 字段。支持 `jq` 查询用于自动化修复循环。
- **API 连接性预检**——E2E 套件在 ConnectionRefused 时立即抛出，在燃烧测试预算之前。
- **`is_error` 检测**——`claude -p` 可以返回 `subtype: "success"` 同时带有 `is_error: true` 表示 API 失败。现在正确分类为 `error_api`。
- **Stream-json NDJSON 解析器**——`parseNDJSON()` 纯函数，用于从 `claude -p --output-format stream-json --verbose` 获取实时 E2E 进度。
- **Eval 持久化**——结果保存到 `~/.gstack-dev/evals/`，自动与上一次运行比较。
- **Eval CLI 工具**——`eval:list`、`eval:compare`、`eval:summary` 用于检查 eval 历史。
- **所有 9 个技能转换为 `.tmpl` 模板**——plan-ceo-review、plan-eng-review、retro、review、ship 现在使用 `{{UPDATE_CHECK}}` 占位符。更新检查 preamble 的单一事实来源。
- **3 层 eval 套件**——第 1 层：静态验证（免费），第 2 层：通过 `claude -p` 的 E2E（约 3.85 美元/次），第 3 层：LLM 评判（约 0.15 美元/次）。由 `EVALS=1` 控制。
- **植入 bug 结果测试**——带已知 bug 的 eval 固件，LLM 评判检测分数。
- 15 个可观测性单元测试，涵盖心跳模式、progress.log 格式、NDJSON 命名、savePartial、finalize、watcher 渲染、过期检测、非致命 I/O。
- plan-ceo-review、plan-eng-review、retro 技能的 E2E 测试。
- 更新检查退出码回归测试。
- `test/helpers/skill-parser.ts`——用于 git remote 检测的 `getRemoteSlug()`。

### 修复
- **Browse 二进制发现对代理损坏**——用 `browse/dist/browse` 显式路径替换 `find-browse` 间接寻址，在 SKILL.md 设置块中。
- **更新检查退出码 1 误导代理**——添加 `|| true` 防止在无可用更新时非零退出。
- **browse/SKILL.md 缺少设置块**——添加了 `{{BROWSE_SETUP}}` 占位符。
- **plan-ceo-review 超时**——在测试目录中初始化 git 仓库，跳过代码库探索，将超时提升至 420 秒。
- 植入 bug eval 可靠性——简化提示、降低检测基准、对 max_turns 抖动有弹性。

### 变更
- **模板系统扩展**——`gen-skill-docs.ts` 中的 `{{UPDATE_CHECK}}` 和 `{{BROWSE_SETUP}}` 占位符。所有使用 browse 的技能从单一事实来源生成。
- 丰富了 14 个命令描述，包含具体的参数格式、有效值、错误行为和返回类型。
- 设置块首先检查工作区本地路径（用于开发），回退到全局安装。
- LLM eval 评判从 Haiku 升级到 Sonnet 4.6。
- `generateHelpText()` 从 COMMAND_DESCRIPTIONS 自动生成（替代手工维护的帮助文本）。

## 0.3.3 — 2026-03-13

### 新增
- **SKILL.md 模板系统**——带 `{{COMMAND_REFERENCE}}` 和 `{{SNAPSHOT_FLAGS}}` 占位符的 `.tmpl` 文件，在构建时从源代码自动生成。从结构上防止文档和代码之间的命令漂移。
- **命令注册表**（`browse/src/commands.ts`）——所有 browse 命令的单一事实来源，带分类和丰富描述。无副作用，可安全从构建脚本和测试中导入。
- **快照标志元数据**（`browse/src/snapshot.ts` 中的 `SNAPSHOT_FLAGS` 数组）——元数据驱动的解析器替代手工编写的 switch/case。在一处添加标志可更新解析器、文档和测试。
- **第 1 层静态验证**——43 个测试：从 SKILL.md 代码块解析 `$B` 命令，对照命令注册表和快照标志元数据验证。
- **第 2 层 E2E 测试**，通过 Agent SDK——生成真实 Claude 会话、运行技能、扫描 browse 错误。由 `SKILL_E2E=1` 环境变量控制（约 0.50 美元/次）。
- **第 3 层 LLM 评判 eval**——Haiku 对生成文档的清晰度/完整性/可操作性评分（阈值 ≥4/5），加上与手工维护基准的回归测试。由 `ANTHROPIC_API_KEY` 控制。
- **`bun run skill:check`**——健康仪表盘，显示所有技能、命令数、验证状态、模板新鲜度。
- **`bun run dev:skill`**——监视模式，在每次模板或源文件变更时重新生成并验证 SKILL.md。
- **CI 工作流**（`.github/workflows/skill-docs.yml`）——在 push/PR 上运行 `gen:skill-docs`，如果生成的输出与已提交的文件不同则失败。
- 用于手动重新生成的 `bun run gen:skill-docs` 脚本。
- 用于 LLM 评判 eval 的 `bun run test:eval`。
- `test/helpers/skill-parser.ts`——从 Markdown 提取并验证 `$B` 命令。
- `test/helpers/session-runner.ts`——带错误模式扫描和转录保存的 Agent SDK 包装器。
- **ARCHITECTURE.md**——设计决策文档，涵盖守护进程模型、安全性、ref 系统、日志记录、崩溃恢复。
- **Conductor 集成**（`conductor.json`）——工作区设置/拆除的生命周期钩子。
- **`.env` 传播**——`bin/dev-setup` 自动将 `.env` 从主工作树复制到 Conductor 工作区。
- 用于 API 密钥配置的 `.env.example` 模板。

### 变更
- 构建现在在编译二进制文件之前运行 `gen:skill-docs`。
- `parseSnapshotArgs` 是元数据驱动的（迭代 `SNAPSHOT_FLAGS` 而不是 switch/case）。
- `server.ts` 从 `commands.ts` 导入命令集，而不是内联声明。
- SKILL.md 和 browse/SKILL.md 现在是生成的文件（改为编辑 `.tmpl`）。

## 0.3.2 — 2026-03-13

### 修复
- Cookie 导入选择器现在返回 JSON 而不是 HTML——`jsonResponse()` 引用了作用域外的 `url`，导致每次 API 调用崩溃。
- `help` 命令正确路由（由于 META_COMMANDS 调度顺序而无法访问）。
- 全局安装的过期服务器不再遮蔽本地变更——从 `resolveServerScript()` 中删除了遗留的 `~/.claude/skills/gstack` 回退。
- 崩溃日志路径引用从 `/tmp/` 更新到 `.gstack/`。

### 新增
- **差异感知 QA 模式**——功能分支上的 `/qa` 自动分析 `git diff`，识别受影响的页面/路由，检测 localhost 上运行的应用，只测试变更的内容。不需要 URL。
- **项目本地 browse 状态**——状态文件、日志和所有服务器状态现在位于项目根目录的 `.gstack/` 中（通过 `git rev-parse --show-toplevel` 检测）。不再有 `/tmp` 状态文件。
- **共享配置模块**（`browse/src/config.ts`）——集中化 CLI 和服务器的路径解析，消除重复的端口/状态逻辑。
- **随机端口选择**——服务器从 10000-60000 中选择随机端口，而不是扫描 9400-9409。不再需要 CONDUCTOR_PORT 魔法偏移量。不再有跨工作区的端口冲突。
- **二进制版本追踪**——状态文件包含 `binaryVersion` SHA；二进制重建时 CLI 自动重启服务器。
- **遗留 /tmp 清理**——CLI 扫描并删除旧的 `/tmp/browse-server*.json` 文件，在发送信号前验证 PID 所有权。
- **Greptile 集成**——`/review` 和 `/ship` 获取并分类 Greptile 机器人评论；`/retro` 跨周追踪 Greptile 命中率。
- **本地开发模式**——`bin/dev-setup` 从仓库符号链接技能用于原地开发；`bin/dev-teardown` 恢复全局安装。
- `help` 命令——代理可以自行发现所有命令和快照标志。
- 带 META 信号协议的版本感知 `find-browse`——检测过期二进制文件并提示代理更新。
- `browse/dist/find-browse` 编译二进制文件，带与 origin/main 的 git SHA 比较（4 小时缓存）。
- 构建时写入 `.version` 文件用于二进制版本追踪。
- cookie 选择器（13 个测试）和 find-browse 版本检查（10 个测试）的路由级测试。
- 配置解析测试（14 个测试），涵盖 git 根目录检测、BROWSE_STATE_FILE 覆盖、ensureStateDir、readVersionHash、resolveServerScript 和版本不匹配检测。
- CLAUDE.md 中的浏览器交互指南——防止 Claude 使用 mcp\_\_claude-in-chrome\_\_\* 工具。
- CONTRIBUTING.md，包含快速入门、开发模式说明和在其他仓库中测试分支的说明。

### 变更
- 状态文件位置：`.gstack/browse.json`（之前是 `/tmp/browse-server.json`）。
- 日志文件位置：`.gstack/browse-{console,network,dialog}.log`（之前是 `/tmp/browse-*.log`）。
- 原子状态文件写入：`.json.tmp` → 重命名（防止部分读取）。
- CLI 将 `BROWSE_STATE_FILE` 传递给生成的服务器（服务器从中派生所有路径）。
- SKILL.md 设置块解析 META 信号并处理 `META:UPDATE_AVAILABLE`。
- `/qa` SKILL.md 现在描述四种模式（差异感知、完整、快速、回归），差异感知为功能分支上的默认模式。
- `jsonResponse`/`errorResponse` 使用选项对象以防止位置参数混淆。
- 构建脚本编译 `browse` 和 `find-browse` 两个二进制文件，清理 `.bun-build` 临时文件。
- README 更新了 Greptile 设置说明、差异感知 QA 示例和修订的演示转录。

### 删除
- `CONDUCTOR_PORT` 魔法偏移量（`browse_port = CONDUCTOR_PORT - 45600`）。
- 端口扫描范围 9400-9409。
- 遗留回退到 `~/.claude/skills/gstack/browse/src/server.ts`。
- `DEVELOPING_GSTACK.md`（重命名为 CONTRIBUTING.md）。

## 0.3.1 — 2026-03-12

### 阶段 3.5：浏览器 cookie 导入

- `cookie-import-browser` 命令——从真实 Chromium 浏览器（Comet、Chrome、Arc、Brave、Edge）解密并导入 cookie。
- 从 browse 服务器提供的交互式 cookie 选择器 Web UI（深色主题、双面板布局、域名搜索、导入/删除）。
- 带 `--domain` 标志的直接 CLI 导入，用于非交互式使用。
- 用于 Claude Code 集成的 `/setup-browser-cookies` 技能。
- 带异步 10 秒超时的 macOS Keychain 访问（无事件循环阻塞）。
- 每浏览器 AES 密钥缓存（每个浏览器每次会话一次 Keychain 提示）。
- DB 锁回退：将锁定的 cookie DB 复制到 /tmp 以安全读取。
- 18 个带加密 cookie 固件的单元测试。

## 0.3.0 — 2026-03-12

### 阶段 3：/qa 技能——系统化 QA 测试

- 带 6 阶段工作流（初始化、认证、定向、探索、记录、收尾）的新 `/qa` 技能。
- 三种模式：完整（系统化，5-10 个问题）、快速（30 秒冒烟测试）、回归（与基准比较）。
- 问题分类：7 个类别、4 个严重级别、每页探索清单。
- 带健康评分（0-100，跨 7 个类别加权）的结构化报告模板。
- Next.js、Rails、WordPress 和 SPA 的框架检测指南。
- `browse/bin/find-browse`——使用 `git rev-parse --show-toplevel` 的 DRY 二进制发现。

### 阶段 2：增强浏览器

- 对话框处理：自动接受/拒绝、对话框缓冲区、提示文本支持。
- 文件上传：`upload <sel> <file1> [file2...]`。
- 元素状态检查：`is visible|hidden|enabled|disabled|checked|editable|focused <sel>`。
- 带 ref 标签叠加的标注截图（`snapshot -a`）。
- 与上一个快照对比（`snapshot -D`）。
- 光标交互元素扫描，用于非 ARIA 可点击元素（`snapshot -C`）。
- `wait --networkidle` / `--load` / `--domcontentloaded` 标志。
- `console --errors` 过滤器（仅错误 + 警告）。
- 带从页面 URL 自动填充域名的 `cookie-import <json-file>`。
- 控制台/网络/对话框缓冲区的 CircularBuffer O(1) 环形缓冲区。
- 使用 Bun.write() 的异步缓冲区刷新。
- 带 page.evaluate + 2 秒超时的健康检查。
- Playwright 错误包装——为 AI 代理提供可操作的消息。
- 上下文重建保留 cookie/存储/URL（useragent 修复）。
- SKILL.md 重写为 QA 导向的操作手册，包含 10 个工作流模式。
- 166 个集成测试（之前约 63 个）。

## 0.0.2 — 2026-03-12

- 修复项目本地 `/browse` 安装——编译后的二进制文件现在从自己的目录解析 `server.ts`，而不是假设全局安装存在。
- `setup` 重建过期的二进制文件（不只是缺失的）并在构建失败时以非零退出。
- 修复 `chain` 命令吞噬写命令的真实错误（例如，导航超时报告为"Unknown meta command"）。
- 修复 CLI 在服务器在同一命令上反复崩溃时的无限重启循环。
- 将控制台/网络缓冲区上限设为 50k 条目（环形缓冲区），而不是无限增长。
- 修复缓冲区达到 50k 上限后磁盘刷新静默停止。
- 修复 setup 中的 `ln -snf` 以避免在升级时创建嵌套符号链接。
- 升级使用 `git fetch && git reset --hard` 而不是 `git pull`（处理强制推送）。
- 简化安装：全局优先，可选项目副本（替代子模块方式）。
- 重构 README：英雄区、前后对比、演示转录、故障排除章节。
- 六个技能（新增 `/retro`）。

## 0.0.1 — 2026-03-11

初始版本。

- 五个技能：`/plan-ceo-review`、`/plan-eng-review`、`/review`、`/ship`、`/browse`。
- 带 40+ 命令、基于 ref 的交互、持久化 Chromium 守护进程的无头浏览器 CLI。
- 作为 Claude Code 技能的一键安装（子模块或全局克隆）。
- 用于二进制编译和技能符号链接的 `setup` 脚本。
