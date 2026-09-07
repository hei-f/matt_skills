# writing-for-agents — 为 Agent 撰写文档

## 第一部分：详细总结

### 概述

`writing-for-agents` 是撰写任何 Agent 消费文档的通用参考指南。无论文档的最终形态是 Skill、`AGENTS.md` / `CLAUDE.md`，还是通过指针引用的附属文档，其写作原则和杠杆都是相同的。核心目标是让 Agent 在每次运行时走相同的*过程*，而非生成相同的输出。当文档是一个 Skill 时，还需参考 `SKILL-MECHANICS.md` 了解 frontmatter、调用方式选择（模型调用 vs 用户调用）和路由 Skill。

### 核心概念：上下文指针（Context Pointers）

上下文指针是保留在 Agent 上下文中的一个引用，它命名了某个上下文外的材料，并编码了触达该材料的条件。一个 Skill 的 description 就是一个上下文指针；`AGENTS.md` 中命名某个文档的一行也是同一类对象。指针的*措辞*——而非其目标——决定了 Agent 何时以及多可靠地触达目标材料。一个措辞薄弱的指针背后有一个必须被触达的目标，就是一个方差 Bug：先强化措辞，只有当强化失败时才内联材料。

指针做两件事：声明材料是什么，以及列出应触发它的**分支**（分支是文档处理的不同情况，不同运行会走不同路径）。始终加载的指针每个词在每一轮都会产生成本，因此其裁剪比正文更严格：

- **前置核心词（leading word）**：指针是它做触发工作的位置
- **一个分支一个触发器**：把重命名同一个分支的同义词合并为一个分支，只保留真正不同的分支
- **砍掉正文已携带的身份信息**

### 两种负载（The Two Loads）

每个文档和指针都会消耗两种预算之一：

- **上下文负载（Context Load）**：始终加载的材料对 Agent 窗口的成本。`AGENTS.md` 中的一行、Skill 的 description，这些每轮都在上下文中，无论是否被触发都要消耗 token 和注意力。
- **认知负载（Cognitive Load）**：对人类施加的成本：哪些文档存在、何时使用哪个。人类是索引器。这不是要最小化的成本（它是人类主动性的代价），而是在人类判断有价值的地方投入，在无价值的地方移除。

仅通过指针触达的材料用指针自身的行来换取免于上下文负载；完全没有指针的材料则完全承载在认知负载上。

### 信息层级（Information Hierarchy）

文档由两种内容类型构成：**步骤**（Agent 执行的有序动作）和**参考**（按需查询的定义、规则、事实）。两者自由混合。核心决策是每段材料在信息层级上的位置，层级按 Agent 需要材料的紧急程度排序：

1. **文件内步骤（In-file step）**：主层级，Agent 做什么、按什么顺序
2. **文件内参考（In-file reference）**：按需查阅。通常是一组合法的扁平同级集（如 Code Review 的每条规则在同一级），这并非坏味道
3. **披露参考（Disclosed reference）**：被推到单独文件中，由上下文指针触达，仅在指针触发时加载。范围从同目录的兄弟文件到完全外部引用

放得太少，顶层会膨胀；放得太多，会隐藏 Agent 真正需要的材料。这种张力就是全部决策。

**渐进披露（Progressive Disclosure）**：沿层级向下移动（从主文件中移出、放到指针后面），使顶层保持清晰。这不主要是 token 优化，而是对层级的保护。分支是最好的披露测试：将每个分支都需要的材料内联，将只有某些分支才需要的推到指针后面。当文档有步骤时，应该被披露的文件内参考会淹没它们，使关注步骤变成掷硬币——这不仅是可读性杠杆，更是方差杠杆。

**就近原则（Co-location）**：文件内的配套原则。层级决定材料放在多靠下的位置，就近原则决定一旦放在那里后旁边放什么。将概念的定义、规则和注意事项放在同一标题下而非分散，使得阅读其中一部分能顺便带上邻近部分。检验标准：文档读起来应该像写给 Agent 的说明书。分组排列的材料读起来是这样，分散的则不是。（与重复区分：重复是在两个地方重复同一含义；分散是把一个含义的片段散布在多个地方。）

