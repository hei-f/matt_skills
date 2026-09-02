# handoff -- 会话交接

## 第一部分：详细总结

### 概述

`handoff` 是一个**用户调用（User-invoked）**的 Skill，将当前对话压缩为一份**交接文档**（一个 Markdown 文件），写入操作系统的临时目录（而非工作区），使一个新的 Agent 可以读取它并继续工作。

它提供的是**可移植性（portability）**，而非压缩。这个 Skill 比听起来更狭窄：仅当工作需要"旅行"到新的 harness、新目录、同事或分支任务时才需要文件。如果没有任何东西需要旅行，你并不需要 handoff：留在当前会话中、`/clear`、子 Agent 和 `/compact` 覆盖了普通的阶段结束场景。

### 调用方式

通过输入 `/handoff` 调用，Agent 不会自己触发。传入一条关于下一个会话目的的备注，文档会围绕该目的编写。

### 四个触发场景

| 场景 | 为什么需要文件 |
| --- | --- |
| 切换 Harness（Claude → Codex） | 新 harness 无法看到旧上下文 |
| 移动到不同目录或仓库 | 原型目录是常见情况 |
| 将工作发给同事 | 他们需要可读的东西 |
| 分支一个阶段中发现的侧任务 | 你继续工作，第二个 Agent 接手分支 |

对于其他情况（同一 harness、同一目录、已完成 grilling 进入实现），`/compact` 是正确的选择。

### 分支：最被忽略的用例

Skill 的描述读起来像会话恢复：写摘要、在这里结束、在那里恢复。这样读它看起来像一个更差的 `/compact`，所以被略过。**分支（Fork）**场景是最值得知道的：你**留在当前会话中**，同时将累积上下文的副本交给另一个并行工作的 Agent。这正是 prototype 的迂回路线所使用的：你在设计对话中深入时，遇到一个只有运行代码才能解答的问题——把上下文交给一个 prototype 会话，拿到答案，交回答案，从原始线程引用它。两次穿越，一个活对话，无需重新解释。

### 阶段边界的三种保留方式

在阶段边界，五个选项中的三个保留不同的事物：`/compact` 保留你的意图，`/clear` 不保留任何东西，`/handoff` 保留工作的移动能力。

### 文档内容：什么传递，什么不传递

文档携带**活跃线程**（正在进行什么、为什么、接下来是什么）以及一个**建议 Skills** 部分，命名下一个 Agent 应该使用的 Skills。机密信息在写入前被**脱敏**。

刻意**不携带**任何已经记录下来的内容。Spec、计划、ADR、Issue、提交和 diff 通过路径或 URL 引用，绝不复制。这保持文件小巧，并将已确定的细节保留在一个位置而非两个可能偏离的位置。

### 常见问题

- **Handoff 还是 compact？** 除非有东西需要旅行，否则用 `/compact`。留在同一任务上是 compact，不是 handoff。`/handoff` 的优势不在于总结得更好，而在于结果是一个可以携带到 `/compact` 无法到达的地方的文件。

- **Compact、clear 和 handoff 的实际区别？** 保留的是三种不同的东西。`/compact` 压缩上下文，让你在新窗口中继续：意图存活。`/clear` 清空窗口从头开始：适用于身后一切可丢弃时。`/handoff` 写出可移植文件：工作存续于移动到别处。注意三者都将**一手素材**（实际发生的对话）转化为**二手素材**（对它的摘要）。继续是唯一不这样做的操作。

- **交接文件去哪了？** 临时目录。这是该 Skill 最常报告的摩擦：路径很长、因 OS 而异、Windows 上 Agent 有时需要多次尝试才能找到正确的。在继续前索要路径并保留它。放在 temp 是故意的：交接文档是转运文档，不是需要维护的产物，也不是持久的。

- **文件在会话之间消失了。** 某些环境在会话间清空 temp（Codex 是报告的情况），且 `/private/tmp` 在重启时消失。如果下一个会话不会在一小时内开始，或在不同 harness 下开始，自己尽快将文件复制到持久位置。

- **实际上怎么交给下一个 Agent？** 打开新会话并指向该路径：读取此文件，然后继续。指向文件而不是将摘要粘贴到 shell 命令中：包含反引号或 `$(...)` 的摘要在被插入 `claude "<summary>"` 时会损坏，通常的失败是静默截断而非报错。

- **这和 `/branch`、`--fork-session` 或内置的 `/handoff` 一样吗？** 类似但不相同。Fork 继承上下文的精确副本；这个 Skill 生成针对指定下一个任务的**定向压缩**，以文件形式存在。Fork 能做的时候（同机器、同 harness、同目录），fork 更省力。文件在目标超出 fork 可达范围时胜出。

