# grill-me -- 无状态深度访谈

## 第一部分：详细总结

### 概述

`grill-me` 是一个**用户调用（user-invoked）**的 Skill，属于 `productivity/` 桶。它将一个**松散的想法**通过多轮深度访谈转化为一个你可以做出承诺的清晰方案。你不需要带着完善的计划开始——产出一个完善的计划正是这个会话存在的原因。

其 SKILL.md 极其精简，仅有一行核心指令：`Call the Skill tool with "grilling"`。这是因为 `grill-me` 本身是一个**薄封装**——它将访谈原语 `grilling` 以用户可直接调用的形式暴露出来，不带任何状态文件。

### 核心定位：无状态版本

`grill-me` 是 grill-with-docs 的**无状态**版本。两者的访谈原语完全相同（都是同一个 `grilling` 引擎），但区别在于：

| 特性 | grill-me | grill-with-docs |
|---|---|---|
| 需要仓库 | 否，任何场景皆可 | 是，需要代码库作为对齐基准 |
| 写入文件 | 否，零写入 | 是，写入 `CONTEXT.md` 和 ADR |
| 主题范围 | 任何事物：代码、商业决策、写作、人生选择 | 代码相关的领域建模 |
| 调用方式 | 用户手动 `/grill-me` | 用户手动 `/grill-with-docs` |
| 状态 | 完全无状态 | 有状态，知识留在工作区中 |

三个 grilling 家族技能的定位：
- **任何场景、任何主题**：`grill-me`——无需仓库，不写文件，主题不必是代码
- **有代码库需要对齐**：`grill-with-docs`——同样的访谈，但是有状态的：读取代码并将学到的东西保存在 `CONTEXT.md` 和 ADR 中
- **工作量超出单次会话容量**：`wayfinder`——将工作量绘制为地图，在其内部运行 grilling 会话

### 访谈机制

#### 轮次（Rounds）与前沿（Frontier）

`grill-me` 以**轮次**为单位提问。每一轮是当前的整个**前沿**——即所有其前置问题已经被解决、独立可答的问题的集合。这意味着：
- 你永远不会被问到依赖尚未回答的问题的内容
- 每一轮都清晰地建立在之前轮次的回答之上
- 轮次的数量（而非问题的总数）是衡量进度的正确指标

#### 会话生命周期

- **开始**：在**新对话**中启动（不是在已由 Agent 写好的计划之上叠加），关闭 Plan Mode——Plan Mode 会促使 Agent 急于产出计划，这与保持在探索状态中的目的相反
- **进行中**：Agent 以 carousel 节奏提问，每个问题探索设计树的一个分支
- **结束条件**：前沿为空——每个分支都被访问过，没有任何假设被默默留存
- **之后**：将同一对话直接交给 `to-spec`（不另开新会话），会话中积累的上下文就是其价值

#### 典型规模

46 个问题分布在 4 轮中是一个**正常**的会话——不要被问题总数吓到。轮次比单题计数更能衡量进展。会话结束的标志不是问题数量达标，而是前沿清空。

### 核心约束：用户的角色

该 Skill 的成败**高度依赖于用户的主动性**。这是一个需要人类参与的 Skill，而非一个可完全委托给 Agent 的自动化流程。

#### 必须主动参与

失败模式是**被动性**：回答四十个问题的 "同意、同意、同意"，最终产出一个 Agent 编写、你只是点头同意的计划。这感觉上很有成效，因为会话很长——但事实上没有任何决定被真正做出，结果带有它不应有的确定性。

正确的参与方式是**驾驭**：
- 当问题的粒度低于你需要的精度时**反驳**它
- 当范围在漂移时**指出**它
- 回答 "我不知道" 时**当真**
- Skill 旨在辅助工程师，而非替代：产出的质量取决于你回答的质量，而非问题的数量

#### 知道何时停止

另一个真实但较少见的错误是：在访谈中停留太久，始终不到达代码阶段。

**不可访谈的（Ungrillable）问题**：有些问题无法通过谈话回答——再多轮次的访谈也无济于事。例如：
- "一个长表单还是三个页面？"——需要看到才能决定
- "这个交互应该怎么感觉？"——需要对物品做出反应

当遇到这类问题时，**停止访谈**。用 `prototype` 构建一次性原型，看看它，然后回来一行话回答。

对着不可访谈的问题空谈是会话膨胀的原因：Agent 不断改述，你不断猜测，范围膨胀到填充不确定性。

#### 模型很重要

访谈的质量对模型比大多数 Skill 更敏感。访谈依赖模型自身对系统如何破裂的认知——给它最好的模型。实现阶段主要跟随上下文，可以容忍较便宜的模型。

### 与 Skills 体系的关系