**臃肿（Sprawl）**：这是信息层级的失败模式——文档太长，即使每一行都是活的和独特，注意力也会分散，每多一行就多一行需要保持相关的内容。解法是层级：将参考披露到指针后面，按分支或序列拆分，使每条路径只承载它所需的内容。

### 步骤与完成标准（Steps and Completion Criteria）

每个步骤都以**完成标准**结束——告诉 Agent 工作完成的条件。两个属性使其成为杠杆：

- **清晰度（Clarity）**：Agent 能否区分完成和未完成？模糊的边界（如"达到理解"）会引发**过早完成**：在真正完成之前就结束步骤，注意力滑向"已完成"。可见的后续步骤（后完成步骤）提供拉力；完成标准的清晰度提供阻力。防守顺序：**先锐化边界**（局部的、便宜的）；只有在边界不可约简的模糊*并且*你观察到仓促完成时，才通过拆分序列来隐藏后续步骤。隐藏仅在跨真正上下文边界时才有效（hand-off 或子 Agent 派遣；内联调用会将后续步骤留在上下文中，无法清理）。

- **要求量（Demand）**：标准要求多少。"每个修改过的模型都被考虑"会迫使深入工作，而"产出一份变更清单"则不会。要求量驱动**脚踏实地的工作量（legwork）**：Agent 在产出过程中的挖掘，潜藏在措辞中而非作为独立步骤书写。它不局限于步骤："每条规则都应用"对一系列扁平参考的约束力，等同于"每个步骤都完成"对序列的约束力——全参考文档也能携带完备性标准。

最强的标准既**可检查**又**完备**。

### 何时拆分（When to Split）

将一个文档拆成两个会消耗两种负载之一，所以仅在拆分带来的收益值得时才拆：

- **按序列拆分**：当后完成步骤的可见性诱使 Agent 仓促完成当前步骤时，将后续步骤移出视野，推动在当前任务上更多的脚踏实地工作量。注意反面：合并序列会使每个步骤的后续步骤暴露给后续内容，引发过早完成。
- **按调用拆分**（Skill 专属）：见 `SKILL-MECHANICS.md`。

### 核心词（Leading Words）

核心词是已存在于模型预训练中的紧凑概念，Agent 在运行文档时借以思考（*lesson*、*fog of war*、*tracer bullets*）。它作为一个 token——而非一个句子——被重复，不断累积分布式定义，以最少的 token 锚定整个行为区域，通过调用模型已有的先验。自定义一个词也行（如果定义得足够清晰），但一个编造的词不调用任何先验：你在定义 token 上付出的成本，一个预训练词可以免费获得。先尝试用已有的词。

核心词锚定两次。在正文中是*执行*：Agent 每次看到这个词就触达相同的行为；在扁平参考内部，它将注意力聚焦在一类要查找的事物上。在指针中是*调用*：当同一个词存在于你的提示、文档和代码库中时，Agent 将共享语言链接到材料并更可靠地触达它。

寻找用核心词重构的机会。三个地方拼出一个三元概念、一个指针花一整句话手势一个想法——每个段落都在恳求坍缩成单个 token：
- "快、确定性、低开销" → *tight*（紧凑循环）
- "一个你信任的循环" → *red*（循环变红表示踩到 Bug，或没有）

赢了两次：更少 token，以及 Agent 思考的更强锚点。假设每个文档都携带着可以被核心词退化的重复陈述。去找它们。

**否定（Negation）** 是这个杠杆旁边的失败模式：通过禁止来引导会把被禁止的行为拖入上下文，使其*更容易*被获取，而非更难。不要想大象，大象就无处不在；否定只是被强激活概念碾压的弱修饰符，所以禁令半读作执行命令。提示**正向**行为：陈述目标行为（"写单行注释"），被禁止的行为根本不提。禁止仅在无法用正向措辞的硬护栏时才有存在意义；即便如此，也要配以正向目标，使注意力落在该做什么上。

### 裁剪原则（Pruning）

