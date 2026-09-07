# to-questionnaire -- 决策转问卷

## 第一部分：详细总结

### 概述

`to-questionnaire` 是一个**用户调用（User-invoked）**的 Skill，将你无法独自解决的决策转化为一份**问卷**：一份 Markdown 文档，你交给唯一拥有你所缺失信息的那个人，让他们异步填写，或你们两人在会议中一起完成。

它追问的是**发送方（the Send）**，而非主题本身。在这里访谈你关于主题是没有意义的：不了解主题正是你要写给别人的原因。因此它只问两个你总能回答的问题（这份问卷给谁、你需要从他们那里得到什么），并将文档中的每一个问题瞄准两者之间的**差距（gap）**。

### 调用方式与触发场景

通过输入 `/to-questionnaire` 调用，Agent 不会自己触发。当决策被阻塞在另一个人脑中的知识上时使用：客户、领域专家、拥有业务规则的高管、你不在一起坐的团队的同事。

| 答案在... | 应使用 |
| --- | --- |
| 你自己脑中，尚未锐化 | grill-me |
| 代码库中 | grill-with-docs |
| 别人脑中 | to-questionnaire |
| 还没在任何人的脑中，问题需要有一个东西来回应 | prototype |

常见情况是一个 grilling 会话停滞了：浮现出的一些内容不是你来回答的。在同一对话中运行 `/to-questionnaire` 将这些离线化，然后带回答案继续。

### 核心原则：追问发送方，而非主题

访谈只有两次交流，然后就停止：

1. **这份问卷给谁？** 他们的角色、专长、与你的关系。这决定语气和文档需要承载多少上下文：外部客户需要被引导，队友不需要。
2. **你需要从他们那里得到什么？** 你无法独自解决的具体决策或事实。这成为衡量完成文档的清单：你命名的每一项都对应一个瞄准它的问题。

之后的一切都是起草。文件在当前目录生成 `to-questionnaire-<slug>.md`。没有设置、没有工作空间、无需配置。

### 文档结构

文档被框定为一份**发现问卷（Discovery Questionnaire）**（你缺乏上下文，接收方持有它），该框定驱动其形态：

- **Purpose 行**：命名取决于它的决策
- **From/To**：明确的发送方和接收方
- **Context 部分**：给从未在你脑中待过的接收方的简短背景
- **问题按最重要优先排序**，并在主题化标题下分组——因为异步意味着你可能只能得到一次回复
- **每个问题一个想法**，绝不复合，下面有答案占位符，仅在问题可能被误读时附带"为什么这很重要"行
- **明确允许回答"我不知道"**：标记出的不确定性是有用的；读起来像事实的自信猜测不是
- **结束时的通用问题**：有什么我们没问到但我们应该知道的事吗？

文档刻意不是两样东西：不是**分支**（问题是扁平分组的列表，不是回答了 A 就跳过 D 节的树）；也不是**多接收方**（一次运行产生一份文档给一个人）。

### 常见问题

- **它会读取我的 grilling 会话并从中提取问题吗？** 不作为自己的步骤。该 Skill 没有摄入阶段：它问关于发送方的问题，然后起草。使它在 grilling 会话后有效的关键是在**同一对话**中运行它，这样会话已在上下文中，起草可以借助它。在新会话中启动它对 grilling 一无所知。

- **缺失的答案不都在同一个人那里。能按接收方拆分吗？** 不能。第一步询问*那个*接收方，单数，整份文档的语气和上下文都是针对他们的。如果三个人持有答案的三部分，运行三次，每人一次。在一个文档中按学科或角色路由问题是人们提过的需求，但不是发布的功能。

- **问题有依赖关系吗：会根据早期答案跳过部分吗？** 没有。依赖问题设计曾被探索但未发布。输出是静态文档：主题分组、最重要优先、每个问题都是活的。反对它的理由是公允的：一个模型在超过两三个问题的真实答案之前就计划，效果很差，而分支文档必须在每个答案之前计划所有问题。

- **如果接收方也不知道呢？** 文档告诉他们这样说。"我不知道"和部分答案是明确被要求的，标记出的不确定性比猜测更有价值，因为模糊的答案和自信但错误的答案一旦回到你的上下文中就看起来一样。

- **它会发送到某处吗（Slack、Issue 追踪器、邮件）？** 不会。它在当前目录写入一个 Markdown 文件并告诉你路径。交付是你的：粘贴到工单中、放到 Slack 线程中、附在邮件中、或在共享屏幕上打开并实时完成。人们手工连接了所有四种方式。

