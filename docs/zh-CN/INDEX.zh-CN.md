# gstack 中文文档索引

> 本目录包含 gstack 项目所有核心文档的中文翻译。

## 从这里开始

**如果你是第一次接触 gstack，推荐按以下顺序阅读：**

1. [综合学习指南](LEARNING-GUIDE.zh-CN.md) — 从零开始的系统学习路径
2. [README](README.zh-CN.md) — 项目介绍和快速上手
3. [技能深度解析](skills.zh-CN.md) — 每个技能的详细说明

## 文档列表

| 文档 | 原文 | 内容 |
|------|------|------|
| [综合学习指南](LEARNING-GUIDE.zh-CN.md) | *原创* | 从零开始的系统学习路径，包含架构解析和使用技巧 |
| [README](README.zh-CN.md) | [README.md](../../README.md) | 项目介绍、安装方法、快速上手、功能概览 |
| [架构文档](ARCHITECTURE.zh-CN.md) | [ARCHITECTURE.md](../../ARCHITECTURE.md) | 系统设计决策、守护进程模型、安全架构、ref 系统、模板系统、E2E 测试基础设施 |
| [浏览器参考](BROWSER.zh-CN.md) | [BROWSER.md](../../BROWSER.md) | Browse CLI 完整命令参考、工作原理、快照系统、认证、对话框处理、多工作区支持 |
| [贡献指南](CONTRIBUTING.zh-CN.md) | [CONTRIBUTING.md](../../CONTRIBUTING.md) | 开发环境搭建、贡献者模式、三层测试、SKILL.md 编辑、Conductor 工作区 |
| [技能深度解析](skills.zh-CN.md) | [docs/skills.md](../skills.md) | 每个技能的哲学、工作流和示例（含 Greptile 集成） |
| [更新日志](CHANGELOG.zh-CN.md) | [CHANGELOG.md](../../CHANGELOG.md) | 每个版本的新功能和修复 |

## 关键概念速查

| 概念 | 说明 | 相关文档 |
|------|------|---------|
| Skill（技能） | 写给 AI 的结构化 prompt，控制 Claude 的行为 | [学习指南 §3](LEARNING-GUIDE.zh-CN.md#三skill-系统详解) |
| Browse CLI | 编译为二进制的无头浏览器工具，~100ms/命令 | [浏览器参考](BROWSER.zh-CN.md) |
| Snapshot（快照） | 可访问性树 + ref 引用系统，AI 操作页面的核心方式 | [架构文档 §ref系统](ARCHITECTURE.zh-CN.md) |
| Template（模板） | `.tmpl` 文件 + `{{占位符}}`，从源码生成文档 | [学习指南 §3.3](LEARNING-GUIDE.zh-CN.md#33-模板驱动的生成系统) |
| Preamble（前导块） | 所有 skill 共享的头部代码，处理更新检查/会话追踪/分析 | [架构文档 §preamble](ARCHITECTURE.zh-CN.md) |
| Fix-First | Review 模式：机械问题直接修，判断问题问用户 | [学习指南 §3.4c](LEARNING-GUIDE.zh-CN.md#c-fix-first-模式) |
| Handoff（人机切换） | AI 遇到障碍时交给用户，用户完成后 AI 继续 | [浏览器参考 §handoff](BROWSER.zh-CN.md) |
| Touchfiles | 测试依赖声明，实现 diff-based 智能测试选择 | [贡献指南 §testing](CONTRIBUTING.zh-CN.md) |

## 常用命令

```bash
# 安装
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup

# 开发
bun install          # 安装依赖
bun test             # 运行免费测试
bun run build        # 构建（生成文档 + 编译二进制）
bun run gen:skill-docs  # 重新生成 SKILL.md
bun run skill:check  # 技能健康检查面板
bun run dev:skill    # 监听模式（修改后自动重新生成和校验）

# 测试（付费）
bun run test:evals   # LLM 评判 + E2E（基于 diff，~$4/次）
bun run test:e2e     # 仅 E2E 测试

# 评估工具
bun run eval:list    # 列出所有评估记录
bun run eval:compare # 比较两次评估
bun run eval:summary # 汇总统计
```
