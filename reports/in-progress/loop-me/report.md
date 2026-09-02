# loop-me 技能报告

## 第一部分：中文详细总结

### 概述

loop-me 是一个用户手动调用的技能，用于运行有状态的"盘问"（grilling）会话，其唯一输出是**工作流规格说明**（workflow specs）。它采用"盘问"原则——一轮接一轮地提问，每轮附带推荐答案——围绕特定的词汇表和目标进行深入挖掘。在盘问过程中，随着问题逐渐解决，可以创建、编辑和删除规格文件。

### 核心概念：循环透镜（The Loop Lens）

**循环**（loop）是用户生活中反复出现的模式：职业生涯、一周、一个早晨、一个重复的活动。将生活想象成环环相扣的循环，可以揭示其活动的可预测性，而这正是它们值得**委托**（delegate）的原因。循环透镜的用途是发现值得规格化的循环，并向用户提出他们尚未注意到的循环。

**工作流**（workflow）是某个循环的规格说明，使其成为现实。你在一个循环上运行工作流：循环是其运行中的实例化。工作流存放在 `workflows/*.md` 中，是唯一的事实来源。

### 词汇表（Vocabulary）

共享语言仅在具体工作流需要时才使用，绝不是清单。不强加任何结构性要求：工作流不一定需要 AI、检查点或时间表，除非盘问显示需要。

- **触发器（Trigger）**：触发每次运行的条件，可以是**事件**（新邮件、新 Issue）或**时间表**（每天早晨）。事件触发通常更高效。
- **检查点（Checkpoint）**：人机交互的验证点，用户被要求核实或决策。某些工作流没有检查点，自主运行；某些完全不使用 AI。
- **向右推（Push Right）**：将检查点尽可能推迟。在涉及人类之前完成最多的工作，这样用户只需被询问一次，且时点靠后，前面一切已准备就绪。
- **简报（Brief）**：检查点呈现的内容，是一份紧凑、可决策的摘要（产出了什么、为什么、以及指向产物的链接），绝不是原始输出。用户读的是简报而非草稿。审阅速度至关重要。

### 完工定义（Definition of Done）

工作流规格完成的标志是：一个实施者智能体（implementer agent）可以在不提出任何问题的情况下构建它。在此之前持续盘问；只要还有问题存在，就还没有完成。

### 工作区结构（The Workspace）

- `workflows/*.md`：每个工作流一个规格文件。
- `NOTES.md`：关于用户世界的原始笔记，包括他们使用的工具、处理的信息渠道、以及他们自己对这两者的术语。当它为空或很薄时，在指定任何内容之前，先就他们的世界进行访谈。随着模糊术语在对话中浮现，将其细化为规范术语，并记录在这里。

---

## 第二部分：原文与翻译对照

<!--原文-->
> Run a stateful `/grilling` session whose only output is **workflow** specs. Use the grilling discipline (relentless, a round of questions at a time, a recommended answer attached to each) aimed at the vocabulary and goal below. Create, edit, and delete specs as the grilling resolves things.
<!--翻译-->
> 运行一个有状态的 `/grilling` 会话，其唯一输出是**工作流**规格说明。采用盘问原则（不松懈，每次一轮问题，每轮附带推荐答案），针对下列词汇表和目标进行。在盘问过程中，随着问题解决，创建、编辑和删除规格文件。

<!--原文-->
> A **loop** is a recurring pattern in the user's life: their career, their week, their morning, a single repeated activity. Picturing a life as loops within loops reveals how predictable its activities really are, which is what makes them worth **delegating**. Use the lens to find loops worth specifying, and propose ones the user hasn't noticed.
<!--翻译-->
> **循环**是用户生活中反复出现的模式：职业生涯、一周、一个早晨、一个重复的活动。将生活想象成环环相扣的循环，可以揭示其活动实际上有多么可预测，而这正是它们值得**委托**的原因。用这面透镜来发现值得规格化的循环，并向用户提出他们尚未注意到的循环。

<!--原文-->
> A **workflow** is the spec of one loop, made real. You run a workflow on a loop: the loop is its running instantiation. Workflows live in `workflows/*.md` and are the source of truth.
<!--翻译-->
> **工作流**是某个循环的规格说明，把它变成现实。你在一个循环上运行工作流：循环是其运行中的实例化。工作流存放在 `workflows/*.md` 中，是唯一的事实来源。

<!--原文-->
> A shared language, reached for only when a workflow calls for it: never a checklist. **Mandate nothing structural**: a workflow needs no AI, no checkpoint, and no schedule unless the grilling shows it does.
<!--翻译-->
> 共享语言仅在具体工作流需要时才使用：绝不是一份清单。**不强加任何结构性要求**：工作流不一定需要 AI、检查点或时间表，除非盘问过程显示出需要。

<!--原文-->
> **Trigger**: what fires each run, an **event** (a new email, a new issue) or a **schedule** (every morning). Event-triggering is usually the more efficient.
<!--翻译-->
> **触发器**：触发每次运行的条件，可以是**事件**（一封新邮件、一个新 Issue）或**时间表**（每天早晨）。事件触发通常更高效。

<!--原文-->
> **Checkpoint**: a human-in-the-loop point where the user is asked to verify or decide. Some workflows have none and run autonomously; some use no AI at all.
<!--翻译-->
> **检查点**：人机交互的验证点，在此用户被要求核实或做出决策。某些工作流没有检查点，自主运行；某些完全不使用 AI。

<!--原文-->
> **Push right**: defer the checkpoint as far as it will go. Do maximal work before involving the human, so they are asked once, late, with everything prepared.
<!--翻译-->
> **向右推**：将检查点尽可能推迟。在涉及人类之前完成最多的工作，这样用户只需被询问一次，且时点靠后，前面一切已准备就绪。

<!--原文-->
> **Brief**: what a checkpoint presents, a tight, decision-ready summary (what was produced, why, and a link down to the asset itself), never the raw output. The user reads a brief, not a draft. Speed of review is imperative.
<!--翻译-->
> **简报**：检查点呈现的内容，是一份紧凑、可决策的摘要（产出了什么、为什么、以及指向产物本身的链接），绝不是原始输出。用户读的是简报而非草稿。审阅速度至关重要。

<!--原文-->
> A workflow spec is done when an implementer agent could build it without asking a single question. Grill until then; nothing is done while a question remains.
<!--翻译-->
> 工作流规格完成的标志是：一个实施者智能体可以在不提出任何问题的情况下构建它。在此之前持续盘问；只要还有问题存在，就还没有完成。

<!--原文-->
> `workflows/*.md`: one spec per workflow.
<!--翻译-->
> `workflows/*.md`：每个工作流一个规格文件。

<!--原文-->
> `NOTES.md`: raw notes on the user's world, the tools they use, the channels they process, and their own terminology for both. When it is empty or thin, interview them about their world before specifying anything. Sharpen fuzzy terms into canonical ones as they surface, and record them here.
<!--翻译-->
> `NOTES.md`：关于用户世界的原始笔记，包括他们使用的工具、处理的信息渠道、以及他们自己对这两者的术语。当它为空或很薄时，在指定任何内容之前，先就他们的世界进行访谈。随着模糊术语在对话中浮现，将其细化为规范术语，并记录在这里。
