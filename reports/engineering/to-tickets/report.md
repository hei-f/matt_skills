# to-tickets Skill 分析报告

---

## 第一部分：中文详细总结

### 一、Skill 目的

`to-tickets` 的核心功能是将一个计划（plan）、规格说明（spec）或当前对话内容，拆解为一组 **tracer-bullet（曳光弹）工单**。每个工单明确声明其 **阻塞边（blocking edges）**——即在该工单可以开始之前，必须先完成的其他工单。

与传统的"按层拆分"（水平切片）不同，每一个 tracer-bullet 工单都是一条**垂直切片**：它贯穿所有层次（schema、API、UI、测试），形成一个完整的功能通路。这样做的结果是，每个工单完成后都可以独立演示和验证，不依赖其他工单中尚未完成的部分。此外，每个工单的大小被控制在**单个上下文窗口（context window）**内，使得一个全新的 AI 会话可以直接接手并完成它。

该 skill 是用户手动调用的（`disable-model-invocation: true`），通过输入 `/to-tickets` 来触发。模型不会自行调用它。

---

### 二、处理流程

#### 步骤 1：收集上下文（Gather context）

从当前对话上下文中获取已有的信息。如果用户通过参数传递了引用（例如 spec 文件路径、issue 编号或 URL），则需要获取并阅读其完整正文和评论内容。

#### 步骤 2：探索代码库（可选，Explore the codebase）

如果尚未探索过代码库，应了解代码的当前状态。工单的标题和描述应使用项目领域词汇表中的术语，并尊重相关区域中已存在的 ADR（架构决策记录）。

在此过程中，寻找**预重构（prefactoring）**的机会。遵循原则："Make the change easy, then make the easy change."（先让变更变得容易，再做容易的变更。）预重构的工单应排在整个工单序列的最前面。

#### 步骤 3：起草垂直切片（Draft vertical slices）

将工作拆分为 **tracer bullet** 工单，遵循以下规则：

- 每个切片是一条窄但**完整**的路径，贯穿所有层次（schema、API、UI、测试）：是**垂直**切片，而非某一层的水平切片。
- 完成的切片可以**独立演示或验证**。
- 每个切片的大小适合**单个全新上下文窗口**。
- 任何预重构工作应**优先**排在前面。

为每个工单标注其**阻塞边**：在该工单开始之前必须完成的其他工单。没有阻塞边的工单可以立即开始。

**宽范围重构（Wide Refactors）是对垂直切片原则的例外。** 宽范围重构是指一个机械性的变更（如重命名列、修改共享符号类型），其**影响范围（blast radius）**遍布整个代码库，单次编辑会同时破坏成千上万个调用点，导致没有任何垂直切片能在保持构建绿色的情况下单独落地。对于这种情况，不要强制将其塞入 tracer bullet 模式，而是采用 **expand-contract（扩展-收缩）**模式：

1. **Expand（扩展）**：在旧形式旁边添加新形式，确保不破坏任何现有代码。
2. **Migrate（迁移）**：按影响范围（每个包、每个目录）分批迁移调用点，每批一个工单，均以 Expand 工单为前置依赖。因为旧形式仍然存在，每批迁移之间 CI 保持绿色。
3. **Contract（收缩）**：当所有调用者迁移完毕后，删除旧形式。该工单以所有迁移批次的工单为前置依赖。

如果各个批次单独无法保持绿色，可以让它们共享一个集成分支，所有批次共同阻塞一个最终的"集成并验证"工单，绿色只在该工单处得到保证。

#### 步骤 4：向用户提问确认（Quiz the user）

将拟定的工单拆解以编号列表的形式呈现出来。每个工单展示：

- **标题（Title）**：简短的描述性名称
- **被哪些工单阻塞（Blocked by）**：哪些工单必须先完成
- **交付什么（What it delivers）**：该工单实现的端到端行为

向用户提问：

- 粒度是否合适？（太粗/太细）
- 阻塞边是否正确：每个工单是否只依赖真正构成门槛的工单？
- 是否有工单应该合并或进一步拆分？

反复迭代直到用户批准拆解方案。

#### 步骤 5：将工单发布到配置的追踪器（Publish the tickets to the configured tracker）

发布方式取决于 `/setup-matt-pocock-skills` 配置的追踪器类型。工单本身是相同的，只有阻塞边的表示形式不同：

- **本地文件模式**：在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下每个工单写一个文件，按依赖顺序从 `01` 开始编号（阻塞者在前）。每个文件的"Blocked by"列出其依赖的编号/标题。
- **真实追踪器（GitHub、Linear 等）**：按依赖顺序发布工单（阻塞者在前），利用平台原生的阻塞/子工单关系；如果没有，则在每个工单的"Blocked by"中写明阻塞工单。应用 `ready-for-agent` 分类标签（除非另有指示）。