- **这不就是批处理模式的 `/grill-me` 吗？** 不是，这个区分值得保持。`grill-me` 已经以**轮次**提问：整个前沿一次提出，然后根据你的答案重新计算，因此"一次给我所有问题"的需求在那里已满足。`to-questionnaire` 关乎不同的轴：不是问题如何交付，而是答案在谁的脑中。自己更快地回答它们是 `grill-me`；从别人那里获取答案是它。

- **我就不能直接让 Agent 做这个而不用 Skill 吗？** 可以，很多人之前就是这样做的。该 Skill 为你买了两个东西：访谈绝不会偏离到主题上；文档以非技术接收方实际能填写的形态产出。如果你已有行之有效的自有格式，诚实的回答是你不需要这个。

### 判断是否正常工作的标志

- 它问关于接收方和你需要什么回来的问题，然后停止。关于主题本身的问题是 Skill 脱轨了
- 你命名的每一个"我需要回来的东西"都可以追溯到文件中的一个问题
- 问题读起来像是瞄准接收方*知道的东西*，而非你自己的待解答问题逐字复制
- 你能将文件交给不在对话中的人，他们也会知道为什么收到了它以及何时回复
- 回来的答案可作为新 grilling 轮次的可使用输入，而非一组新问题

### 在 Skill 体系中的位置

`to-questionnaire` 是一个随时可用的独立 Skill。它位于你自身知识的边界，下一步是另一个人而非另一个 Skill，最常见的是流程中——当规划停滞于不属于你决定的内容时。它的邻居是 grill-me，两者的分界线在答案居住的地方：grilling 挖掘你，问卷挖掘别人。回来的内容是原材料：将其喂入另一个 grilling 轮次，或如果工作走向构建，喂入 grill-with-docs 或 to-spec。

---

## 第二部分：原文与翻译

<!--原文-->
> `to-questionnaire` turns a decision you can't settle on your own into a **questionnaire**: a Markdown document you hand to the one person who holds what you're missing, for them to fill in async or for the two of you to work through in a meeting.

<!--翻译-->
> `to-questionnaire` 将你无法独自解决的决策转化为一份**问卷**：一份 Markdown 文档，你交给唯一拥有你所缺失信息的那个人，让他们异步填写，或你们两人在会议中一起完成。

<!--原文-->
> It grills you about the **send**, never the subject. Interviewing you about the topic is pointless here: not knowing the topic is why you're writing to someone else. So it asks the two things you can always answer (who this is going to, and what you need back from them) and aims every question in the document at the **gap** between the two.

<!--翻译-->
> 它追问的是**发送方**，而非主题本身。在这里访谈你关于主题是没有意义的：不了解主题正是你要写给别人的原因。因此它只问两个你总能回答的问题（这份问卷给谁、你需要从他们那里得到什么），并将文档中的每一个问题瞄准两者之间的**差距**。

<!--原文-->
> The interview is two exchanges, and then it stops. **Who is it going to?** Their role, their expertise, their relationship to you. This fixes the tone and how much context the document has to carry: an outside client needs orienting, a teammate does not. **What do you need back?** The concrete decisions or facts you can't resolve alone. This becomes the checklist the finished document is measured against: every item you named gets a question aimed at it.

<!--翻译-->
> 访谈只有两次交流，然后就停止。**这份问卷给谁？** 他们的角色、专长、与你的关系。这决定语气和文档需要承载多少上下文：外部客户需要被引导，队友不需要。**你需要从他们那里得到什么？** 你无法独自解决的具体决策或事实。这成为衡量完成文档的清单：你命名的每一项都对应一个瞄准它的问题。

<!--原文-->
> It is framed as a **discovery questionnaire** (you lack the context, the recipient holds it), and that framing drives its shape: A purpose line naming the decision riding on it, and a short context section for a recipient who was never in your head. Questions ordered **most-important-first** and grouped under themed headings, because async means you may only get one pass. One idea per question, never compound, with an answer stub beneath it and a *why this matters* line only where a question could be misread. Explicit permission to answer "I don't know": a flagged uncertainty is useful; a confident guess that reads like a fact is not. A closing catch-all: anything we didn't ask that we should know?

<!--翻译-->
> 它被框定为一份**发现问卷**（你缺乏上下文，接收方持有它），该框定驱动其形态：Purpose 行命名取决于它的决策；给从未在你脑中待过的接收方的简短 Context 部分。问题按**最重要优先**排序并在主题化标题下分组——因为异步意味着你可能只能得到一次回复。每个问题一个想法，绝不复合，下面有答案占位符，仅在问题可能被误读时附带*为什么这很重要*行。明确允许回答"我不知道"：标记出的不确定性是有用的；读起来像事实的自信猜测不是。结束时的通用问题：有什么我们没问到但我们应该知道的事吗？

