# to-spec -- 对话转 Spec 并发布

## 第一部分：详细总结

### 概述

`to-spec` 是一个由用户手动调用的 Skill（禁止模型隐式调用），其核心目的是将**当前对话中已经达成的共识**合成为一份正式的 Spec（规格说明文档），并直接发布到项目的 Issue 追踪器中。

该 Skill 的定位是"决策记录者"而非"决策者"。它不对用户进行任何形式的访谈或追问，只负责将对话线程、代码库理解、`CONTEXT.md` 术语表以及相关 ADR（架构决策记录）中已经确定的内容，用项目自身的术语体系整理成一份结构化的规格说明。

### 适用场景

`to-spec` 服务于"多会话构建链"中的一个关键节点。当功能规模超过单个 Agent 会话的处理能力、需要被拆解为多个会话执行时，Spec 作为跨会话传递决策的载体，确保后续会话无需用户重复解释即可继续工作。

决策路径：

| 场景 | 应使用的 Skill/流程 |
|---|---|
| 尚未做出任何决策 | 先执行 `grill-with-docs` 进行需求研磨 |
| 已决策，且工作量可在一个上下文窗口内完成 | 使用 `implement`，跳过 Spec |
| 已决策，且工作量跨越多个会话 | `/to-spec`，然后 `/to-tickets` |
| Wayfinder 探索地图已完成 | `/to-spec #<map_issue>` |

完整构建链：`grill-with-docs -> to-spec -> to-tickets -> implement -> code-review`

### 核心原则：禁止访谈

这是 `to-spec` 最重要的行为准则。Skill 明确要求："Do NOT interview the user; just synthesize what you already know."（不要访谈用户；只需综合你已经知道的内容。）

其背后的设计哲学是：当用户调用此 Skill 时，所有决策都已经在 `grill-with-docs` 或之前的对话中完成了。Spec 的作用是**保存**这些决策，而非开启新一轮讨论。如果 Spec 中断言了用户从未说过的事情，那是一个缺陷。

### 前置条件

Issue 追踪器和 Triage 标签词汇表必须已经通过 `/setup-matt-pocock-skills` 配置完成。支持两种追踪器类型：
- 真实的追踪器（如 GitHub Issues）
- 本地 Markdown 文件（位于 `.scratch/` 目录下，开箱即用）

如果这些尚未配置，Skill 会提示用户先运行 `/setup-matt-pocock-skills`。

### 执行流程

#### 第一步：探索代码库

如果尚未进行，先深入探索仓库以理解代码库的当前状态。在撰写 Spec 的整个过程中，必须：

- **使用项目的领域词汇表**：以 `CONTEXT.md` 中定义的术语为准，确保 Spec 中的语言与项目术语一致，而非使用通用的产品管理套话。
- **尊重相关 ADR**：读取并遵守与当期功能相关的架构决策记录，避免做出违反既定架构方向的决策。

#### 第二步：确定测试接缝（Seams）

在动笔写 Spec 之前，必须先绘制出功能将要被测试的**接缝（Seams）**。这是 `to-spec` 最独特的设计特性之一。

接缝选择的优先级规则：

1. **优先使用现有接缝**，而非创建新的
2. **使用尽可能高层的接缝**
3. **接缝越少越好**——理想情况下整个变更是**一个接缝**

选定接缝后，**必须与用户确认**这些接缝是否符合预期。

这些被确认的接缝将在整个构建链中传导：
- `tdd` 技能**仅在预先确认的接缝处**进行测试
- `code-review` 技能**对照 Spec 来审查 diff**，任何未经确认的接缝会作为 review 发现项呈现出来

这意味着接缝的约束是间接但严格的：它通过 Spec 文档贯穿整个流程。这正是为什么在 `to-spec` 阶段认真对待接缝对话非常重要，而不是推迟到实现阶段再讨论。

#### 第三步：撰写并发布 Spec

使用固定的模板撰写 Spec，然后直接发布到项目 Issue 追踪器。为 Issue 打上 `ready-for-agent` 标签，无需额外分流。

### Spec 模板各章节说明

#### Problem Statement（问题陈述）

从用户视角描述用户正在面临的问题。这是 Spec 的起点——说明为什么要做这个功能。

#### Solution（解决方案）

从用户视角描述问题的解决方案。聚焦于用户体验层面的变化，而非技术实现细节。

#### User Stories（用户故事）

一个**详尽**的编号列表，每个故事遵循固定格式：

```
1. As an <actor>, I want a <feature>, so that <benefit>
```

