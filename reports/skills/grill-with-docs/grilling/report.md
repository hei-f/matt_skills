# grilling -- 深度访谈原语

## 第一部分：详细总结

### 概述

`grilling` 是一个**模型调用（Model-invoked）**的 Skill，也是整个 grilling 家族（grill-me、grill-with-docs、triage、wayfinder、improve-codebase-architecture）背后的可复用访谈原语。它通过**设计树（Design Tree）**模型对计划、决策或想法进行无情的深度访谈：将主题建模为一棵决策树，每个决策又分支出依赖于它的子决策。访谈沿着树枝逐个推进，直到没有任何遗漏的假设为止。

### 核心概念：设计树、前沿与轮次

三个概念承载了整个 Skill：

1. **设计树（Design Tree）**：对主题的建模方式。每个决策（decision）都是一个节点，其依赖的前置决策被解决后，它才可以被提出。决策层层嵌套，形成一棵树。

2. **前沿（Frontier）**：所有前置决策已确定的决策的集合。只有这些决策才是当前可以诚实提出的问题。一个问题的答案如果还依赖另一个未回答的问题，它就不在当前前沿中。

3. **轮次（Round）**：一个前沿中的所有问题一次性全部提出并等待回答。两个问题如果互相依赖，绝不会出现在同一轮中。一次典型的 13 个问题的访谈通常只需约 3 轮，而非 13 轮逐题提问。

### 问题格式

每轮中的每个问题有固定格式：带编号、以 `❓` 开头的标题、正文、以及 Agent 在单独 `➡️` 行上给出的**推荐答案**，并要求把问题写成回答"是"即接受推荐答案的措辞。这种格式使得用户可以按编号回答整轮问题（如"1 是，2 选第二个选项，3 否，原因是..."），而不必逐题引用回原文。

需要注意的一个已知粗糙点：推荐答案有时会**反对**问题本身的措辞，这时同意推荐答案意味着对问题回答"否"。当这种情况发生时，直接回答推荐答案并说明即可。

### 事实与决策的分工

这是设计中的另一半核心：**事实（Facts）是 Agent 的工作**，当前沿问题需要环境能解决的信息时，Agent 会派发子 Agent（sub-agent）去查找，而不是问用户。正在进行的调研不会阻塞整轮：只有依赖于该项调研的下游问题会等待。**决策（Decisions）是用户的**，Agent 必须等待用户。如果一个运行 `grilling` 的 Agent 自己回答了自己的决策问题，那是一个 bug，而非自由发挥。当前沿为空时，会话结束，Agent 必须在用户确认双方达成共识后才开始行动。

### 前沿的诚实限制

前沿是 Agent 的判断，不是计算出的图结构。它可能将两个问题放在同一轮，之后才发现一个答案本应改变另一个问题。对此没有防护措施，唯一的补救是直接告诉 Agent，它会在下一轮重新打开受影响的分支。

### 与其他 Skill 的关系

本 Skill 文档涵盖机制本身（设计树、前沿、轮次、问题格式、事实 vs 决策）。具体的会话长度、如何处理无法通过对话解答的问题、如何避免盲目点头等问题在 grill-me 文档中解决。GLOSSARY.md 和 ADR 的写入在 grill-with-docs 文档中解决。

### 与 grill-me、grill-with-docs 的选择

| 场景 | 应使用的 Skill |
| --- | --- |
| 不在工作目录中 | grill-me |
| 在工作目录中 | grill-with-docs |
| 工作量太大，单次会话无法容纳 | wayfinder |
| 对话无法解决的问题（外观/体验） | prototype |
| 自己的 Skill 需要访谈能力 | 从你的 Skill 中调用 `/grilling` |

### 常见问题

- **能否回到逐题提问模式？** 可以。在全局 CLAUDE.md 中添加 `When grilling, ask one question at a time.` 即可。基于轮次的默认方式确实存在争议：阅读速度慢、非母语使用者、或将逐题格式作为专注支架的用户都报告逐题模式更适合他们，这个退出选项是被支持的。

- **batch-grill-me 去哪了？** 合并进了 grilling。基于轮次的提问曾短暂作为独立 Skill 存在，后来移入 grilling 本体，以便所有基于该原语的 Skill 都能同时获得该能力。通过 CLAUDE.md 中的配置行即可回到逐题模式。