<!--原文-->
> Two things it deliberately isn't. It isn't **branching**: the questions are a flat, grouped list, not a tree that skips section D if you answered A. And it isn't **multi-recipient**: one run produces one document for one person.

<!--翻译-->
> 它刻意不是两样东西。不是**分支**：问题是扁平分组的列表，不是回答了 A 就跳过 D 节的树。也不是**多接收方**：一次运行产生一份文档给一个人。

<!--原文-->
> Does it read my grilling session and extract the questions from it? Not as a step of its own. The skill has no ingest phase: it asks about the send, then drafts. What makes it work after a grilling session is that you run it in the **same conversation**, so the session is already in context and the drafting can draw on it. Start it in a fresh session and it knows nothing about the grilling.

<!--翻译-->
> 它会读取我的 grilling 会话并从中提取问题吗？不作为自己的步骤。该 Skill 没有摄入阶段：它问关于发送方的问题，然后起草。使它在 grilling 会话后有效的关键是在**同一对话**中运行它，这样会话已在上下文中，起草可以借助它。在新会话中启动它对 grilling 一无所知。

<!--原文-->
> The missing answers don't all live with the same person. Can it split them by recipient? No. Step one asks for *the* recipient, singular, and the tone and context of the whole document are pitched at them. If three people hold three parts of the answer, run it three times, once per person.

<!--翻译-->
> 缺失的答案不都在同一个人那里。能按接收方拆分吗？不能。第一步询问*那个*接收方，单数，整份文档的语气和上下文都是针对他们的。如果三个人持有答案的三部分，运行三次，每人一次。

<!--原文-->
> Are the questions dependent: does it skip sections based on earlier answers? No. The dependent-question design was explored and did not ship. The output is a static document: themed groups, most-important-first, every question live. The objection against it is a fair one: a model planning more than two or three questions ahead of a real answer plans badly, and a branching document has to plan all of them ahead of every answer.

<!--翻译-->
> 问题有依赖关系吗：会根据早期答案跳过部分吗？没有。依赖问题设计曾被探索但未发布。输出是静态文档：主题分组、最重要优先、每个问题都是活的。反对它的理由是公允的：一个模型在超过两三个问题的真实答案之前就计划，效果很差，而分支文档必须在每个答案之前计划所有问题。

<!--原文-->
> What if the recipient doesn't know either? The document tells them to say so. "I don't know" and partial answers are asked for explicitly, and a flagged uncertainty is worth more than a guess, because a vague answer and a confidently wrong one look identical once they're back in your context.

<!--翻译-->
> 如果接收方也不知道呢？文档告诉他们这样说。"我不知道"和部分答案是明确被要求的，标记出的不确定性比猜测更有价值，因为模糊的答案和自信但错误的答案一旦回到你的上下文中就看起来一样。

<!--原文-->
> Isn't this just `/grill-me` in batch mode? No, and the distinction is worth holding. `grill-me` already asks in **rounds**: the whole frontier at once, then recomputed from your answers, so the "give me all the questions at once" need is met there. `to-questionnaire` is about a different axis: not how the questions are delivered, but whose head the answers are in. Answering them yourself faster is `grill-me`; getting them out of someone else is this.

<!--翻译-->
> 这不就是批处理模式的 `/grill-me` 吗？不是，这个区分值得保持。`grill-me` 已经以**轮次**提问：整个前沿一次提出，然后根据你的答案重新计算，因此"一次给我所有问题"的需求在那里已满足。`to-questionnaire` 关乎不同的轴：不是问题如何交付，而是答案在谁的脑中。自己更快地回答它们是 `grill-me`；从别人那里获取答案是它。

<!--原文-->
> `to-questionnaire` is a reach-for-it-anytime standalone. It sits at the boundary of your own knowledge, where the next move is another person rather than another skill, most often mid-flow, when planning has stalled on something that isn't yours to decide. Its neighbour is grill-me, and the two split on where the answers live: grilling mines you, a questionnaire mines someone else. What comes back is raw material: feed it into another grilling round, or into grill-with-docs or to-spec if the work is heading for a build.

<!--翻译-->
> `to-questionnaire` 是一个随时可用的独立 Skill。它位于你自身知识的边界，下一步是另一个人而非另一个 Skill，最常见的是流程中——当规划停滞于不属于你决定的内容时。它的邻居是 grill-me，两者的分界线在答案居住的地方：grilling 挖掘你，问卷挖掘别人。回来的内容是原材料：将其喂入另一个 grilling 轮次，或如果工作走向构建，喂入 grill-with-docs 或 to-spec。