工作方式是按照 **frontier（前沿）**原则：前沿是指所有阻塞工单均已完成、可以立即开始的工单集合。对于纯线性链，这意味着从上到下依次执行。

不要关闭或修改任何父级 issue。

---

### 三、工单结构

#### 本地文件模板

```
# <NN>: <工单标题>

**What to build:** 从用户视角描述该工单实现的端到端行为，而非逐层实现列表。

**Blocked by:** 阻塞该工单的工单编号/标题，或 "None (can start immediately)"。

**Status:** ready-for-agent

- [ ] 验收标准 1
- [ ] 验收标准 2
```

#### Issue 模板（用于真实追踪器）

```
## Parent
父级 issue 的引用（如果来源是已有 issue；否则省略此节）。

## What to build
从用户视角描述端到端行为，非逐层实现。

## Acceptance criteria
- [ ] 标准 1
- [ ] 标准 2

## Blocked by
- 每个阻塞工单的引用，或 "None (can start immediately)"
```

无论采用哪种形式，都应避免在工单中使用具体的文件路径或代码片段，因为它们很快就会过时。例外情况：如果原型阶段产出的代码片段比文字描述更精确地编码了某个决策（如状态机、reducer、schema、类型结构），则可以内联该片段，并简要注明其来自原型。只保留决策关键的部分，而非一个完整可运行的原型。

---

### 四、阻塞边（Blocking Edges）

阻塞边是该 skill 产出的核心价值。根据追踪器类型的不同，阻塞边有两种表现形式：

| 追踪器 | 边的存储位置 | 工作方式 |
|--------|-------------|----------|
| 本地 Markdown | 每个工单文件内的文本，位于 `.scratch/<feature>/issues/<NN>-<slug>.md`，按阻塞者优先编号 | 从上到下，手动推进 |
| 真实追踪器（GitHub、Linear） | 原生阻塞链接，或子工单关系（如果追踪器支持） | 任何阻塞已清除的工单处于**前沿**，可以被领取 |

无论哪种方式，阻塞边都存在于工单本身之中。媒介只是决定了是否可以并行处理。`to-tickets` 只产出工单，运行工单（单会话或并行团队）是用户的工作，而非该 skill 的职责。

---

### 五、前置条件

`to-tickets` 将工单发布到追踪器中，因此必须通过 `/setup-matt-pocock-skills` 为当前仓库配置好追踪器，以及分类标签词汇表（triage label vocabulary）。两种方式都支持：真实追踪器（如 GitHub 或 Linear），或本地 markdown 文件（`.scratch/` 目录下），后者开箱即用。

---

### 六、在整体流程中的位置

`to-tickets` 是主构建链中的一个步骤：

```
grill-with-docs → to-spec → to-tickets → implement → code-review
```

- 上游是 `to-spec`，它提供一份已确定的规格说明供切片使用；两者应在同一个未中断的上下文窗口中运行。
- 下游是 `implement`，它在每个新会话中构建一个工单，使用 TDD 驱动测试，并以 `code-review` 收尾。

---

## 第二部分：SKILL.md 原文与翻译

---

<!--原文-->
> ---
> name: to-tickets
> description: Break a plan, spec, or the current conversation into a set of tracer-bullet tickets, each declaring its blocking edges, published to the configured tracker (edges as text in one file per ticket locally, or native blocking links on a real tracker).
> disable-model-invocation: true
> ---
<!--翻译-->
> ---
> name: to-tickets
> description: 将计划、规格说明或当前对话拆解为一组 tracer-bullet 工单，每个工单声明其阻塞边，发布到已配置的追踪器（本地模式下每个工单一个文件，阻塞边以文本形式记录；真实追踪器上则使用原生阻塞链接）。
> disable-model-invocation: true
> ---

---

<!--原文-->
> # To Tickets
<!--翻译-->
> # 转为工单（To Tickets）

---

<!--原文-->
> Break a plan, spec, or conversation into a set of **tickets**: tracer-bullet vertical slices, each declaring the tickets that **block** it.
<!--翻译-->
> 将一个计划、规格说明或对话内容拆解为一组**工单**：tracer-bullet 垂直切片，每个工单声明**阻塞**它的那些工单。

---

<!--原文-->
> The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.
<!--翻译-->
> Issue 追踪器和分类标签词汇表应该已经提供给你。如果没有，告知用户运行 `/setup-matt-pocock-skills`。

---

<!--原文-->
> ## Process
<!--翻译-->
> ## 流程