- **一次性问整轮问题是否丢失了早期答案会引发的问题？** 这是对轮次设计最常见的质疑，而前沿就是回答：一轮只包含彼此不依赖的问题，因此轮内任何答案都不会使轮内其他问题失效。但答案确实会重塑下游：下一轮是重新计算的，而非预先写好的。

- **模型在问题问完后就开始构建了。** 确认门正是为此存在：前沿清空不意味着完成，用户确认共识达成才算完成。较弱或较快的模型仍然会突破这个限制。在 AGENTS.md 或 CLAUDE.md 中告诉 Agent 未经许可不能实施是可靠修复。

- **Agent 自己回答了自己的问题。** 这是运行中的 bug，不是预期行为，这也是为什么事实和决策在 Skill 文本中被明确分离。当另一个 Skill 在"解决此工单"框架内运行 grilling 时最容易出现。

- **能否限制问题数量？** 不能，这是有意为之。一些计划需要 3 个问题，一些需要 50 个。用自然语言引导：告诉它收尾，或停下来接受当前计划。如果会话过长，通常是范围太大，应拆解工作并逐个访谈。

- **只安装了 grill-me，但它不工作。** grill-me 是一个一行 Skill，其全部内容是"运行一次 `/grilling` 会话"，因此需要 grilling 也安装。选择性安装时需要同时安装原语。

### 判断是否正常工作的标志

- 轮次以编号列表形式到达，每个问题带单独的 `➡️` 推荐答案行，可以按编号回答整轮
- 轮次中没有问题需要同轮其他问题先回答
- 后轮提出的是首轮无法提出的问题
- Agent 主动查找事实（读取文件、派发子 Agent），而非问你它能自己找到的东西
- 后台运行的调研不阻塞整轮，只有依赖它的问题等待
- 结束时停下来请你确认共识，而非直接开始工作
- 问题总数高但轮次数低

---

## 第二部分：原文与翻译

<!--原文-->
> `grilling` is the interview loop that stress-tests a plan, a decision, or an idea before anyone acts on it. It maps the subject as a **design tree**: every decision branches into the decisions that hang off it, and interviews you branch by branch until no assumption stays unstated.

<!--翻译-->
> `grilling` 是一个访谈循环，在任何人行动之前对计划、决策或想法进行压力测试。它将主题映射为一棵**设计树**：每个决策分支出依赖于它的子决策，然后逐个分支进行访谈，直到没有任何假设未被言明。

<!--原文-->
> It does not ask one question at a time, and it does not ask everything at once. Each **round** asks the whole **frontier**: every decision whose prerequisites are already settled, and nothing else. Two questions never share a round if one depends on the other; a question that hinges on an answer still open belongs to a later round. Your answers settle decisions, the frontier moves outward, and the next round asks what that unblocked. Thirteen questions typically land in about three rounds rather than thirteen.

<!--翻译-->
> 它不会一次问一个问题，也不会一次性问完所有问题。每个**轮次**询问整个**前沿**：所有前置条件已确定的决策，仅此而已。如果两个问题相互依赖，它们绝不会共享一个轮次；一个答案还悬而未决的问题属于后续轮次。你的回答确定决策，前沿向外移动，下一轮询问被解锁的内容。13 个问题通常只需约 3 轮，而非 13 轮。

<!--原文-->
> The skill rests on three ideas. The **design tree** is the model of the subject: decisions with decisions hanging off them. The **frontier** is the set of decisions whose prerequisites are all settled: the only questions the agent can ask yet. A **round** is one frontier, asked in full and answered in full.

<!--翻译-->
> 这个 Skill 建立在三个概念之上。**设计树**是对主题的建模：层层嵌套的决策。**前沿**是所有前置条件已确定的决策集合：当前 Agent 唯一可以提出的问题。一个**轮次**就是一个前沿，完整提出并完整回答。

<!--原文-->
> Inside a round, every question has a fixed format: numbered and titled behind a `❓`, then the body, then the agent's recommended answer alone on a `➡️` line. This format lets you answer a round by number ("1 yes, 2 the second option, 3 no, here's why") instead of by quoting questions back.

<!--翻译-->
> 在一轮之内，每个问题都有固定格式：带编号、以 `❓` 开头的标题、正文，以及 Agent 的推荐答案单独放在 `➡️` 行上。这种格式让你可以通过编号回答整轮（"1 是，2 第二个选项，3 否，原因是..."），而不必逐题引用回原文。