- 将每个含义保留在**单一真实来源**中：一个权威位置，改变行为只需一次编辑。**重复**（同一含义出现在多个位置）消耗维护和 token，并膨胀含义在层级上的显眼度至超过其真实排名。（核心词的有意对立面：有意重复 token，从不重复含义。）
- **环境**也是一个真实来源（`package.json` 脚本、配置文件、目录布局、`--help` 输出），重述环境的文档是**缓存**：一次查找的拷贝，仅在查找昂贵时才值得付费。缓存 Agent 查找不到的内容：不成文的惯例、选择背后的原因、配置文件不会承认的陷阱。将单文件、单命令的查找留给环境，它们在那里不会过时。
- 检查每一行的**相关性**：它仍然关系到文档的职责吗？一行通过从未与任务相关（纯粹的说明，或应被披露的分支）或随着描述的行为或世界变化而过时，来失去相关性。更短的文档更容易保持相关性。没有裁剪纪律，默认命运是**沉积**：过时图层因添加感到安全而删除感觉危险而堆积，直到你必须穿透它们找出仍然存活的内容。
- 逐句搜寻**空操作（no-ops）**：模型默认已遵守的指令，负载白付了。测试（它是否改变与默认行为的差异？）是相对于模型而非读者的：两人关于空操作的分歧是关于默认行为的分歧，通过运行文档来解决，而非辩论。当一句失败，整句删除而非削减词数。此测试也评定核心词级别：一个太弱而无法击败默认行为的词（Agent 已经大致是 thorough，而你说 *be thorough*）是空操作，解决方式是更强的词（*relentless*），而非更换技术。

### 补充文件

- **SKILL-MECHANICS.md**：Skill 专属分支，涵盖 frontmatter、调用方式选择（模型调用 vs 用户调用）、路由 Skill。模型调用的 Skill 保留 description 供 Agent 自主发现；用户调用的 Skill 设置 `disable-model-invocation: true`，零上下文负载但消耗认知负载。

---

## 第二部分：原文与翻译

<!--原文-->
> # Writing for Agents
> Reference for writing any document an agent consumes: a skill, an `AGENTS.md` / `CLAUDE.md`, a doc reached by a pointer. The packaging differs; the writing does not: the same levers make each one predictable, since the agent takes the same _process_ every run rather than producing the same output.

<!--翻译-->
> # 为 Agent 撰写文档
> 撰写 Agent 消费的任何文档的参考指南：一个 Skill、一份 `AGENTS.md` / `CLAUDE.md`、一份通过指针触达的文档。包装形式不同，写作方式不变：相同的杠杆使每份文档都可预测，因为 Agent 每次运行走相同的*过程*，而非生成相同的输出。

<!--原文-->
> ## Context pointers
> A **context pointer** is a reference held in the agent's context that names some out-of-context material and encodes the condition for reaching it. A skill's description is one; a line in `AGENTS.md` naming a doc is the same object. The pointer's _wording_, not its target, decides when the agent reaches the material, and how reliably. A must-have target behind a weakly worded pointer is a variance bug: sharpen the wording first, and inline the material only if sharpening fails.

<!--翻译-->
> ## 上下文指针
> **上下文指针**是保留在 Agent 上下文中的引用，它命名了某个上下文外的材料并编码了触达它的条件。一个 Skill 的 description 就是一个指针；`AGENTS.md` 中命名文档的一行是同一对象。指针的*措辞*——而非其目标——决定 Agent 何时以及多可靠地触达该材料。一个措辞薄弱的指针背后有一份必须被触达的目标材料，就是一个方差 Bug：先锐化措辞，仅当锐化失败时才将材料内联。

<!--原文-->
> A pointer does two jobs: state what the material is, and list the **branches** that should trigger reaching it (a branch is a distinct case the document handles, so different runs take different paths through it). Every word of an always-loaded pointer costs on every turn, so it earns even harder pruning than the body:
> - **Front-load the leading word**: the pointer is where it does its triggering work.
> - **One trigger per branch.** Synonyms that rename a single branch are one branch written twice; collapse them and keep only genuinely distinct branches.
> - **Cut identity the body already carries.**

