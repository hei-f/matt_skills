# Matt Pocock Skills 仓库 - 完整索引

> 本报告基于 [Matt Pocock 的 Skills 仓库](https://github.com/mattpocock/skills) 分析生成。
> 这是 Matt 日常使用的 Agent Skills 集合，专为**真正的工程开发**（而非"氛围编程"）而设计。

---

## 仓库概述

Matt Pocock 将这些 Skills 设计为**小型、易于适配、可组合**的工具。它们适用于任何模型，基于数十年的工程经验。所有 Skills 分为以下几个桶（bucket）：

| 桶 | 说明 |
|---|---|
| `engineering/` | 日常编码工作中使用的 Skills |
| `productivity/` | 通用工作流工具，不限于编码 |
| `misc/` | 保留但很少使用，不在插件中推广 |
| `in-progress/` | Beta 阶段，公开征求反馈，可能随时变更 |
| `deprecated/` | 已弃用（当前为空） |

Skills 按调用方式分为两种：
- **User-invoked（用户调用）**：只能由用户手动输入触发（如 `/grill-me`），负责编排流程
- **Model-invoked（模型调用）**：既可以由用户调用，也可以由 AI Agent 根据任务自动触发

---

## 一、Engineering（工程类 Skills）

日常编码工作中使用的核心 Skills。

### 用户调用（User-invoked）

| # | Skill | 简要描述 | 详细报告 |
|---|-------|---------|---------|
| 1 | **ask-matt（trash）** | 询问哪个 Skill 或流程适合你的情况，是本仓库用户调用 Skills 的路由器 | [→ 详细报告](engineering/ask-matt/report.md) |
| 3 | **triage** | 通过状态机将 Issue 在分诊角色之间流转 | [→ 详细报告](engineering/triage/report.md) |
| 5 | **setup-matt-pocock-skills（trash）** | 为工程 Skills 配置仓库（Issue 追踪器、分诊标签、领域文档布局），每个仓库只需运行一次 | [→ 详细报告](engineering/setup-matt-pocock-skills/report.md) |
| 7 | **to-tickets** | 将任何计划、Spec 或对话分解为一组追踪子弹（tracer-bullet）工单，每个工单声明其阻塞边 | [→ 详细报告](engineering/to-tickets/report.md) |
| 8 | **implement** | 按照 Spec 或工单集合实现工作，在预先约定的边界上驱动 `/tdd`，提交前以 `/code-review` 收尾 | [→ 详细报告](engineering/implement/report.md) |
| 9 | **wayfinder** | 规划超出单个 Agent 会话容量的超大块工作，作为 Issue 追踪器上的共享决策工单地图，逐个解决直到路径清晰 | [→ 详细报告](engineering/wayfinder/report.md) |

### 模型调用（Model-invoked）

| # | Skill | 简要描述 | 详细报告 |
|---|-------|---------|---------|
| 13 | **tdd** | 红-绿-重构循环的测试驱动开发，一次一个垂直切片地构建功能或修复 Bug | [→ 详细报告](engineering/tdd/report.md) |
| 16 | **code-review** | 自固定点以来对 diff 的双轴审查：**标准轴**（是否遵循仓库编码标准 + Fowler 坏味道基线？）和 **Spec 轴**（是否忠实实现了原始 Issue/Spec？），作为并行子 Agent 运行 | [→ 详细报告](engineering/code-review/report.md) |

---

## 二、Productivity（生产力类 Skills）

通用工作流工具，不限于编码场景。

### 用户调用（User-invoked）

| # | Skill | 简要描述 | 详细报告 |
|---|-------|---------|---------|
| 19 | **grill-me** | 对一个计划或设计进行无情的深度访谈，直到设计树的每个分支都被解决 | [→ 详细报告](productivity/grill-me/report.md) |
| 20 | **handoff** | 将当前对话压缩为交接文档，使另一个 Agent 可以继续工作 | [→ 详细报告](productivity/handoff/report.md) |
| 23 | **wait-what** | 当一条消息没理解时立即触发。Agent 使用你缺失的上下文、用通俗英语和你的 CONTEXT.md 词汇重新表述 | [→ 详细报告](productivity/wait-what/report.md) |

---

## 三、Misc（trash）

保留但很少使用，不在插件中推广。

| # | Skill | 简要描述 | 详细报告 |
|---|-------|---------|---------|
| 26 | **git-guardrails-claude-code** | 设置 Claude Code 钩子，在执行前阻止危险的 git 命令（push、reset --hard、clean 等） | [→ 详细报告](misc/git-guardrails-claude-code/report.md) |
| 27 | **migrate-to-shoehorn** | 将测试文件从 `as` 类型断言迁移到 @total-typescript/shoehorn | [→ 详细报告](misc/migrate-to-shoehorn/report.md) |
| 28 | **scaffold-exercises** | 创建包含章节、问题、解决方案和解释的练习目录结构 | [→ 详细报告](misc/scaffold-exercises/report.md) |
| 29 | **setup-pre-commit** | 使用 Husky 设置 pre-commit 钩子，包含 lint-staged、Prettier、类型检查和测试 | [→ 详细报告](misc/setup-pre-commit/report.md) |

---

## 四、In-Progress（trash）

Beta 阶段。这些 Skills 公开征求反馈，可能随时变更或消失，不在插件和主 README 中展示。

| # | Skill | 简要描述 | 详细报告 |
|---|-------|---------|---------|
| 30 | **loop-me** | 通过多个会话对自己进行深度访谈，逐步形成可实施的工作流 Spec，使用当前目录作为有状态工作空间 | [→ 详细报告](in-progress/loop-me/report.md) |
| 31 | **writing-beats** | 以节拍（beat）旅程的形式塑造文章，选择你自己的冒险风格。选择一个起始节拍，只写那个节拍，然后转向下一个，直到文章自然结束 | [→ 详细报告](in-progress/writing-beats/report.md) |
| 32 | **writing-fragments** | 深度访谈挖掘你的写作碎片（异质的写作金块），将碎片追加到单个文档中，作为未来文章的原材料 | [→ 详细报告](in-progress/writing-fragments/report.md) |
| 33 | **writing-shape** | 将 Markdown 原始材料文件逐段塑造成文章，在每一步论证格式选择 | [→ 详细报告](in-progress/writing-shape/report.md) |
| 34 | **claude-handoff** | 将当前对话交接给一个新的后台 Agent，该 Agent 立即接手工作，通过 `claude --bg` 种子交接摘要 | [→ 详细报告](in-progress/claude-handoff/report.md) |
| 35 | **setup-ts-deep-modules** | 将 dependency-cruiser 接入 TypeScript 仓库，使每个包成为深层模块：实现隐藏在子文件夹中，只能通过入口点文件访问，测试通过该入口点进行 | [→ 详细报告](in-progress/setup-ts-deep-modules/report.md) |

---

## Skills 之间的关系图

```
grilling (原语)
  ├── grill-me ────────────── 通用深度访谈
  ├── grill-with-docs ─────── 访谈 + 领域建模 + 文档更新
  ├── triage ──────────────── 访谈 + Issue 状态机
  ├── wayfinder ───────────── 访谈 + 大规模工作规划
  └── improve-codebase-architecture ── 访谈 + 代码架构改善

setup-matt-pocock-skills ──── 仓库初始化（先决条件）
  └── 配置 Issue 追踪器、标签、文档布局

to-spec ───────────────────── 对话 → Spec
  └── to-tickets ──────────── Spec → 阻塞工单
       └── implement ──────── 工单 → 代码（驱动 tdd + code-review）

ask-matt ──────────────────── 路由器，映射所有用户可调用 Skills
```

---

## 核心设计理念

1. **小型可组合**：每个 Skill 做一件事并做好，可以组合使用
2. **真正的工程**：强调反馈循环（TDD）、领域建模、代码设计，而非"氛围编程"
3. **共享语言**：通过 CONTEXT.md 和领域模型减少 Agent 与人类之间的沟通鸿沟
4. **深度访谈（Grilling）**：核心模式——在编码之前通过系统性提问确保双方对齐
5. **适配任何模型**：不绑定特定 AI 模型，通用设计

---

> 本索引文件基于 [Matt Pocock Skills 仓库](https://github.com/mattpocock/skills) 的 `README.md`、`CLAUDE.md` 及各桶的 `README.md` 生成。
> 每个 Skill 的详细分析报告请点击上方表格中的链接查看。