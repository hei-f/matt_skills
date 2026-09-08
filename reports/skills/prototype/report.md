# prototype 技能报告

## 第一部分：技能内容详细总结

### 一、技能概述

`prototype` 构建**用于回答一个设计问题的一次性代码**。原型是"从第一天起就注定要被丢弃的代码"，其目的不是生产功能，而是通过快速构建来验证一个假设或澄清一个设计方向。

该技能的核心哲学是：**问题决定形态**。先确定要回答什么问题，再决定构建什么样的原型。构建错误问题的原型是对时间和 token 的纯粹浪费。

### 二、两个分支

技能根据问题类型分为两个截然不同的分支，产生完全不同的产出物：

#### 分支一：逻辑/状态模型

- **适用问题**："这个逻辑 / 状态模型感觉对吗？"
- **产出物**：一个**单个可共享的 HTML 文件**。无需构建、无需服务器，双击即可打开。
- **核心组件**：带标签的状态面板（每次点击后重新渲染）、自由操作按钮（按任意顺序试探模型）、带标签的引导式演练（每个标签一个场景，下方附有操作按钮）。所有内容使用领域语言标注，可以交给设计师、产品经理或领域专家让他们亲自感受模型。
- **底层逻辑**：是一个小型纯模块（reducer、状态机或一组函数），与 DOM 保持清洁分离，验证通过后可直接提取到真实代码中。

#### 分支二：UI 外观

- **适用问题**："这应该看起来是什么样？"
- **产出物**：在单个路由上生成**几个截然不同的 UI 变体**，通过浮动底部栏和 `?variant=` URL 参数切换。
- **核心要求**：变体必须在**结构**上不同，而非仅仅是颜色——三个仅微调过卡片网格的版本是壁纸，不是原型。尽可能在真实页面中渲染，使用真实数据、真实密度。

#### 问题不明确时的默认策略

如果问题确实模糊且用户不可达，默认选择与周围代码更匹配的分支（后端模块 → 逻辑分支；页面或组件 → UI 分支），并在原型顶部声明所做的假设。

### 三、两个分支通用的六条规则

1. **从第一天起就是一次性的，并明确标记**：将原型代码定位在实际使用位置附近，命名使读者能看出它是原型而非生产代码。
2. **简单到无需思考即可运行**：UI 原型从任务运行器中的一条命令启动，逻辑演示是双击即可打开的 HTML 文件。
3. **默认不持久化**：状态存在于内存中。持久化是原型正在检验的东西，而非依赖的东西。
4. **跳过修饰**：不写测试、不处理超出可运行范围的错误、不做抽象。目的是快速学到东西。
5. **暴露状态**：每次操作后（逻辑）或每次变体切换后（UI），渲染完整的相关状态。
6. **完成后捕获**：将已验证的决策合并到真实代码中，然后将原型本身捕获为主要来源：提交到一个不在 main 上的临时分支，并在实现工单上留下指向该分支的上下文指针。

### 四、原型作为主要来源

完成后的原型留下两样东西：

| 产出 | 去向 |
|------|------|
| **答案**（裁决 + 解决的问题） | 持久捕获：提交消息、ADR、实现工单。main 分支保留的内容。 |
| **原型**（可运行的证据） | 提交到 `prototype/<name>` 分支，永不合并。实现工单上留下上下文指针。 |

### 五、使用场景

**应该使用的时机**：当你遇到一个无法通过讨论解决的设计问题时——状态机的边缘情况无法在脑中完整推理、UI 布局无法在脑中可视化。`grilling` 会话在遇到这种问题时往往不断扩展范围——停止访谈，构建原型，看一眼，然后一句话回答。

**不应该使用的时机**：如果设计已经完全确定，下一步是 `/implement`。Agent 在工单存在时可能会按名称推荐 `prototype`，因为"原型"是一个通用且有吸引力的词——此时应忽略。

**与 `wayfinder` 的关系**：`wayfinder` 将其地图上的原型决策工单委托给此技能。`prototype` 是 wayfinder 工单的四种类型之一。

### 六、已知问题与反模式

1. **将全量应用原型化**：`prototype` 只为一个问题而存在。"整个应用"不是一个问题。全量原型没有自然的停止点，会因惯性变成生产应用。
2. **Token 消耗**：如果原型化可以通过讨论解决的问题，或让一个原型跨越整个功能，可能成为最快的 token 消耗方式。
3. **命名问题**：Agent 有时在设计已完全确定时会推荐 `/prototype`，应该用 `/implement`。
4. **终端应用的弃用**：逻辑分支曾构建终端应用，现已改为 HTML 文件，以便非开发者也能驱动。

### 七、工作判定标准

- 能用一句话说清楚原型用来回答什么问题，并且写在演示的顶部。
- 不读代码的人也能驱动逻辑演示：打开文件，按演练标签中的按钮，用自己的话描述所看到的。
- 有人说"等等，那不应该是可能发生的"——这是想法中的 bug，正是原型化的全部意义。
- UI 变体在布局和信息层级上存在分歧，而不仅仅是颜色和文案。
- 一次坐下来就能完成。如果一天后仍在构建，问题太大了，需要拆分。
- 结束后，main 包含决策而没有任何原型代码，实现工单指向仍持有原型的临时分支。

---

## 第二部分：SKILL.md 原文及翻译

