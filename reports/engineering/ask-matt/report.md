# ask-matt — Skill 路由器

## 第一部分：详细总结

### 概述

`ask-matt` 是本仓库所有用户调用 Skills 的**路由器**。当你不记得有哪些 Skills 可用时，通过它来确定哪个 Skill 或流程适合你的当前情况。它不执行具体工作，而是引导你进入正确的流程。

### 核心概念

#### 主流程：idea → ship（想法 → 交付）

大多数工作遵循的主路径：

1. **`/grill-with-docs`** — 通过访谈细化想法。在工作目录中始终从这里开始，因为它是**有状态的**，会在 `CONTEXT.md` 和 ADR 中保留学到的东西。
2. **分支：能否在对话中解决所有问题？** — 如果某个问题需要可运行的答案（状态、业务逻辑、UI），则绕道通过 `/prototype` 原型验证，使用 `/handoff` 在双向桥接。
3. **分支：这是多会话构建吗？**
   - **是** → `/to-spec`（对话→Spec）→ `/to-tickets`（Spec→追踪子弹工单，声明阻塞边）→ `/implement`（逐个工单实现，内部驱动 `/tdd` 和 `/code-review`）
   - **否** → 直接 `/implement`，在同一上下文窗口中完成

#### 上下文卫生

- 步骤 1-3 保持在一个**不间断的上下文窗口**中
- 注意 **Smart Zone** 限制（约 150K tokens），如果会话接近此限制，在阶段边界处 `/compact`
- 每个 `/implement` 在清空上下文后从工单开始

#### 接入匝道（On-ramps）

- **Bug 和请求堆积** → `/triage`：通过状态机流转 Issue，产出 Agent 就绪的工单
- **某物坏了** → `/diagnosing-bugs`：纪律性诊断循环，先构建紧反馈循环，再假设、修复
- **超大模糊工作**（绿地项目或超大功能） → `/wayfinder`：在 Issue 追踪器上绘制决策工单共享地图，逐个解决

#### 代码库健康

- `/improve-codebase-architecture`：扫描代码库寻找深化机会，产出候选列表
- `/codebase-design`：设计深层模块的共享词汇

#### 底层词汇

- `/domain-modeling`：优化项目的领域语言，挑战模糊术语，记录 ADR
- `/codebase-design`：深层模块词汇（模块、接口、深度、接缝、适配器、杠杆、局部性）

#### 阶段边界

在阶段边界有五个选择：Continue、`/clear`、`/handoff`、Subagent、`/compact`。决策树从上到下，第一个"是"获胜。

#### 独立 Skills

- `/grill-me`：无状态的深度访谈（不在工作目录中时使用）
- `/grilling`：访谈原语
- `/resolving-merge-conflicts`：逐个 hunk 解决合并冲突
- `/prototype`：一次性原型
- `/research`：后台 Agent 研究
- `/to-questionnaire`：将决策转化为问卷
- `/wizard`：引导人类完成只有人类能做的步骤
- `/wait-what`：当消息没理解时重新表述
- `/teach`：多会话教学
- `/writing-for-agents`：为 Agent 编写文档的参考

### 前置条件

运行 `/setup-matt-pocock-skills` 来配置 Issue 追踪器、分诊标签和文档布局。

---

## 第二部分：原文与翻译

---

<!--原文-->
> # Ask Matt
>
> You don't remember every skill, so ask.

<!--翻译-->
> # 问 Matt
>
> 你记不住每个 skill，所以直接问。

---

<!--原文-->
> A **flow** is a path through the skills. Most paths run along one **main flow**, and two **on-ramps** merge onto it. Everything else is standalone, or a vocabulary layer that runs underneath.

<!--翻译-->
> **流程（flow）**是穿越各 skills 的一条路径。大多数路径沿着一条**主流程**运行，两条**接入匝道（on-ramp）**汇入其中。其他所有内容要么是独立的，要么是运行在底层的词汇层。

---

<!--原文-->
> ## The main flow: idea → ship
>
> The route most work travels. You have an idea and want it built.

<!--翻译-->
> ## 主流程：想法 → 交付
>
> 大多数工作经历的路线。你有一个想法，并希望将其构建出来。

---