```text
grilling (原语 -- 核心访谈引擎)
  ├── grill-me ────────────── 无状态入口，任何场景
  ├── grill-with-docs ─────── 访谈 + 领域建模 + 文档更新
  ├── triage ──────────────── 访谈 + Issue 状态机
  ├── wayfinder ───────────── 访谈 + 大规模工作规划
  └── improve-codebase-architecture ── 访谈 + 代码架构改善
```

`grill-me` 是**用户调用的前端入口**，携带零负载。它的可移植性来自其无状态性——不需要仓库、不需要工作区、不需要设置，甚至不假设想法是关于软件的。人们用它来处理商业决策、写作、接下来要做什么——任何在头脑中无法安静下来的事物。

### 触发与使用

- **调用方式**：仅 `/grill-me`（用户调用，Agent 不会自动触达它）
- **先决条件**：无。不需要仓库，不需要设置
- **推荐状态**：新对话 + Plan Mode 关闭
- **主题范围**：不限于代码——任何需要认真对待的想法

### 常见问题与应对

| 问题 | 应对 |
|---|---|
| 该预期多少问题？如何知道结束？ | 数轮次，不数问题。46 题 4 轮是正常会话。当前沿为空时结束 |
| 问了我 200 个问题，出了什么问题？ | 通常是范围太大。先将工作拆分为更小的部分，然后逐个访谈。超长会话也会漂入"愚蠢区间"——上下文窗口填满后问题质量下降 |
| 可以回到每次只问一个问题吗？ | 可以。在全局 `CLAUDE.md` 中加入 `When grilling, ask one question at a time.` |
| 真的不知道答案怎么办？ | 说出来。"我不知道"是一个真实的答案。一个你无法回答的问题通常是需要原型而非猜测的信号 |
| 写 Spec 前要开新会话吗？ | 不要。会话的价值就是刚刚建立的上下文。将同一对话直接交给 `to-spec` |
| 模型重要吗？ | 比大多数 Skill 重要。给访谈最好的模型。实现可以容忍较便宜的模型 |

---

## 第二部分：原文与翻译

<!--原文-->
> Call the Skill tool with "grilling".

<!--翻译-->
> 使用 "grilling" 调用 Skill 工具。