---

<!--原文-->
> ### 1. Gather context
<!--翻译-->
> ### 1. 收集上下文

---

<!--原文-->
> Work from whatever is already in the conversation context. If the user passes a reference (a spec path, an issue number or URL) as an argument, fetch it and read its full body and comments.
<!--翻译-->
> 从当前对话上下文中已有的信息入手。如果用户通过参数传递了引用（如 spec 文件路径、issue 编号或 URL），获取该引用并阅读其完整正文和评论。

---

<!--原文-->
> ### 2. Explore the codebase (optional)
<!--翻译-->
> ### 2. 探索代码库（可选）

---

<!--原文-->
> If you have not already explored the codebase, do so to understand the current state of the code. Ticket titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.
<!--翻译-->
> 如果你尚未探索过代码库，请进行探索以了解代码的当前状态。工单的标题和描述应使用项目的领域词汇表术语，并尊重你所触及区域中的 ADR（架构决策记录）。

---

<!--原文-->
> Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."
<!--翻译-->
> 寻找预重构代码的机会，使后续的实现更加容易。原则是："先让变更变得容易，再做容易的变更"（Make the change easy, then make the easy change）。

---

<!--原文-->
> ### 3. Draft vertical slices
<!--翻译-->
> ### 3. 起草垂直切片

---

<!--原文-->
> Break the work into **tracer bullet** tickets.
<!--翻译-->
> 将工作拆分为 **tracer bullet** 工单。

---

<!--原文-->
> - Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests): vertical, NOT a horizontal slice of one layer
> - A completed slice is demoable or verifiable on its own
> - Each slice is sized to fit in a single fresh context window
> - Any prefactoring should be done first
<!--翻译-->
> - 每个切片是一条窄但**完整**的路径，贯穿所有层次（schema、API、UI、测试）：是**垂直**切片，而非某一层的水平切片。
> - 完成的切片可以**独立演示或验证**。
> - 每个切片的大小适合**单个全新上下文窗口**。
> - 任何预重构工作应**优先**排在前面。

---

<!--原文-->
> Give each ticket its **blocking edges**: the other tickets that must complete before it can start. A ticket with no blockers can start immediately.
<!--翻译-->
> 为每个工单赋予其**阻塞边**：在该工单开始之前必须完成的其他工单。没有阻塞边的工单可以立即开始。

---

<!--原文-->
> **Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Don't force it into a tracer bullet; sequence it as **expand–contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket; green is promised only there.
<!--翻译-->
> **宽范围重构是对垂直切片原则的例外。** 宽范围重构是指一个机械性的变更（如重命名列、修改共享符号类型），其**影响范围（blast radius）**遍布整个代码库，单次编辑会同时破坏成千上万个调用点，导致没有任何垂直切片能保持绿色地单独落地。不要强制将其塞入 tracer bullet 模式，而是采用**扩展-收缩（expand-contract）**顺序。首先扩展：在旧形式旁边添加新形式，确保不破坏任何现有代码。然后按影响范围（每个包、每个目录）分批迁移调用点，每批为一个工单，均以扩展工单为阻塞依赖。因为旧形式仍然存在，各批次之间 CI 保持绿色。最后收缩：当所有调用者迁移完毕后，删除旧形式，该工单以所有迁移批次的工单为阻塞依赖。如果各个批次单独无法保持绿色，保持该顺序但让它们共享一个集成分支，所有批次共同阻塞一个最终的集成并验证工单；绿色只在该处得到保证。

---

<!--原文-->
> ### 4. Quiz the user
<!--翻译-->
> ### 4. 向用户确认

---

<!--原文-->
> Present the proposed breakdown as a numbered list. For each ticket, show:
>
> - **Title**: short descriptive name
> - **Blocked by**: which other tickets (if any) must complete first
> - **What it delivers**: the end-to-end behaviour this ticket makes work
<!--翻译-->
> 将拟定的工单拆解以编号列表的形式呈现。每个工单展示：
>
> - **标题（Title）**：简短的描述性名称
> - **被阻塞于（Blocked by）**：哪些工单（如有）必须先完成
> - **交付什么（What it delivers）**：该工单实现的端到端行为

---

<!--原文-->
> Ask the user:
>
> - Does the granularity feel right? (too coarse / too fine)
> - Are the blocking edges correct: does each ticket only depend on tickets that genuinely gate it?
> - Should any tickets be merged or split further?
<!--翻译-->
> 向用户提问：
>
> - 粒度是否合适？（太粗/太细）
> - 阻塞边是否正确：每个工单是否只依赖真正构成门槛的工单？
> - 是否有工单应该合并或进一步拆分？