<!--原文-->
> The other half of the design is the split between facts and decisions. Facts are the skill's own job. When a frontier question needs something the environment can settle, the agent dispatches a sub-agent to find out rather than asking you. The round does not wait for that research; only the questions that depend on it wait. Decisions are yours, and the agent must wait for them. An agent running `grilling` that answers its own decisions has broken the skill. The session ends when the frontier is empty. The agent then waits for you to confirm a shared understanding before it acts on what you agreed.

<!--翻译-->
> 设计的另一半是事实与决策的分工。**事实**是 Skill 自己的工作。当前沿问题需要环境能解决的信息时，Agent 会派发子 Agent 去查明，而不是问你。整轮不会等待这项调研；只有依赖它的问题才会等待。**决策**是你的，Agent 必须等待。一个运行 `grilling` 的 Agent 如果自己回答了自己的决策，那就是破坏了 Skill。当前沿为空时，会话结束。之后 Agent 会等待你确认双方达成共享理解，然后才对你同意的事项采取行动。

<!--原文-->
> The frontier has one limit. The agent chooses it by judgement; it does not compute it from a graph. So it can put two questions in one round and only later find that one answer should have changed the other. The only guard is to tell it. That reopens the affected branch in the next round.

<!--翻译-->
> 前沿有一个限制。它是 Agent 凭判断选出的，而不是从图中计算出来的。所以它可能把两个问题放进同一轮，之后才发现一个答案本应改变另一个。唯一的防护就是直接告诉它。它会在下一轮重新打开受影响的分支。

<!--原文-->
> Not everyone prefers the round-based default. Users who read slowly, who work in a second language, or who use the sequential format to help them focus report that one question at a time works better for them. The opt-out is a supported setting, not a workaround.

<!--翻译-->
> 并非所有人都偏好基于轮次的默认方式。阅读速度慢的用户、用第二语言工作的用户、或借助逐题格式来帮助自己专注的用户都报告说，一次问一个问题对他们效果更好。这个退出选项是一项受支持的设置，而不是变通办法。

<!--原文-->
> Asking a whole round at once must lose the questions my earlier answers would have raised. Doesn't it? This is the most common objection to rounds, and the frontier answers it. A round only contains questions that do not depend on each other, so no answer in a round can invalidate another question in that round. Your answers still change what comes after them, because the agent works out each next round from scratch.

<!--翻译-->
> 一次性问整轮问题一定会丢失我早期答案本会引发的问题。不是吗？这是对轮次最常见的质疑，而前沿就是它的答案。一轮只包含彼此不依赖的问题，因此轮内任何答案都不会使轮内其他问题失效。你的答案仍然会改变它们之后的内容，因为 Agent 会从头推算出每一轮。

<!--原文-->
> The confirmation gate exists for this. The skill is finished when you say the understanding is shared, not when the frontier empties. Weaker and faster models still skip the gate. Users report this most often on lower-effort or non-frontier models, which collapse "interview until shared understanding" into a couple of questions and an outline.

<!--翻译-->
> 确认门正是为此而存在。当你说理解已共享时，Skill 才算完成，而不是当前沿清空时。较弱和较快的模型仍会跳过这道门。用户最常在低努力度或非前沿模型上报告这一点，它们将"访谈到共享理解为止"坍缩为几个问题和一份大纲。

<!--原文-->
> It answered its own questions instead of asking me. That is a bug in the run, and it is the reason the skill's text separates facts from decisions.

<!--翻译-->
> 它自己回答了自己的问题，而不是问我。这是运行中的一个 bug，这也是 Skill 文本把事实与决策分开的原因。

<!--原文-->
> Can I cap the number of questions? No, and a cap is out of scope on purpose. Some plans need three questions and some need fifty; a fixed ceiling either truncates the hard case or feels arbitrary on the easy one. The intended control is plain language. Tell it to wrap up, or stop and accept the plan where it stands.

<!--翻译-->
> 能否限制问题数量？不能，上限是有意排除在范围之外的。一些计划需要 3 个问题，一些需要 50 个；固定上限要么截断困难情况，要么在简单情况下显得武断。预期的控制方式是用自然语言。告诉它收尾，或停下来接受当前计划。

<!--原文-->
> `grilling` is a **primitive**, not a step you schedule: the single source of truth for the interview technique, kept in one place so every skill that needs an interview reaches for it instead of inventing one.

<!--翻译-->
> `grilling` 是一个**原语**，不是你安排的一个步骤：访谈技术的单一真相来源，集中在一处，以便每个需要访谈的 Skill 都能调用它，而非各自发明一套。