<!--原文-->
> `grill-me` takes a **loose idea** and interviews you until you can commit to it. You do not need a worked-out plan to start: producing one is what the [session](https://www.aihero.dev/ai-coding-dictionary/session) is for. It asks in **rounds**: each round is the whole **frontier** (every question whose prerequisites you have already settled), so you are never asked something that hinges on an answer it hasn't heard yet.

<!--翻译-->
> `grill-me` 接收一个**松散的想法**，然后访谈你，直到你能对它做出承诺。你不需要带着完善的计划开始——产出一个完善的计划正是这个[会话](https://www.aihero.dev/ai-coding-dictionary/session)的目的。它以**轮次**提问：每一轮是整个**前沿**（所有前置问题已被解决的问题集合），所以你永远不会被问到依赖尚未回答的问题的内容。

<!--原文-->
> It is **[stateless](https://www.aihero.dev/ai-coding-dictionary/stateless)**. It writes no files and leaves no workspace behind. The only thing it leaves is a sharper version of the idea, in your own head.

<!--翻译-->
> 它是**[无状态](https://www.aihero.dev/ai-coding-dictionary/stateless)**的。它不写文件，不留工作区痕迹。它唯一留下的是你头脑中想法的一个更清晰的版本。

<!--原文-->
> You invoke this by typing `/grill-me`; the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own. Start it in a **fresh conversation**, not on top of a plan you already had an agent write.

<!--翻译-->
> 你通过输入 `/grill-me` 来调用它；[Agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己触达它。在一个**全新对话**中启动，而非在已让 Agent 写好的计划之上叠加。

<!--原文-->
> Reach for it as soon as you have an idea worth taking seriously (a feature, a product direction, a business call, a piece of writing), and long before you have worked out what it involves. Vagueness is not a reason to wait; it is the thing the session eats. If you can already specify the thing precisely, you don't need to grill it.

<!--翻译-->
> 一旦你有一个值得认真对待的想法（一个功能、一个产品方向、一个商业决策、一篇文字），并且远在你弄清它涉及什么之前就触达它。模糊不是等待的理由；它正是这个会话消化的东西。如果你已经能够精确地指定它，你就不需要访谈它了。

<!--原文-->
> Which of the three grilling skills you want depends on what is in front of you:
> - **Anything, anywhere**: `grill-me`. It needs no repo and writes no files, and the subject doesn't have to be code.
> - **A codebase to align against**: [grill-with-docs](https://aihero.dev/skills-grill-with-docs). The same interview, but [stateful](https://www.aihero.dev/ai-coding-dictionary/stateful): it reads your code and keeps what it learns in `CONTEXT.md` and ADRs.
> - **Too big for one session**: [wayfinder](https://aihero.dev/skills-wayfinder). It charts the effort as a map and runs grilling sessions inside it.

<!--翻译-->
> 三个访谈 Skill 中你想用哪个取决于你面前是什么：
> - **任何场景、任何地方**：`grill-me`。不需要仓库，不写文件，主题不必是代码。
> - **有代码库需要对齐**：[grill-with-docs](https://aihero.dev/skills-grill-with-docs)。同样的访谈，但是[有状态](https://www.aihero.dev/ai-coding-dictionary/stateful)的：它读取你的代码，并将学到的东西保存在 `CONTEXT.md` 和 ADR 中。
> - **工作量超出单次会话**：[wayfinder](https://aihero.dev/skills-wayfinder)。它将工作量绘制为地图，在其内部运行访谈会话。

<!--原文-->
> Leave [plan mode](https://www.aihero.dev/ai-coding-dictionary/agent-mode) off. Plan mode primes the agent to rush toward producing a plan, which is the opposite of staying in inquiry.

<!--翻译-->
> 保持 [Plan Mode](https://www.aihero.dev/ai-coding-dictionary/agent-mode) 关闭。Plan Mode 会促使 Agent 急于产出计划，这与保持在探索状态中的目的相反。

<!--原文-->
> The skill asks the questions, but **you** own the scope. That is the part people miss, and it separates a session that turns an idea into decisions from one that produces confident nonsense.

<!--翻译-->
> 这个 Skill 问问题，但**你**拥有范围的掌控权。这是人们容易忽略的部分，它将一个把想法变成决定的会话与一个产出自信但无意义内容的会话区分开来。

<!--原文-->
> The failure mode is **passivity**: answering "agreed, agreed, agreed" for forty questions and coming out with a plan the agent wrote and you nodded at. It feels productive because it was long. Nothing was actually decided, and the result carries a certainty it hasn't earned.

<!--翻译-->
> 失败模式是**被动性**：对着四十个问题回答"同意、同意、同意"，最终产出一个 Agent 编写、你只是点头同意的计划。它感觉很高效，因为花了很长时间。实际上什么也没被决定，结果带有它不应有的确定性。

<!--原文-->
> Being active means steering. Push back on a question pitched beneath the fidelity you need. Say when the scope is drifting. Answer "I don't know" and mean it. This skill is built to aid an engineer, not to replace one: what comes out tracks the quality of your answers, not the number of questions asked.

<!--翻译-->
> 积极参与意味着驾驭。当问题的粒度低于你需要的精度时反驳它。当范围在漂移时指出它。回答"我不知道"并当真。这个 Skill 旨在辅助工程师，而非替代：产出的质量取决于你回答的质量，而非问题的数量。

<!--原文-->
> The opposite error is real but rarer: staying in the interview so long you never reach code.

<!--翻译-->
> 相反的错误确实存在但更少见：在访谈中停留太久，始终不到达代码阶段。

<!--原文-->
> Some questions can be answered by talking. Others can't, and no amount of grilling will get you there.

<!--翻译-->
> 有些问题可以通过谈话回答。其他问题不能，再多的访谈也无济于事。

<!--原文-->
> "One long form or three pages?" and "how should this interaction feel?" are **ungrillable**: they need something to react to. When you hit one, stop grilling. Build the throwaway version with [prototype](https://aihero.dev/skills-prototype), look at it, then come back and answer in one line.

<!--翻译-->
> "一个长表单还是三个页面？"以及"这个交互应该怎么感觉？"是**不可访谈的**：它们需要对物品做出反应。当你遇到一个，停止访谈。用 [prototype](https://aihero.dev/skills-prototype) 构建一次性版本，看看它，然后回来一句话回答。

<!--原文-->
> Talking your way through an ungrillable question is where sessions balloon. The agent keeps rephrasing, you keep guessing, and the scope grows to fill the uncertainty.

<!--翻译-->
> 对着不可访谈的问题空谈是会话膨胀的原因。Agent 不断改述，你不断猜测，范围膨胀到填充不确定性。

<!--原文-->
> ## It's working if
> - You disagree with something. A session with no pushback from you is a session you didn't need.
> - Questions arrive in a few rounds rather than one long drip, and later rounds clearly build on what you said earlier.
> - You end up somewhere you didn't expect, because a question surfaced a decision you had been making implicitly.
> - At the end you could defend each choice to someone who wasn't there.

<!--翻译-->
> ## 运行正常的标志
> - 你对某件事持不同意见。一个没有你反驳的会话是你不需要的会话。
> - 问题以少数几轮的形式到来，而非漫长的涓涓细流，后续轮次清晰地建立在你之前所说的基础上。
> - 你到达了一个意料之外的地方，因为一个问题浮出了你一直在隐式做出的决定。
> - 在结束时，你能向不在场的人为每个选择进行辩护。

<!--原文-->
> **How many questions should I expect, and how do I know when it ends?**
> Count rounds, not questions. Forty-six questions across four rounds is an ordinary session. It ends when the frontier is empty: every branch visited, nothing left silently assumed.

<!--翻译-->
> **我该预期多少问题？如何知道何时结束？**
> 数轮次，不数问题。四轮中四十六个问题是正常会话。当前沿为空时结束：每个分支都被访问过，没有默默留存的假设。

<!--原文-->
> **It asked me two hundred questions. What went wrong?**
> Usually the scope was too large. Ask the agent to break the work into smaller pieces first, then grill each one. Very long sessions also drift into the **[dumb zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**, where the [context window](https://www.aihero.dev/ai-coding-dictionary/context-window) is full enough that the questions get worse.

<!--翻译-->
> **它问了我两百个问题。出了什么问题？**
> 通常是范围太大。先让 Agent 将工作拆分为更小的部分，然后逐个访谈。超长会话也会漂入**[愚蠢区间](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**，[上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)填满到问题质量下降的程度。

<!--原文-->
> **Can I go back to one question at a time?**
> Yes. Add this to your global `CLAUDE.md`: `When grilling, ask one question at a time.`

<!--翻译-->
> **可以回到每次只问一个问题吗？**
> 可以。将此添加到你的全局 `CLAUDE.md` 中：`When grilling, ask one question at a time.`

<!--原文-->
> **What if I genuinely don't know the answer?**
> Say so. "I don't know" is a real answer, and a question you can't answer is usually a sign to prototype rather than to guess.

<!--翻译-->
> **如果我真的不知道答案怎么办？**
> 说出来。"我不知道"是一个真实的答案，而你无法回答的问题通常是需要原型而非猜测的信号。

<!--原文-->
> **Do I start a fresh session before writing the spec?**
> No. The value of the session is the [context](https://www.aihero.dev/ai-coding-dictionary/context) you just built. Hand the same conversation straight to [to-spec](https://aihero.dev/skills-to-spec).

<!--翻译-->
> **写 Spec 之前要开新会话吗？**
> 不需要。会话的价值正是你刚刚建立的[上下文](https://www.aihero.dev/ai-coding-dictionary/context)。将同一对话直接交给 [to-spec](https://aihero.dev/skills-to-spec)。

<!--原文-->
> **Does the model matter?**
> More than for most skills. Grilling leans on the [model](https://www.aihero.dev/ai-coding-dictionary/model)'s own sense of how systems break, so give it your best one. Implementation mostly follows context and tolerates a cheaper model.

<!--翻译-->
> **模型重要吗？**
> 比大多数 Skill 更重要。访谈依赖[模型](https://www.aihero.dev/ai-coding-dictionary/model)自身对系统如何破裂的认知，所以给它最好的模型。实现阶段主要跟随上下文，可以容忍较便宜的模型。

<!--原文-->
> `grill-me` is a **standalone you can run anywhere, on anything**. Being stateless is what makes it portable: no repo, no workspace, no setup, and no assumption that the idea is even about software. People point it at business decisions, at writing, at what to do next: anything that won't sit still in their head.

<!--翻译-->
> `grill-me` 是一个**你可以在任何地方、对任何事物运行的独立工具**。无状态正是它可移植的原因：不需要仓库、不需要工作区、不需要设置，甚至不假设想法是关于软件的。人们用它处理商业决策、写作、接下来做什么——任何在头脑中无法安静下来的事物。

<!--原文-->
> That portability is the whole difference from [grill-with-docs](https://aihero.dev/skills-grill-with-docs), which runs the same interview but reads a codebase to align against and records what it learns as `CONTEXT.md` and ADRs. Both sit on the [grilling](https://aihero.dev/skills-grilling) primitive; `grill-me` is the user-invoked front door that carries nothing with it.

<!--翻译-->
> 这种可移植性是与 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 的全部区别——后者运行同样的访谈，但读取代码库作为对齐基准，并将所学记录为 `CONTEXT.md` 和 ADR。两者都建立在 [grilling](https://aihero.dev/skills-grilling) 原语之上；`grill-me` 是用户调用的前端入口，不携带任何负载。

<!--原文-->
> If what you grilled does turn out to be software, you can hand the same conversation to [to-spec](https://aihero.dev/skills-to-spec) and carry on into the build flow (an option, not the point of the skill). When you're unsure which flow fits, [ask-matt](https://aihero.dev/skills-ask-matt) routes you.

<!--翻译-->
> 如果你访谈的对象最终确实是软件，你可以将同一对话交给 [to-spec](https://aihero.dev/skills-to-spec) 并继续进入构建流程（这是一个选项，而非这个 Skill 的核心目的）。当你不确定哪个流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 为你路由。