- **什么情况下应该放入 CLAUDE.md 而非 handoff？** 问自己它下个月是不是还成立。CLAUDE.md 是关于项目的常设上下文，每次会话都加载。Handoff 是关于一项进行中的工作，该工作一旦落地就失效了。

- **它捕获了"什么"而非"为什么"。** 这是一个公允但反复出现的批评。两个帮助：传入参数（告诉它下一个会话的目的），以便相关推理被保留。并注意会话从未实际验证的自信断言："X 还没建"、"Y 已经完成了"。下一个 Agent 将文档视为契约，不会重新检查，因此一个写为事实的信念会成为后续一切的错误前提。

- **为什么是个 Skill 而非斜杠命令？** 两者都可行，适合不同场景。作为 Skill，它通过与其他所有内容相同的安装路径发布和更新，这使其可分享；Agent 不会自己触发它的约束由 frontmatter 设置。

### 判断是否正常工作的标志

- 文档是对话的一小部分，Spec、Issue 和 diff 以路径和 URL 而非复制文本出现
- 你可以在没有原始会话的情况下冷读它，并知道接下来该做什么
- 新 Agent 开始工作而非要求你重新解释设置
- 在分支情况下，你回来后原始会话仍然原封未动
- 建议 Skills 部分列出了你自己会选择的 Skill
- 其中没有任何密钥、令牌或密码

### 在 Skill 体系中的位置

`handoff` 是一个**随时可用的独立 Skill**，位于会话之间的接缝处，而非构建链中。它是一个狭窄的工具，诚实的地图是你在阶段边界的五个选项中会较少使用它。它最近的邻居是 prototype，因为原型存在于自己的目录中，往返出去再回来正是这个 Skill 所服务的穿越。

---

## 第二部分：原文与翻译

<!--原文-->
> `handoff` compacts the conversation you are in into a **handoff document**: one markdown file, written to your OS's temporary directory rather than into the workspace, that a fresh agent can read to pick the work up.

<!--翻译-->
> `handoff` 将你所在的对话压缩为一份**交接文档**：一个 Markdown 文件，写入操作系统的临时目录而非工作区，使一个新的 Agent 可以读取它并继续工作。

<!--原文-->
> What it buys is **portability**, not compression. That makes the skill narrower than it sounds. You need a file only when the work has to *travel*: to a new harness, a new directory, a colleague, or a side task you want to fork off. If nothing is travelling, you do not need a handoff: staying in the session, `/clear`, a subagent and `/compact` cover the ordinary end-of-phase case, and `/compact` covers it more often than this skill does.

<!--翻译-->
> 它提供的是**可移植性**，而非压缩。这使得该 Skill 比听起来更狭窄。仅当工作需要*旅行*时才需要文件：到新的 harness、新目录、同事、或你想分支出去的侧任务。如果没有任何东西在旅行，你不需要 handoff：留在会话中、`/clear`、子 Agent 和 `/compact` 覆盖了普通的阶段结束场景，且 `/compact` 比这个 Skill 更常用。

<!--原文-->
> Branching is the use people skip. You **stay in your session** and hand a copy of the accumulated context to a second agent working in parallel.

<!--翻译-->
> 分支是最被忽略的用例。你**留在当前会话中**，将累积上下文的副本交给另一个并行工作的 Agent。

<!--原文-->
> That is what the detour through prototype uses. You are deep in a design conversation, you hit a question that only running code will settle, and you do not want to spend the thread you built on finding out. Hand off to a prototype session, get the answer, hand the answer back, and reference it from the original thread. Two crossings, one live conversation, nothing re-explained.

<!--翻译-->
> 这正是 prototype 的迂回路线所使用的。你在设计对话中深入时，遇到一个只有运行代码才能解答的问题，而你不愿花掉已建立的线程来弄清楚。交接给一个 prototype 会话，拿到答案，交回答案，从原始线程引用它。两次穿越，一个活对话，无需重新解释。

<!--原文-->
> Three of the five options at a phase boundary preserve different things: `/compact` preserves your intent, `/clear` preserves nothing, `/handoff` preserves the work's ability to move.

<!--翻译-->
> 在阶段边界，五个选项中的三个保留不同的事物：`/compact` 保留你的意图，`/clear` 不保留任何东西，`/handoff` 保留工作的移动能力。

<!--原文-->
> The document carries the live thread (what's in flight, why, and what's next) plus a **suggested skills** section naming what the next agent should reach for. Secrets are redacted before it's written.

<!--翻译-->
> 文档携带活跃线程（正在进行什么、为什么、接下来是什么）以及一个**建议 Skills** 部分，命名下一个 Agent 应该使用的 Skills。机密信息在写入前被脱敏。

<!--原文-->
> What it deliberately does not carry is anything already written down. Specs, plans, ADRs, issues, commits and diffs are referenced by path or URL, never copied. That keeps the file small, and it keeps the settled detail in one place instead of two that drift.

