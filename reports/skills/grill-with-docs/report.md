# grill-with-docs 技能报告

## 第一部分：技能内容详细总结

### 一、技能概述

`grill-with-docs` 是一个**薄包装技能（thin wrapper skill）**，本身不包含逻辑，而是将工作委托给两个底层技能：`grilling`（访谈原始能力）和 `domain-modeling`（领域建模）。它的核心定位是 `grill-me` 的**有状态版本**——在访谈过程中将学到的内容持久化到 `CONTEXT.md` 和 ADR（架构决策记录）中，为项目留下书面痕迹（paper trail）。

该技能位于 Matt Pocock 技能体系的主构建链头部：

```
grill-with-docs → to-spec → to-tickets → implement → code-review
```

### 二、核心机制

#### 2.1 委托模式

`SKILL.md` 内容仅一行指令：

```
Call the Skill tool twice, for "grilling" and "domain-modeling".
```

安装该技能时必须同时安装 `grilling` 和 `domain-modeling` 两个依赖，否则技能会静默失败。实践中常见的失败模式有两种：

- **完全未加载依赖**：Agent 会猜测"grilling"的含义，输出一堆无差别的提问，不会提及 `CONTEXT.md`。
- **部分加载**：`grilling` 成功加载但 `domain-modeling` 未加载，表现为访谈正常但没有任何文件产出。

#### 2.2 有状态访谈

与 `grill-me` 不同，`grill-with-docs` 将访谈结论实时写入磁盘：

- **术语（Term）**：一旦在对话中明确，立即写入 `CONTEXT.md`，而非在会话结束时批量写入。
- **决策（Decision）**：只有同时满足三个条件的决策才会生成 ADR：难以逆转、缺乏上下文时令人惊讶、且是真正的权衡取舍。大多数会话不会产生 ADR，这是正常行为。

### 三、产出物

| 产出类型 | 存放位置 | 写入时机 |
|---------|---------|---------|
| 术语定义 | `CONTEXT.md` | 实时写入，逐条添加 |
| ADR 决策 | `docs/adr/` | 满足三个条件时 |
| 其他所有决策 | 仅存在于对话中 | 不写入文件 |

`CONTEXT.md` 被设计为纯粹的**术语表（glossary）**，不含实现细节、不写规格说明、不记草稿笔记。这是最容易被误解的设计点：大量会话中达成的共识只存在于上下文窗口中，需要手动交给 `to-spec` 技能来固化。

### 四、使用场景与选择

该技能为用户调用型（`disable-model-invocation: true`），Agent 不会自动触发。适用场景对比：

| 当前状态 | 选择的技能 |
|---------|-----------|
| 完全不在任何工作目录中 | `grill-me` |
| 在仓库中，变更可在一个会话内完成 | `grill-with-docs` |
| 工作量太大，无法在一个会话内持有 | `wayfinder`（多会话规划） |
| 仓库没有领域文档，也无特定功能要开发 | `grill-with-docs`，面向整个仓库 |
| 决策依赖于他人头脑中的知识 | `to-questionnaire` |

关键区分点在于**会话数量**：`grill-with-docs` 用于单会话规划，`wayfinder` 用于多会话规划。

### 五、工作判定标准

技能正常工作的标志：

- `CONTEXT.md` 在会话**过程中**逐条更新，而非会话结束时一次性出现。
- 术语表读起来像纯粹的词汇表（项目自有词汇及其精确定义），不包含实现细节。
- 代码库能回答的问题通过阅读代码库来回答，而非向用户提问。
- 产生很少或没有 ADR，若有则是你会恼火不得不重新争论的决策。
- 技能会因为你使用的词汇与已有术语表中定义的冲突而挑战你。

### 六、已知问题

1. **依赖加载失败**：这是报告最多的问题。当技能在其他编排层内运行时，文件写入部分可能静默不发生。
2. **决策丢失**：大量会话中确定的答案（排序保证、负向需求、数值默认值等）仅存在于对话中。
3. **命名争议**：当前名称不尽如人意，存在将其重命名为 `grill-domain-model` 的公开提议。
4. **会话结束后的衔接**：技能的结束消息通常是开放式的，推荐做法是在同一对话中直接调用 `to-spec` 或 `implement`。

---

## 第二部分：SKILL.md 原文及翻译

```
<!--原文-->
> ---
> name: grill-with-docs
> description: A relentless interview to sharpen a plan or design, which also creates docs (ADR's and glossary) as we go.
> disable-model-invocation: true
> ---
```
```
<!--翻译-->
> ---
> name: grill-with-docs
> description: 一场持续深入的访谈，用于打磨计划或设计方案，同时在过程中创建文档（ADR 和术语表）。
> disable-model-invocation: true
> ---
```

```
<!--原文-->
> Call the Skill tool twice, for "grilling" and "domain-modeling".
```
```
<!--翻译-->
> 调用 Skill 工具两次，分别使用 "grilling" 和 "domain-modeling"。
```