<!--原文-->
> 1. **`/grill-with-docs`** sharpens the idea by interview. Start here whenever you are **working in a working directory**: it's stateful, retaining what it learns in `CONTEXT.md` and ADRs. (No working directory? Use `/grill-me` instead, covered under Standalone. Both run the same `/grilling` primitive; `grill-with-docs` is the one that leaves a paper trail, which makes it the better of the two whenever a repo is there to leave it in.)

<!--翻译-->
> 1. **`/grill-with-docs`** 通过访谈来细化想法。当你在**工作目录中工作时**始终从这里开始：它是有状态的，会在 `CONTEXT.md` 和 ADR 中保留学到的东西。（没有工作目录？使用 `/grill-me`，参见"独立 Skills"部分。两者运行相同的 `/grilling` 原语；`grill-with-docs` 会留下书面记录，因此只要有仓库可以存放，它就是两者中更好的选择。）

---

<!--原文-->
> 2. **Branch: can you settle every question in conversation?** If a question needs a runnable answer (state, business logic, a UI you have to see), detour through a prototype, bridged by **`/handoff`** in both directions (a prototype lives in its own directory, which is exactly what `/handoff` is for; see Phase boundaries):
>    - **`/handoff`** out, then open a fresh session against that file,
>    - **`/prototype`** to answer the question with throwaway code,
>    - **`/handoff`** back what you learned, and reference it from the original idea thread.

<!--翻译-->
> 2. **分支：你能在对话中解决所有问题吗？** 如果某个问题需要可运行的答案（状态、业务逻辑、你必须看到的 UI），那就绕道通过原型，用 **`/handoff`** 双向桥接（原型存在于自己的目录中，这正是 `/handoff` 的用途；参见阶段边界）：
>    - 用 **`/handoff`** 导出，然后对该文件打开一个新会话，
>    - 用 **`/prototype`** 以一次性代码回答问题，
>    - 用 **`/handoff`** 将所学内容带回，并从原始想法线程中引用它。

---

<!--原文-->
> 3. **Branch: is this a multi-session build?**
>    - **Yes** → **`/to-spec`** (turn the thread into a spec), then **`/to-tickets`** to split it into tracer-bullet tickets, each declaring its **blocking edges**. On a local tracker that's one file per ticket under `.scratch/<feature>/issues/`, worked blockers-first by hand; on a real tracker the edges become native blocking links, so any ticket whose blockers are done can be grabbed: kick off **`/implement`** per ticket, **`/clear`ing context between each one**. Each ticket is self-contained, so the last one's context is disposable.
>    - **No** → **`/implement`** right here, in the same context window.
>
>    Either way, **`/implement`** builds each issue by driving **`/tdd`** internally (one red-green slice at a time), then closes out by running **`/code-review`**, a two-axis review (Standards + Spec) of the diff, before committing. Reach for **`/tdd`** on its own when you just want to build a concrete behaviour test-first without a full spec, and **`/code-review`** on its own whenever you want to review a branch or PR against a fixed point.

<!--翻译-->
> 3. **分支：这是多会话构建吗？**
>    - **是** → **`/to-spec`**（将线程转化为 Spec），然后 **`/to-tickets`** 将其拆分为追踪子弹工单，每个工单声明其**阻塞边**。在本地追踪器中，每个工单一个文件位于 `.scratch/<feature>/issues/` 下，手动按阻塞者优先处理；在真正的追踪器上，阻塞边变为原生阻塞链接，任何阻塞者已完成的工单都可以被抓取：对每个工单启动 **`/implement`**，在工单之间 **`/clear`** 上下文。每个工单自包含，所以最后一个工单的上下文是可丢弃的。
>    - **否** → 就在此处，在同一上下文窗口中 **`/implement`**。
>
>    无论哪种方式，**`/implement`** 通过内部驱动 **`/tdd`** 来构建每个 Issue（一次一个红-绿切片），然后在提交前通过运行 **`/code-review`**收尾——对 diff 进行双轴审查（标准 + Spec）。当你想在没有完整 Spec 的情况下以测试先行方式构建具体行为时，单独使用 **`/tdd`**；当你想针对固定点审查分支或 PR 时，单独使用 **`/code-review`**。

---