```
<!--原文-->
> ---
> name: prototype
> description: Build a throwaway prototype to answer a design question. Use when the user wants to sanity-check whether a state model or logic feels right, or explore what a UI should look like.
> ---
```
```
<!--翻译-->
> ---
> name: prototype
> description: 构建一次性原型来回答设计问题。当用户想要快速验证状态模型或逻辑是否正确，或者探索 UI 应该是什么样子时使用。
> ---
```

```
<!--原文-->
> # Prototype
>
> A prototype is **throwaway code that answers a question**. The question decides the shape.
```
```
<!--翻译-->
> # 原型
>
> 原型是**用于回答一个问题的可丢弃代码**。问题决定形态。
```

```
<!--原文-->
> ## Pick a branch
>
> Identify which question is being answered, using the user's prompt, the surrounding code, or by asking if the user is around:
>
> - **"Does this logic / state model feel right?"** → [LOGIC.md](LOGIC.md). Build a single shareable HTML file (free-play buttons plus tabbed guided walkthroughs) that pushes the state machine through cases that are hard to reason about on paper, and that a non-developer can drive.
> - **"What should this look like?"** → [UI.md](UI.md). Generate several radically different UI variations on a single route, switchable via a URL search param and a floating bottom bar.
```
```
<!--翻译-->
> ## 选择分支
>
> 使用用户的提示、周围的代码，或者在用户在线时询问，来确定正在回答哪个问题：
>
> - **"这个逻辑 / 状态模型感觉对吗？"** → [LOGIC.md](LOGIC.md)。构建一个单个可共享的 HTML 文件（自由操作按钮加上带标签的引导式演练），将状态机推过在纸面上难以推理的情况，且非开发人员也能驱动。
> - **"这应该看起来是什么样？"** → [UI.md](UI.md)。在单个路由上生成几个截然不同的 UI 变体，可通过 URL 查询参数和浮动底部栏切换。
```

```
<!--原文-->
> The two branches produce very different artifacts, so getting this wrong wastes the whole prototype. If the question is genuinely ambiguous and the user isn't reachable, default to whichever branch better matches the surrounding code (a backend module → logic; a page or component → UI) and state the assumption at the top of the prototype.
```
```
<!--翻译-->
> 两个分支产生截然不同的产出物，因此选错会浪费整个原型。如果问题确实模糊且用户不可达，默认选择与周围代码更匹配的分支（后端模块 → 逻辑；页面或组件 → UI），并在原型顶部声明所做的假设。
```

```
<!--原文-->
> ## Rules that apply to both
>
> 1. **Throwaway from day one, and clearly marked as such.** Locate the prototype code close to where it will actually be used (next to the module or page it's prototyping for) so context is obvious, but name it so a casual reader can see it's a prototype, not production. For throwaway UI routes, obey whatever routing convention the project already uses; don't invent a new top-level structure.
> 2. **Trivial to run.** A UI prototype starts from one command in the project's task runner: `pnpm <name>`, `python <path>`, `bun <path>`, etc. A logic demo is a single HTML file the user double-clicks. Either way, no thinking required to start it.
> 3. **No persistence by default.** State lives in memory. Persistence is the thing the prototype is _checking_, not something it should depend on. If the question explicitly involves a database, hit a scratch DB or a local file with a clear "PROTOTYPE, wipe me" name.
> 4. **Skip the polish.** No tests, no error handling beyond what makes the prototype _runnable_, no abstractions. The point is to learn something fast.
> 5. **Surface the state.** After every action (logic) or on every variant switch (UI), print or render the full relevant state so the user can see what changed.
> 6. **Capture it when done.** Fold any validated decision into the real code, then capture the prototype itself as a **primary source**: commit it to a throwaway branch, out of main, and leave a context pointer to that branch on the implementation issue. Capture the answer too (the verdict and the question it settled) in the issue or a commit. The main branch keeps only the validated decision.
```
```
<!--翻译-->
> ## 两个分支通用的规则
>
> 1. **从第一天起就是一次性的，并明确标记。** 将原型代码定位在实际使用位置附近（与它所原型化的模块或页面相邻）使上下文显而易见，但命名要使偶然看到的读者能看出它是原型而非生产代码。对于一次性 UI 路由，遵循项目已有的路由约定；不要发明新的顶层结构。
> 2. **简单到无需思考即可运行。** UI 原型从项目任务运行器中的一条命令启动：`pnpm <name>`、`python <path>`、`bun <path>` 等。逻辑演示是用户双击即可打开的单个 HTML 文件。无论哪种方式，启动都不需要思考。
> 3. **默认不持久化。** 状态存在于内存中。持久化是原型正在*检验*的东西，而不是它应该依赖的东西。如果问题明确涉及数据库，使用带有清晰"PROTOTYPE，请删除我"名称的临时数据库或本地文件。
> 4. **跳过修饰。** 不写测试、不处理超出让原型*可运行*范围的错误、不做抽象。目的是快速学到东西。
> 5. **暴露状态。** 每次操作后（逻辑）或每次变体切换后（UI），打印或渲染完整的相关状态，以便用户看到发生了什么变化。
> 6. **完成后捕获。** 将任何已验证的决策合并到真实代码中，然后将原型本身捕获为**主要来源**：提交到一个不在 main 分支上的临时分支，并在实现工单上留下指向该分支的上下文指针。同时在工单或提交中捕获答案（裁决以及它所解决的问题）。main 分支只保留已验证的决策。
```