示例：
```
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
```

用户故事列表应**极其详尽**，覆盖功能的所有方面。

> **已知局限**：该模板高度依赖用户故事格式，对于重构或模块边界调整等架构性工作不够贴合。这种情况下，应侧重 Implementation Decisions 和 Testing Decisions 章节，并让持久的架构决策以 ADR 的形式落地（通过 `grill-with-docs`），而非试图让 Spec 承载它们。

#### Implementation Decisions（实现决策）

列出已做出的实现决策，可包括：

- 将被构建/修改的模块
- 这些模块将被修改的接口
- 开发人员的技术澄清
- 架构决策
- Schema 变更
- API 契约
- 具体的交互方式

**关键约束**：不要包含具体的文件路径或代码片段，因为它们很快就会过时。

**例外**：如果某个原型（prototype）产出了一段代码片段，该片段比文字描述更精确地编码了某个决策（如状态机、reducer、Schema、类型形状），可以在相关决策中内联该片段，并简要注明它来自原型。只保留对决策有价值的部分，而非完整的工作演示。

#### Testing Decisions（测试决策）

列出已做出的测试决策，包括：

- 什么样的测试是好的测试（只测试外部行为，不测试实现细节）
- 哪些模块将被测试
- 测试的参考范例（即代码库中类似的测试类型）

#### Out of Scope（范围之外）

明确说明哪些内容**不属于**本 Spec 的范围。通常，用户明确拒绝的内容是最有价值的行——它们划定边界，防止范围蔓延。

#### Further Notes（补充说明）

关于该功能的任何附加说明。

### 发布后的标签策略

Spec 发布时直接打上 `ready-for-agent` 标签。这个标签的含义是"无需进一步分流"：文档足够完整，Agent 可以据此工作。它是一个输入设计标签（input designation），而非工作指令（work order）。

**注意事项**：如果你运行了轮询 `ready-for-agent` 标签的 AFK Agent，它们会看到父 Spec 并尝试一次性构建整个 Spec，而不是逐个取用 `to-tickets` 拆分出的 Ticket。这是该 Skill 最常被反馈的粗糙边缘。解决方案：
- 在 AFK Agent 的 prompt 中显式排除父 Spec Issue
- 或在 `/to-tickets` 执行完成后移除父 Spec 的 `ready-for-agent` 标签

### Spec 的定位与生命周期

- **Spec 是一次性快照**：它在写出时记录了你当时的认知，一旦实现过程中学到了新东西，Spec 就会过时。把它当作工作交付后即可丢弃的文档。
- **真正持久的产物是 `CONTEXT.md` 和 ADR**：如果在实现过程中学到的东西值得保留，应将其写入 `CONTEXT.md` 或 ADR，而非去编辑 Spec。
- **Ticket 不可持久，Spec 不可替代**：每个 Ticket 为一个上下文窗口设计，使用完毕后删除或关闭；而 Spec 作为这些 Ticket 背后的推理记录持续存在。这是 Spec 在多会话工作中的核心价值——即使 Ticket 被清理了，决策依据仍然在。
- 不会自动搜索追踪器中是否有重叠的 Issue，也不会在 Spec 中引用其所遵守的 ADR。如果相关领域活跃，用户应自行搜索以避免重复。

### 成功标准

- Skill 开始**写**而不是提出新一轮问题
- 先向你确认接缝，并且尽可能少地提案接缝
- 用你项目的术语返回 Spec，而非通用的产品管理套话
- 其中的每一项决策都是你能回忆起做出的——没有任何内容是为了填充模板而编造的
- Out of Scope 章节包含真实的内容：你明确拒绝的事情通常是最有价值的

---

## 第二部分：原文与翻译

<!--原文-->
> This skill takes the current conversation context and codebase understanding and produces a spec. Do NOT interview the user; just synthesize what you already know.

<!--翻译-->
> 此 Skill 利用当前对话上下文和代码库理解来生成一份 Spec。不要访谈用户；只需综合你已经知道的内容。

---

<!--原文-->
> The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

<!--翻译-->
> Issue 追踪器和 Triage 标签词汇表应该已提供给你。如果没有，告诉用户运行 `/setup-matt-pocock-skills`。

---

<!--原文-->
> ## Process

<!--翻译-->
> ## 流程

---

<!--原文-->
> 1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.

<!--翻译-->
> 1. 探索仓库以理解代码库的当前状态（如果尚未进行）。在 Spec 中全程使用项目的领域词汇表，并遵守你正在处理的区域中的任何 ADR。