<!--原文-->
> ### Context hygiene
>
> Keep steps 1–3 in **one unbroken context window** (don't compact or clear until after `/to-tickets`) so the grilling, spec, and tickets all build on the same thinking. Each `/implement` then starts fresh, working from the ticket.
>
> The limit on this is the **[smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**: the window (~150k tokens on state-of-the-art models) within which the model still reasons sharply. If a session approaches it before `/to-tickets`, don't push on degraded; `/compact` at the nearest phase boundary and carry on (see Phase boundaries).

<!--翻译-->
> ### 上下文卫生
>
> 将步骤 1-3 保持在一个**不间断的上下文窗口**中（在 `/to-tickets` 之前不要 compact 或 clear），这样访谈、Spec 和工单都建立在相同的思考基础上。每个 `/implement` 然后从工单开始全新启动。
>
> 这受到 **[智能区（smart zone）](https://www.aihero.dev/ai-coding-dictionary/smart-zone)** 的限制：窗口（在最先进模型上约 150K tokens），在此范围内模型仍然能清晰推理。如果会话在 `/to-tickets` 之前接近此限制，不要在质量下降的情况下继续；在最近的阶段边界处 `/compact` 然后继续（参见阶段边界）。

---

<!--原文-->
> ## On-ramps
>
> A starting situation that generates work, then merges onto the main flow.

<!--翻译-->
> ## 接入匝道
>
> 一种产生工作的起始情况，然后汇入主流程。

---

<!--原文-->
> - **Bugs and requests piling up** → **`/triage`**. It moves issues through triage roles and produces agent-ready issues, which **`/implement`** later picks up.
>
>   Triage is only for issues **you didn't create**: bug reports, incoming feature requests, anything that arrives raw. Tickets that `/to-tickets` produced are already agent-ready, so **don't triage them**.

<!--翻译-->
> - **Bug 和请求堆积** → **`/triage`**。它通过分诊角色流转 Issue，产出 Agent 就绪的 Issue，之后由 **`/implement`** 拾取。
>
>   分诊仅适用于**不是你创建的** Issue：Bug 报告、传入的功能请求、任何原始到达的内容。`/to-tickets` 生成的工单已经是 Agent 就绪的，所以**不要对它们进行分诊**。

---

<!--原文-->
> - **Something's broken** → **`/diagnosing-bugs`**. For the hard ones: the bug that resists a first glance, the intermittent flake, the regression that crept in between two known-good states. It refuses to theorise until it has a **tight feedback loop** (one command that already goes red on *this* bug), then fixes with a regression test. Its post-mortem hands off to **`/improve-codebase-architecture`** when the real finding is that there's no good seam to lock the bug down.

<!--翻译-->
> - **某物坏了** → **`/diagnosing-bugs`**。针对困难的问题：一眼看不出的 Bug、间歇性的 flake、在两个已知良好状态之间悄悄出现的回归。在拥有**紧反馈循环**（一个已经在此 Bug 上变红的命令）之前，它拒绝进行理论推测，然后用回归测试修复。当事后分析发现真正的问题是"没有好的接缝来锁定这个 Bug"时，它会移交给 **`/improve-codebase-architecture`**。

---

<!--原文-->
> - **A huge, foggy effort: a greenfield project or a huge feature build, too big for one session** → **`/wayfinder`**, the most cognitively demanding flow here. When the way from here to the destination isn't visible yet, it charts a **shared map** of **decision tickets** on the issue tracker and resolves them one at a time, producing **decisions, not deliverables**, until the fog is pushed back and the way is clear. Where **`/grill-with-docs`** sharpens an idea you can hold in one session, wayfinder is for the idea you can't, and it's slower and denser, so save it for exactly that, never a well-scoped feature.
>
>   When the map clears, **it hands off, it doesn't build**: merge onto the main flow at **`/to-spec`**, which collapses the map's linked decisions into a buildable plan, then `/to-tickets` and `/implement` as usual. Looping the map straight into `/implement` skips that collapse and throws the linked detail away, so go straight to `/implement` only when the effort turned out genuinely small.

<!--翻译-->
> - **一个巨大、模糊的工作：绿地项目或超大功能构建，对一个会话来说太大了** → **`/wayfinder`**，这里认知需求最高的流程。当从这里到目的地的路径尚不可见时，它在 Issue 追踪器上绘制**决策工单**的**共享地图**，逐个解决，产出**决策而非交付物**，直到迷雾被驱散，路径清晰。**`/grill-with-docs`** 细化的是你能在一个会话中把握的想法，而 wayfinder 针对的是你无法在一个会话中把握的想法，它更慢、更密集，所以只在这种情况下使用，绝不要用于范围明确的功能。
>
>   当迷雾驱散后，**它进行移交，而不是构建**：在 **`/to-spec`** 处汇入主流程，将地图上关联的决策折叠为可构建的计划，然后照常 `/to-tickets` 和 `/implement`。直接将地图循环到 `/implement` 会跳过这个折叠步骤并丢弃关联的细节，所以只有在工作确实很小的情况下才直接进入 `/implement`。

---

<!--原文-->
> ## Codebase health
>
> Not feature work, just upkeep.
>
> - **`/improve-codebase-architecture`** runs whenever you have a spare moment to keep the codebase good for agents to operate in. It surfaces **deepening opportunities**; picking one _generates an idea_ you can take into the main flow at `/grill-with-docs`. It's the survey that finds the candidates; **`/codebase-design`** (below) is the bench you design the chosen one on.

<!--翻译-->
> ## 代码库健康
>
> 不是功能工作，只是维护。
>
> - **`/improve-codebase-architecture`** 在你有空时运行，以保持代码库适合 Agent 操作。它浮现**深化机会**；选择一个会_生成一个想法_，你可以将其带入主流程的 `/grill-with-docs`。它是找到候选者的调查；**`/codebase-design`**（见下文）是你设计所选候选者的工作台。

---

<!--原文-->
> ## Vocabulary underneath
>
> Two model-invoked references that run *beneath* the other skills, each the single source of truth for its vocabulary. Reach for them directly when the **words**, not the process, are the problem; or let the skills above pull them in.

<!--翻译-->
> ## 底层词汇
>
> 两个模型调用的参考，运行在其他 Skills *之下*，每个都是其词汇的单一真相来源。当问题是**词语**而非流程时，直接使用它们；或者让上层的 Skills 来调用它们。

---

<!--原文-->
> - **`/domain-modeling`**: sharpen the project's *domain* language: challenge a fuzzy term, resolve an overloaded word ("account" doing three jobs), record a hard-to-reverse decision as an ADR. It's the active discipline `/grill-with-docs` drives to keep `CONTEXT.md` a clean glossary.
> - **`/codebase-design`** is the deep-module vocabulary (module, interface, depth, seam, adapter, leverage, locality) for designing a module's *shape*: a lot of behaviour behind a small interface at a clean seam. `/tdd` and `/improve-codebase-architecture` both speak it.

<!--翻译-->
> - **`/domain-modeling`**：优化项目的*领域*语言：挑战模糊术语，解决重载词汇（"account"做三件事），将难以逆转的决策记录为 ADR。它是 `/grill-with-docs` 驱动的主动纪律，旨在保持 `CONTEXT.md` 是一个干净的术语表。
> - **`/codebase-design`** 是深层模块词汇（模块、接口、深度、接缝、适配器、杠杆、局部性），用于设计模块的*形态*：在一个干净的接缝处，小接口背后承载大量行为。`/tdd` 和 `/improve-codebase-architecture` 都使用这套语言。

---

<!--原文-->
> ## Phase boundaries
>
> A **phase** is a chunk of work inside a session: the grilling, the implementation, the QA. At the **boundary** between two of them you have five options, and picking between them is the fuzziest decision in this whole map:
>
> - **Continue**: stay put. Costs nothing, loses nothing.
> - **`/clear`**: empty the window, when nothing here matters to what's next.
> - **`/handoff`** writes a portable markdown file. Narrow: only for a **new harness**, a **new directory**, a **colleague**, or forking a side task **mid-phase**. What it buys is portability.
> - **Subagent**: send a tightly-scoped task to its own window and get a report back.
> - **`/compact`** compresses this context and seeds a fresh session with it. The **default**, at the bottom of the tree rather than the first reach.
>
> Read [PHASE-BOUNDARIES.md](PHASE-BOUNDARIES.md) for the ordered tree: the five questions, the reasoning behind each branch, and why the primary-source cost makes **Continue** the one to rule out first. Make the decision **at** a boundary; mid-phase, continue or split the rest into subagents.

<!--翻译-->
> ## 阶段边界
>
> **阶段（phase）**是会话中的一块工作：访谈、实现、QA。在两者之间的**边界**处，你有五个选择，在这个整个地图中这是最模糊的决策：
>
> - **Continue（继续）**：原地不动。零成本，零损失。
> - **`/clear`（清空）**：清空窗口，当这里的一切对下一步都不重要时。
> - **`/handoff`（交接）**：写一个可移植的 markdown 文件。范围窄：仅用于**新 harness**、**新目录**、**同事**，或在**阶段中途**分叉一个侧任务。它购买的是可移植性。
> - **Subagent（子 Agent）**：将一个范围紧凑的任务发送到自己的窗口并获取报告。
> - **`/compact`（压缩）**：压缩此上下文并用它播种一个新会话。这是**默认选项**，位于决策树底部而非首选。
>
> 阅读 [PHASE-BOUNDARIES.md](PHASE-BOUNDARIES.md) 了解有序决策树：五个问题，每个分支背后的推理，以及为什么一手来源的成本使 **Continue** 成为第一个要排除的选项。在边界**处**做出决定；阶段中途，继续或将剩余部分拆分为子 Agent。

---

<!--原文-->
> ## Standalone
>
> Off the main flow entirely.

<!--翻译-->
> ## 独立 Skills
>
> 完全脱离主流程。

---

<!--原文-->
> - **`/grill-me`**: the same relentless interview as `/grill-with-docs`, but **stateless**: it saves nothing locally and builds no `CONTEXT.md`. Reach for it when you are **not working in a working directory** (sharpening a plan, a design, a piece of writing, anything with no repo under it). If you are in a working directory, use `/grill-with-docs` instead: it runs the same interview and leaves a paper trail, so it is strictly the better one.

<!--翻译-->
> - **`/grill-me`**：与 `/grill-with-docs` 相同的无情访谈，但是**无状态的**：不在本地保存任何内容，不构建 `CONTEXT.md`。当**不在工作目录中工作时**使用它（细化计划、设计、写作内容，任何没有仓库支撑的东西）。如果你在工作目录中，请改用 `/grill-with-docs`：它运行相同的访谈并留下书面记录，因此严格来说是更好的选择。

---

<!--原文-->
> - **`/grilling`** is the interview primitive itself: rounds, the frontier, facts are the agent's job and decisions are yours. `/grill-me` and `/grill-with-docs` are the two named ways in, and `/triage`, `/wayfinder` and `/improve-codebase-architecture` all run it internally. Reach for it directly only when you want the interview with no wrapper around it.

<!--翻译-->
> - **`/grilling`** 是访谈原语本身：轮次、前沿、事实是 Agent 的工作，决策是你的。`/grill-me` 和 `/grill-with-docs` 是两种命名的进入方式，`/triage`、`/wayfinder` 和 `/improve-codebase-architecture` 都在内部运行它。只有当你想要不带任何包装的访谈时才直接使用它。

---

<!--原文-->
> - **`/resolving-merge-conflicts`** works an in-progress merge or rebase conflict hunk by hunk, resolving by **intent** traced to each side's primary source rather than by picking lines, then finishes the operation. It never runs `--abort`. Standalone and off every flow: reach for it when you are already mid-conflict.

<!--翻译-->
> - **`/resolving-merge-conflicts`** 逐个 hunk 处理进行中的 merge 或 rebase 冲突，通过追溯到每方一手来源的**意图**来解决（而非挑选行），然后完成操作。它绝不运行 `--abort`。独立且脱离所有流程：当你已经处于冲突中时使用它。

---

<!--原文-->
> - **`/prototype`** is a small, throwaway program that answers one design question: does this state model feel right, or what should this UI look like. Throwaway is a constraint on how the code is written, not a promise to destroy it: the answer folds into the real code, and the prototype itself is kept as a **primary source** on a `prototype/<name>` branch out of main, pointed at from the implementation issue. It's the detour in step 2 of the main flow, but reach for it any time a design question is hard to settle on paper.

<!--翻译-->
> - **`/prototype`** 是一个小型、一次性程序，回答一个设计问题：这个状态模型感觉对吗，或者这个 UI 应该是什么样子。一次性是代码编写方式的约束，而不是销毁它的承诺：答案融入真实代码，原型本身作为**一手来源**保留在脱离 main 的 `prototype/<name>` 分支上，从实现 Issue 指向它。它是主流程步骤 2 中的绕道，但任何时候设计问题难以在纸面上解决时都可以使用它。

---

<!--原文-->
> - **`/research`**: delegate reading legwork to a **background agent**: it investigates a question against **primary sources**, then leaves a cited Markdown file in the repo. Keep working while it reads. The file it produces is something to take *into* the main flow at `/grill-with-docs`, since research feeds the thinking rather than replacing it.

<!--翻译-->
> - **`/research`**：将阅读工作委托给**后台 Agent**：它针对**一手来源**调查问题，然后在仓库中留下一份带有引用的 Markdown 文件。在它阅读时你继续工作。它生成的文件是要带入主流程 `/grill-with-docs` 的内容，因为研究是滋养思考的，而非替代它。

---

<!--原文-->
> - **`/to-questionnaire`** comes in when the thing blocking you isn't in your head or the codebase but in **someone else's**, and it writes them a questionnaire to fill in. It's the inverse of `/grill-me`: instead of interviewing you about the subject, it interviews you about the **send** (who it's going to, what you need back) and aims the questions at the gap. What comes back is material for `/grill-with-docs` or `/to-spec`.

<!--翻译-->
> - **`/to-questionnaire`** 在阻碍你的东西不在你的头脑或代码库中而是在**别人的**头脑中时登场，它为对方写一份问卷让其填写。它是 `/grill-me` 的反面：不是就主题访谈你，而是就**发送**（发给谁、你需要什么回报）访谈你，并将问题瞄准缺口。返回的内容是 `/grill-with-docs` 或 `/to-spec` 的素材。

---

<!--原文-->
> - **`/wizard`** is for the steps only a **human** can take: provisioning infrastructure, setting up credentials or CI secrets, clicking through an unfamiliar third-party dashboard, running a one-off migration or cutover. It generates an interactive bash script that opens each URL, captures each value, and writes it into `.env` and GitHub secrets, so the procedure stops being something you re-explain to an agent every time. Model-invoked, so the agent reaches for it the moment it hits a wall only you can pass. If the agent could just do it itself, it should; this is for where a human is genuinely in the loop.

<!--翻译-->
> - **`/wizard`** 用于只有**人类**能执行的步骤：配置基础设施、设置凭据或 CI 密钥、在陌生的第三方仪表板中点击操作、运行一次性迁移或切换。它生成一个交互式 bash 脚本，打开每个 URL，说明点击和复制什么，捕获每个值，并将其写入 `.env` 和 GitHub secrets，这样这个过程就不再是每次都要向 Agent 重新解释的东西。它是模型调用的，因此 Agent 在遇到只有你能通过的障碍时会立即使用它。如果 Agent 可以自己做，它就应该做；这用于人类确实在循环中的情况。

---

<!--原文-->
> - **`/wait-what`** is the corrective for a message that didn't land. Use it mid-conversation, inside any other skill, and the agent re-pitches what it just said with the context you were missing, in plain English, using the `CONTEXT.md` vocabulary. It works after the fact; `/grill-with-docs` is the upfront cure, because a shared language agreed early is what stops the jargon arriving at all.

<!--翻译-->
> - **`/wait-what`** 是对未能理解的消息的纠正。在对话中途，在任何其他 skill 内部使用它，Agent 会用你缺失的上下文、以通俗英语、使用 `CONTEXT.md` 词汇重新表述它刚说的内容。它在事后起作用；`/grill-with-docs` 是前瞻性的解决方案，因为早期达成一致的共享语言是从根本上阻止行话出现的方法。

---

<!--原文-->
> - **`/teach`**: learn a concept over multiple sessions, using the current directory as a stateful workspace.
> - **`/writing-for-agents`** is the reference for writing documents agents consume: skills, AGENTS.md, pointed-at docs.

<!--翻译-->
> - **`/teach`**：通过多个会话学习一个概念，使用当前目录作为有状态的教学工作空间。
> - **`/writing-for-agents`** 是为 Agent 编写文档的参考：Skills、AGENTS.md、被指向的文档。

---

<!--原文-->
> ## Precondition
>
> **`/setup-matt-pocock-skills`**: run before your first engineering flow to configure the issue tracker, triage labels, and doc layout the other skills assume. Custom issue trackers also work.

<!--翻译-->
> ## 前置条件
>
> **`/setup-matt-pocock-skills`**：在第一次工程流程之前运行，配置其他 Skills 所假设的 Issue 追踪器、分诊标签和文档布局。自定义 Issue 追踪器同样适用。