<!--翻译-->
> 指针做两个工作：声明材料是什么，并列出应触发它的**分支**（分支是文档处理的独立情况，不同运行走不同路径）。始终加载的指针每个词在每一轮都产生成本，因此其裁剪比正文更严格：
> - **将核心词前置**：指针是它做触发工作的地方。
> - **一个分支一个触发器。** 重命名同一个分支的同义词是一个分支写了两次；合并它们，只保留真正不同的分支。
> - **砍掉正文已携带的身份信息。**

<!--原文-->
> ## The two loads
> Every document and pointer you add spends one of two budgets:
> - **Context load** is the cost of always-loaded material on the agent's window: an `AGENTS.md` line, a skill description, anything sitting in context every turn, spending tokens and attention whether or not it fires.
> - **Cognitive load** is the cost on the human: which documents exist and when to reach for each. The human is the index. Not a cost to minimise: it is the price of human agency; spend it where human judgement matters, remove it where it does not.

<!--翻译-->
> ## 两种负载
> 你添加的每个文档和指针都会消耗两种预算之一：
> - **上下文负载**是始终加载的材料对 Agent 窗口的成本：`AGENTS.md` 中的一行、Skill 的 description、任何每轮都在上下文中的东西，无论是否被触发都在消耗 token 和注意力。
> - **认知负载**是对人类施加的成本：哪些文档存在、何时使用每个。人类是索引器。这不是要最小化的成本：它是人类主动性的代价；在人类判断有价值的地方投入，在无价值的地方移除。

<!--原文-->
> Material reached only through a pointer escapes context load at the price of the pointer's own line; material with no pointer at all rides entirely on cognitive load.

<!--翻译-->
> 仅通过指针触达的材料以指针自身的行的代价换取免于上下文负载；完全没有指针的材料则完全承载在认知负载上。