---

<!--原文-->
> Iterate until the user approves the breakdown.
<!--翻译-->
> 反复迭代直到用户批准拆解方案。

---

<!--原文-->
> ### 5. Publish the tickets to the configured tracker
<!--翻译-->
> ### 5. 将工单发布到已配置的追踪器

---

<!--原文-->
> Publish the approved tickets. **How** depends on the tracker `/setup-matt-pocock-skills` configured; the tickets are the same either way, only the shape of the blocking edges changes:
<!--翻译-->
> 发布已批准的工单。**如何发布**取决于 `/setup-matt-pocock-skills` 配置的追踪器类型；工单本身是相同的，只有阻塞边的表现形式不同：

---

<!--原文-->
> - **Local files** → write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first). Each file's "Blocked by" lists the numbers/titles it depends on. Use the per-ticket file template below: one ticket per file, never a single combined file.
> - **A real issue tracker (GitHub, Linear, ...)** → publish one issue per ticket in dependency order (blockers first) so each ticket's blocking edges can reference real identifiers. Use the platform's native blocking / sub-issue relationship where it has one; otherwise set each ticket's "Blocked by" to the blocking issues. Apply the `ready-for-agent` triage label unless instructed otherwise; the tickets are agent-grabbable by construction.
<!--翻译-->
> - **本地文件** → 在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下每个工单写一个文件，按依赖顺序从 `01` 开始编号（阻塞者在前）。每个文件中的"Blocked by"列出其依赖的编号/标题。使用下面的单工单文件模板：每个文件一个工单，永远不要使用单个合并文件。
> - **真实 issue 追踪器（GitHub、Linear 等）** → 按依赖顺序发布工单（阻塞者在前），使得每个工单的阻塞边可以引用真实的标识符。如果平台有原生阻塞/子工单关系则使用；否则在每个工单的"Blocked by"中写明阻塞的 issue。除非另有指示，应用 `ready-for-agent` 分类标签；这些工单从结构上就适合由 agent 领取。

---

<!--原文-->
> Work the **frontier**: any ticket whose blockers are all done. For a purely linear chain that means top to bottom.
<!--翻译-->
> 按照**前沿（frontier）**方式工作：前沿是指所有阻塞工单均已完成、可以立即开始的工单集合。对于纯线性链，这意味着从上到下依次执行。

---

<!--原文-->
> Do NOT close or modify any parent issue.
<!--翻译-->
> 不要关闭或修改任何父级 issue。

---

<!--原文-->
> ```
> # <NN>: <Ticket title>
>
> **What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective, not a layer-by-layer implementation list.
>
> **Blocked by:** the numbers/titles of the tickets that gate this one, or "None (can start immediately)".
>
> **Status:** ready-for-agent
>
> - [ ] Acceptance criterion 1
> - [ ] Acceptance criterion 2
> ```
<!--翻译-->
> ```
> # <NN>: <工单标题>
>
> **What to build:** 从用户视角描述该工单实现的端到端行为，而非逐层实现列表。
>
> **Blocked by:** 阻塞该工单的工单编号/标题，或 "None (can start immediately)"。
>
> **Status:** ready-for-agent
>
> - [ ] 验收标准 1
> - [ ] 验收标准 2
> ```

---

<!--原文-->
> ```
> ## Parent
>
> A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).
>
> ## What to build
>
> The end-to-end behaviour this ticket makes work, from the user's perspective, not layer-by-layer implementation.
>
> ## Acceptance criteria
>
> - [ ] Criterion 1
> - [ ] Criterion 2
>
> ## Blocked by
>
> - A reference to each blocking ticket, or "None (can start immediately)".
> ```
<!--翻译-->
> ```
> ## Parent
>
> 父级 issue 在追踪器上的引用（如果来源是已有 issue；否则省略此节）。
>
> ## What to build
>
> 从用户视角描述端到端行为，非逐层实现。
>
> ## Acceptance criteria
>
> - [ ] 标准 1
> - [ ] 标准 2
>
> ## Blocked by
>
> - 每个阻塞工单的引用，或 "None (can start immediately)"。
> ```

---

<!--原文-->
> In either form, avoid specific file paths or code snippets: they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.
<!--翻译-->
> 无论采用哪种形式，都应避免使用具体的文件路径或代码片段：它们很快就会过时。例外情况：如果原型阶段产出的代码片段比文字描述更精确地编码了某个决策（如状态机、reducer、schema、类型结构），则可以内联该片段，并简要注明其来自原型。只保留决策关键的部分，而非一个完整可运行的原型，仅保留重要的部分。

---

## （完）