---

<!--原文-->
> 2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

<!--翻译-->
> 2. 绘制出你将要测试功能的接缝（Seams）。应优先使用现有接缝而非新建接缝。使用尽可能高层的接缝。如果需要新接缝，在尽可能高的位置提议它们。整个代码库中的接缝越少越好——理想数量是一个。

---

<!--原文-->
> Check with the user that these seams match their expectations.

<!--翻译-->
> 与用户确认这些接缝是否符合其预期。

---

<!--原文-->
> 3. Write the spec using the template below, then publish it to the project issue tracker. Apply the `ready-for-agent` triage label - no need for additional triage.

<!--翻译-->
> 3. 使用以下模板撰写 Spec，然后将其发布到项目 Issue 追踪器。打上 `ready-for-agent` 分流标签——无需额外分流。

---

<!--原文-->
> ## Problem Statement

<!--翻译-->
> ## 问题陈述

---

<!--原文-->
> The problem that the user is facing, from the user's perspective.

<!--翻译-->
> 用户正在面临的问题，从用户的视角描述。

---

<!--原文-->
> ## Solution

<!--翻译-->
> ## 解决方案

---

<!--原文-->
> The solution to the problem, from the user's perspective.

<!--翻译-->
> 针对问题的解决方案，从用户的视角描述。

---

<!--原文-->
> ## User Stories

<!--翻译-->
> ## 用户故事

---

<!--原文-->
> A LONG, numbered list of user stories. Each user story should be in the format of:

<!--翻译-->
> 一个冗长的、带编号的用户故事列表。每个用户故事应遵循以下格式：

---

<!--原文-->
> 1. As an <actor>, I want a <feature>, so that <benefit>

<!--翻译-->
> 1. 作为<角色>，我想要<功能>，以便<收益>

---

<!--原文-->
> <user-story-example>
> 1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
> </user-story-example>

<!--翻译-->
> <用户故事示例>
> 1. 作为移动银行客户，我想查看账户余额，以便我能对消费做出更明智的决策
> </用户故事示例>

---

<!--原文-->
> This list of user stories should be extremely extensive and cover all aspects of the feature.

<!--翻译-->
> 该用户故事列表应极其详尽，并覆盖功能的所有方面。

---

<!--原文-->
> ## Implementation Decisions

<!--翻译-->
> ## 实现决策

---

<!--原文-->
> A list of implementation decisions that were made. This can include:

<!--翻译-->
> 已做出的实现决策列表。可以包括：

---

<!--原文-->
> - The modules that will be built/modified
> - The interfaces of those modules that will be modified
> - Technical clarifications from the developer
> - Architectural decisions
> - Schema changes
> - API contracts
> - Specific interactions

<!--翻译-->
> - 将被构建/修改的模块
> - 这些模块将被修改的接口
> - 来自开发人员的技术澄清
> - 架构决策
> - Schema 变更
> - API 契约
> - 具体的交互方式

---

<!--原文-->
> Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

<!--翻译-->
> 不要包含具体的文件路径或代码片段。它们可能很快就会过时。

---

<!--原文-->
> Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

<!--翻译-->
> 例外：如果某个原型产出了一段代码片段，该片段比文字描述更精确地编码了某个决策（如状态机、reducer、Schema、类型形状），则在相关决策中内联该片段，并简要注明它来自原型。只保留对决策有价值的部分，而非完整的工作演示，只需重要的部分。

---

<!--原文-->
> ## Testing Decisions

<!--翻译-->
> ## 测试决策

---

<!--原文-->
> A list of testing decisions that were made. Include:

<!--翻译-->
> 已做出的测试决策列表。包括：

---

<!--原文-->
> - A description of what makes a good test (only test external behavior, not implementation details)
> - Which modules will be tested
> - Prior art for the tests (i.e. similar types of tests in the codebase)

<!--翻译-->
> - 什么样的测试是好的测试的描述（只测试外部行为，不测试实现细节）
> - 哪些模块将被测试
> - 测试的参考范例（即代码库中类似的测试类型）

---

<!--原文-->
> ## Out of Scope

<!--翻译-->
> ## 范围之外

---

<!--原文-->
> A description of the things that are out of scope for this spec.

<!--翻译-->
> 对本 Spec 范围之外的内容的描述。

---

<!--原文-->
> ## Further Notes

<!--翻译-->
> ## 补充说明

---

<!--原文-->
> Any further notes about the feature.

<!--翻译-->
> 关于该功能的任何补充说明。