<!--原文-->
> ## Information hierarchy
> A document is built from two content types: **steps** (the ordered actions the agent performs) and **reference** (definitions, rules, facts consulted on demand). The two mix freely: all steps (a recipe), all reference (a review's rules, this skill), or both. The core decision is where each piece sits on the **information hierarchy**, a ladder ranked by how immediately the agent needs the material:
> 1. **In-file step** is the primary tier: what the agent does, in order.
> 2. **In-file reference** is consulted on demand. Often a legitimately flat peer-set (every rule of a review on one rung), which is a fine arrangement, not a smell.
> 3. **Disclosed reference** is pushed out into a separate file, reached by a context pointer, loaded only when the pointer fires. Spans a sibling file in the same folder through fully external reference that lives anywhere and any document can point at.

<!--翻译-->
> ## 信息层级
> 文档由两种内容类型构成：**步骤**（Agent 执行的有序动作）和**参考**（按需查询的定义、规则、事实）。两者自由混合：全部步骤（一份食谱）、全部参考（Review 的规则、本 Skill 自身）、或两者兼备。核心决策是每段材料在**信息层级**上的位置，层级按 Agent 需要材料的紧急程度排序：
> 1. **文件内步骤**是主层级：Agent 做什么，按什么顺序。
> 2. **文件内参考**是按需查阅的。通常是一组合法的扁平同级集（Review 的每条规则在同一级），这是好的安排，并非坏味道。
> 3. **披露参考**被推到单独文件中，由上下文指针触达，仅在指针触发时加载。范围从同目录兄弟文件到完全外部引用，可存在于任何位置并被任何文档指向。

<!--原文-->
> Push too little down and the top bloats; push too much and you hide material the agent actually needs. That tension is the whole decision.

<!--翻译-->
> 放得太少，顶层会膨胀；放得太多，会隐藏 Agent 真正需要的材料。这种张力就是全部决策。

<!--原文-->
> **Progressive disclosure** is the move down the ladder (out of the main file and behind a pointer) so the top stays legible. Not primarily a token optimisation: it is how the hierarchy is protected. Branching is the cleanest disclosure test: inline what every branch needs, and push behind a pointer what only some branches reach. When a document has steps, in-file reference that should be disclosed buries them and turns attending to them into a coin-flip: a variance lever, not just a legibility one.

<!--翻译-->
> **渐进披露**是沿层级向下移动（从主文件中移出、放到指针后面），使顶层保持清晰。不主要是 token 优化：这是层级被保护的方式。分支是最干净的披露测试：将每个分支都需要的材料内联，将只有某些分支才需要的推到指针后面。当文档有步骤时，应该被披露的文件内参考会淹没它们，使关注步骤变成掷硬币——这是方差杠杆，而不仅仅是可读性杠杆。

<!--原文-->
> **Co-location** is the within-file companion: where the ladder decides _how far down_ a piece sits, co-location decides _what sits beside it_ once there. Keep a concept's definition, rules, and caveats under one heading rather than scattered, so reading one part brings its neighbours with it. The test: the document should read like documentation written for the agent. Grouped material reads that way; scattered material does not. (Distinct from duplication: that repeats one meaning in two places; scattering fragments one meaning across many.)

<!--翻译-->
> **就近原则**是文件内的配套原则：层级决定材料放在多靠下的位置，就近原则决定一旦放在那里后*旁边放什么*。将概念的定义、规则和注意事项放在同一标题下而非分散在各处，使得阅读其中一部分能顺便带上邻近部分。检验标准：文档读起来应该像写给 Agent 的说明书。分组排列的材料读起来是这样，分散的则不是。（与重复区分：重复是在两个地方重复同一含义；分散是把一个含义的片段散布在多个地方。）

<!--原文-->
> **Sprawl** is the failure mode here: a document simply too long, even when every line is live and unique. Attention thins across the excess, and every extra line is one more to keep relevant. The cure is the ladder: disclose reference behind pointers, and split by branch or sequence so each path carries only what it needs.

<!--翻译-->
> **臃肿**是这里的失败模式：文档就是太长了，即便每一行都是活的和独特的。注意力在超量中变薄，每多一行就多一行需要保持相关的内容。解法是层级：将参考披露到指针后面，按分支或序列拆分，使每条路径只承载它所需的内容。

<!--原文-->
> ## Steps and completion criteria
> Every step ends on a **completion criterion**, the condition that tells the agent the work is done. Two properties make it a lever:
> - **Clarity**: can the agent tell done from not-done? A vague bound ("understanding reached") invites **premature completion**: ending the step before it is genuinely done, attention slipping to _being done_. The visible steps still ahead (the **post-completion steps**) supply the pull; the criterion's clarity is the resistance. Defend in order: **sharpen the bound first** (local and cheap); only if it is irreducibly fuzzy _and_ you observe the rush, hide the later steps by splitting the sequence. Hiding only works across a real context boundary (a hand-off or a subagent dispatch; an inline call leaves the later steps in context and clears nothing).
> - **Demand**: how much it requires. "Every modified model accounted for" forces thorough work where "produce a change list" does not. Demand drives **legwork** (the digging the agent does within the work, latent in the wording rather than written as its own step), and it is not step-bound: "every rule applied" binds a body of flat reference just as "every step done" binds a sequence, which is how an all-reference document still carries an exhaustiveness bar.

<!--翻译-->
> ## 步骤与完成标准
> 每个步骤都以**完成标准**结束——告诉 Agent 工作完成的条件。两个属性使其成为杠杆：
> - **清晰度**：Agent 能否区分完成和未完成？模糊的边界（"达到理解"）会引发**过早完成**：在真正完成之前就结束步骤，注意力滑向*已完成*。可见的后续步骤（**后完成步骤**）提供拉力；完成标准的清晰度提供阻力。防守顺序：**先锐化边界**（局部的、便宜的）；只有在边界不可约简的模糊*并且*你观察到仓促完成时，才通过拆分序列来隐藏后续步骤。隐藏仅在跨真正上下文边界时才有效（hand-off 或子 Agent 派遣；内联调用会将后续步骤留在上下文中，无法清理）。
> - **要求量**：标准要求多少。"每个修改过的模型都被考虑"会迫使深入工作，而"产出一份变更清单"则不会。要求量驱动**脚踏实地的工作量**（Agent 在产出过程中的挖掘，潜藏在措辞中而非作为独立步骤书写）。它不局限于步骤："每条规则都应用"对一系列扁平参考的约束力，等同于"每个步骤都完成"对序列的约束力——全参考文档也能携带完备性标准。

<!--原文-->
> The strongest criteria are both checkable and exhaustive.

<!--翻译-->
> 最强的标准既可检查又完备。

<!--原文-->
> ## When to split
> Splitting one document into two spends one of the two loads, so split only when the cut earns it:
> - **By sequence**: split a run of steps where the post-completion steps tempt the agent to rush the one in front of it. Keeping them out of view drives more legwork on the current task. Beware the reverse: merging sequences exposes each step's later steps to what follows, inviting premature completion.
> - **By invocation**, skill-specific: see [`SKILL-MECHANICS.md`](SKILL-MECHANICS.md).

<!--翻译-->
> ## 何时拆分
> 将一个文档拆成两个会消耗两种负载之一，所以仅在拆分带来的收益值得时才拆：
> - **按序列拆分**：当后完成步骤的可见性诱使 Agent 仓促完成当前步骤时，将其移出视野，推动在当前任务上更多的脚踏实地工作量。注意反面：合并序列会使每个步骤的后续步骤暴露给后续内容，引发过早完成。
> - **按调用拆分**（Skill 专属）：见 `SKILL-MECHANICS.md`。

<!--原文-->
> ## Leading words
> A **leading word** is a compact concept already living in the model's pretraining that the agent thinks with while running the document (_lesson_, _fog of war_, _tracer bullets_). Repeated as a token, never as a sentence, it accumulates a distributed definition and anchors a whole region of behaviour in the fewest tokens, by recruiting priors the model already holds. Coining your own works if you define it clearly, but a made-up word recruits no priors: you pay in definition tokens what a pretrained word gives free; reach for an existing word first.

<!--翻译-->
> ## 核心词
> **核心词**是已存在于模型预训练中的紧凑概念，Agent 在运行文档时借以思考（*lesson*、*fog of war*、*tracer bullets*）。它作为一个 token——而非一个句子——被重复，不断累积分布式定义，以最少的 token 锚定整个行为区域，通过调用模型已有的先验。自定义一个词也行（如果定义得足够清晰），但一个编造的词不调用任何先验：你在定义 token 上付出的成本，一个预训练词可以免费获得；先尝试用已有的词。

<!--原文-->
> It anchors twice. In the body, _execution_: the agent reaches for the same behaviour every time the word appears, and inside flat reference it focuses attention on a class of thing to look for. In a pointer, _invocation_: when the same word lives in your prompts, your docs, and your codebase, the agent links that shared language to the material and reaches it more reliably.

<!--翻译-->
> 核心词锚定两次。在正文中是*执行*：Agent 每次看到这个词就触达相同的行为；在扁平参考内部，它将注意力聚焦在一类要查找的事物上。在指针中是*调用*：当同一个词存在于你的提示、文档和代码库中时，Agent 将共享语言链接到材料并更可靠地触达它。

<!--原文-->
> Hunt for opportunities to refactor with leading words. A triad spelled out at three sites, a pointer spending a sentence to gesture at one idea. Each is a passage begging to collapse into a single token:
> - "fast, deterministic, low-overhead" → _tight_ (a _tight_ loop).
> - "a loop you believe in" → _red_, turning a fuzzy gate into a binary observable state (the loop goes _red_ on the bug, or it doesn't).

<!--翻译-->
> 寻找用核心词重构的机会。三个地方拼出一个三元概念、一个指针花一整句话手势一个想法。每个段落都在恳求坍缩成单个 token：
> - "快、确定性、低开销" → *tight*（一个*紧凑*循环）。
> - "一个你信任的循环" → *red*（*变红*），将模糊的阈值转化为二值可观察状态（循环在 Bug 上*变红*，或不变红）。

<!--原文-->
> **Negation** is the failure mode beside this lever: steering by prohibition drags the forbidden behaviour into context and makes it _more_ available, not less. _Don't think of an elephant_, and the elephant is all there is; the negation is a weak modifier the strongly-activated concept overruns, so the ban half-reads as an instruction to do the thing. Prompt the **positive**: state the target behaviour ("write one-line comments") so the banned one is never spoken. A prohibition earns its place only as a hard guardrail you cannot phrase positively; even then, pair it with the positive target so attention lands on what to do.

<!--翻译-->
> **否定**是这个杠杆旁边的失败模式：通过禁止来引导会把被禁止的行为拖入上下文，使其*更容易*被获取，而非更难。*不要想大象*，大象就无处不在；否定只是被强激活概念碾压的弱修饰符，所以禁令半读作执行命令。提示**正向**行为：陈述目标行为（"写单行注释"），被禁止的行为根本不提。禁止仅在无法用正向措辞的硬护栏时才有存在意义；即便如此，也要配以正向目标，使注意力落在该做什么上。

<!--原文-->
> ## Pruning
> - Keep each meaning in a **single source of truth**: one authoritative place, so changing the behaviour is a one-place edit. **Duplication** (the same meaning in more than one place) costs maintenance and tokens, and inflates a meaning's prominence on the ladder past its real rank. (The accidental inverse of a leading word, which repeats a token on purpose, never the meaning.)
> - The **environment** is a source of truth too (`package.json` scripts, config files, the directory layout, `--help` output), and a document that restates it is a **cache**: a copy of a lookup, earning its load only when the lookup is expensive. Cache what the agent cannot find by looking: the unwritten convention, the reason behind a choice, the gotcha no config confesses. Leave the one-file, one-command lookups to the environment, where they cannot go stale.
> - Check every line for **relevance**: does it still bear on what the document does? A line loses relevance by never bearing on the task (mere exposition, or a branch that should be disclosed) or by going stale as the behaviour or world it describes changes. Shorter documents are easier to keep relevant. Without a pruning discipline the default fate is **sediment**: stale layers that settle because adding feels safe and removing feels risky, until you must core down through them to find what is still live.
> - Hunt **no-ops** sentence by sentence: an instruction the model already obeys by default pays load to say nothing. The test (does it change behaviour versus the default?) is model-relative, not reader-relative: two people disagreeing about a no-op disagree about the default, and settle it by running the document, not by debate. When a sentence fails, delete the whole sentence rather than trim words from it. The test also grades leading words: a word too weak to beat the default (_be thorough_ when the agent is already thorough-ish) is a no-op, and the fix is a stronger word (_relentless_), not a different technique.

<!--翻译-->
> ## 裁剪原则
> - 将每个含义保留在**单一真实来源**中：一个权威位置，改变行为只需一次编辑。**重复**（同一含义出现在多个位置）消耗维护和 token，并膨胀含义在层级上的显眼度至超过其真实排名。（核心词的有意对立面：有意重复 token，从不重复含义。）
> - **环境**也是一个真实来源（`package.json` 脚本、配置文件、目录布局、`--help` 输出），重述环境的文档是**缓存**：一次查找的拷贝，仅在查找昂贵时才值得付费。缓存 Agent 查找不到的内容：不成文的惯例、选择背后的原因、配置文件不会承认的陷阱。将单文件、单命令的查找留给环境，它们在那里不会过时。
> - 检查每一行的**相关性**：它仍然关系到文档的职责吗？一行通过从未与任务相关（纯粹的说明，或应被披露的分支）或随着描述的行为或世界变化而过时，来失去相关性。更短的文档更容易保持相关性。没有裁剪纪律，默认命运是**沉积**：过时图层因添加感到安全而删除感觉危险而堆积，直到你必须穿透它们找出仍然存活的内容。
> - 逐句搜寻**空操作**：模型默认已遵守的指令，负载白付了。测试（它是否改变与默认行为的差异？）是相对于模型而非读者的：两人关于空操作的分歧是关于默认行为的分歧，通过运行文档来解决，而非辩论。当一句失败，整句删除而非削减词数。此测试也评定核心词级别：一个太弱而无法击败默认行为的词（Agent 已经大致是 thorough，而你说 *be thorough*）是空操作，解决方式是更强的词（*relentless*），而非更换技术。