<!--翻译-->
> 刻意不携带的是任何已经记录下来的内容。Spec、计划、ADR、Issue、提交和 diff 通过路径或 URL 引用，绝不复制。这保持文件小巧，并将已确定的细节保留在一个位置而非两个可能偏离的位置。

<!--原文-->
> Handoff or compact? `/compact` unless something is travelling. Staying on the same task is a compact, not a handoff: same harness, same directory, and you need to stay in the loop is where the phase-boundary tree lands most days. `/handoff`'s advantage is not that it summarises better; it's that the result is a file you can carry somewhere `/compact` can't reach.

<!--翻译-->
> Handoff 还是 compact？除非有东西在旅行，否则用 `/compact`。留在同一任务上是 compact，不是 handoff：同 harness、同目录、且你需要保持在循环中——这就是阶段边界决策树大多数时候的落脚点。`/handoff` 的优势不在于总结得更好，而在于结果是一个你可以携带到 `/compact` 无法到达的地方的文件。

<!--原文-->
> Three different things being preserved. `/compact` compresses this context and keeps you going in a fresh window: intent survives. `/clear` empties the window and starts from nothing: correct when everything behind you is disposable, and one-way if it isn't. `/handoff` writes a portable file: the work survives the move to somewhere else. Note that all three turn a **primary source** (the conversation as it happened) into a **secondary source** (a summary of it). Continuing is the only move that doesn't, which is why it's the first one to rule out.

<!--翻译-->
> 保留的是三种不同事物。`/compact` 压缩此上下文，让你在新窗口中继续：意图存活。`/clear` 清空窗口从头开始：适用于身后一切可丢弃时，如果不可丢弃则是单向的。`/handoff` 写出可移植文件：工作存续于移动到别处。注意三者都将**一手素材**（实际发生的对话）转化为**二手素材**（对它的摘要）。继续是唯一不这样做的操作，这就是为什么它是第一个要考虑排除的选项。

<!--原文-->
> Where did my handoff file go? The temp directory, which is the most-reported friction with the skill: the paths are long, they differ per OS, and on Windows agents sometimes take several attempts to find the right one. Ask for the path back and keep it before you move on. Temp is deliberate: a handoff is a transit document, not an artifact you maintain.

<!--翻译-->
> 我的交接文件去哪了？临时目录——这是该 Skill 最常报告的摩擦：路径很长、因 OS 而异、Windows 上 Agent 有时需要多次尝试才能找到正确的。在继续前索要路径并保留它。放在 temp 是故意的：交接文档是转运文档，不是你维护的产物。

<!--原文-->
> Is this the same as `/branch`, `--fork-session`, or the built-in `/handoff`? Analogous, not identical. A fork inherits an exact copy of the context; this skill produces a *targeted* compression aimed at a stated next task, in a file. Where a fork will do (same machine, same harness, same directory), a fork is less work. The file wins the moment the destination is somewhere the fork can't go.

<!--翻译-->
> 这和 `/branch`、`--fork-session` 或内置的 `/handoff` 一样吗？类似但不相同。Fork 继承上下文的精确副本；这个 Skill 生成针对指定下一个任务的*定向*压缩，以文件形式存在。Fork 能做的时候（同机器、同 harness、同目录），fork 更省力。文件在目标超出 fork 可达范围时胜出。

<!--原文-->
> When does something belong in `CLAUDE.md` instead? Ask whether it's true next month. `CLAUDE.md` is standing context about the project, loaded into every session whether it's relevant or not. A handoff is about one piece of work in flight and is dead once that work lands. Facts that keep getting re-explained are a `CLAUDE.md` problem; a half-finished task is a handoff.

<!--翻译-->
> 什么情况下应该放入 `CLAUDE.md` 而非 handoff？问自己它下个月是不是还成立。`CLAUDE.md` 是关于项目的常设上下文，每次会话都加载，无论是否相关。Handoff 是关于一项进行中的工作，该工作一旦落地就失效了。反复需要重新解释的事实是 `CLAUDE.md` 的问题；一个未完成的任务是 handoff 的问题。

<!--原文-->
> It captures the what, not the why. A fair and repeated criticism. Two things help. Pass the argument (tell it what the next session is for) so the reasoning that bears on *that* is kept rather than flattened. And watch for confident claims the session never actually verified: "X isn't built", "Y is done". The next agent treats the document as a contract and will not re-check it, so a belief written as a fact becomes a false premise for everything that follows.

<!--翻译-->
> 它捕获了"什么"而非"为什么"。一个公允但反复出现的批评。两个帮助：传入参数（告诉它下一个会话的目的），以便与之相关的推理被保留而非被扁平化。并注意会话从未实际验证的自信断言："X 还没建"、"Y 已经完成了"。下一个 Agent 将文档视为契约，不会重新检查，因此一个写为事实的信念会成为后续一切的错误前提。
