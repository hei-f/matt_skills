# writing-shape 技能报告

## 第一部分：中文详细总结

### 概述

writing-shape 是写作的**利用阶段**（exploit）技能：将原始材料逐段塑造成文章。用户传入（或将传入）一个原始素材的 Markdown 文件。将其视为输入素材堆：从整齐的碎片列表到非结构化的长篇文字再到对话记录都可以，格式不重要。在做任何事情之前，从头到尾完整阅读它。

然后运行一个塑造会话，产生一篇独立的文章文档。这是利用阶段：探索已经完成，素材堆是固定的——选定结构并从素材堆中开采内容来填充它。不要编辑原始素材文件：对此技能而言它是只读的。

如果用户没有指定文章保存位置，先询问一次并记住路径。

### 塑造循环

1. **读取素材堆**：完整阅读输入文件。形成对其中内容的感知。
2. **建立前置条件**：与用户敲定读者从一开始就知道什么——这些概念是**已经夯实的**。其他所有概念必须被某个段落夯实后，后面的段落才能使用它。
3. **起草 2 到 3 个候选开头**：每个开头应暗示文章的不同论点或角度。全部展示。要求用户选择或组合一个混合版本。选定的开头定义了文章其余部分必须完成的任务。
4. **逐段推进**：开头落地后，问"鉴于这个开头，读者下一步需要听到什么？"从素材堆中拉取材料来回答。下一个段落只能依赖于已夯实的概念，并在落地时夯实新的概念。就下一个段落的格式进行辩论：段落、列表、表格、标注框、引用、代码块。每个格式选择都应当有意识且可辩驳。
5. **随行追加**：不要批量写入。在达成一致后立即写入每个段落或区块，让用户看到文章正在成型。
6. **循环步骤 4 直到文章完成**：由用户决定何时完成。

### 基础化（Grounding）

与 writing-beats 相同的机制，但应用于段落级别：

每个**概念**必须在段落可以依赖它之前被**夯实**：读者要么一开始就知道它，要么在之前的段落中接触过它。一个试图使用未夯实概念的段落会让读者迷失方向。基本单位是概念本身，而不是表达它的词语。当概念有名字（一个**术语**）时，夯实它意味着同时落地概念和术语。

夯实方式：
- **前置条件**：在开头之前已夯实。读者自带。开始时固定。
- **引入**：某个段落建立了这个概念，此后它在文章其余部分都是夯实的。

维护一个已夯实的运行列表。当你问"读者下一步需要听到什么？"时，下一个动作需要的但未夯实的概念本身就是答案：先夯实它（在这里或更早的段落），否则你无法做这个动作。

关键杠杆同样是前置条件与内部夯实的权衡。前期要求太多会排斥读者；在内部夯实太多会让开头淹没在定义中。

### 对话感（Conversational Feel）

这是一个倒置的盘问会话。在构思阶段，问题是"你实际上在注意什么？"而在这里是"这篇文章实际上在论证什么，读者需要以什么顺序听到它？"要推进。拒绝让薄弱的过渡蒙混过关。如果一个段落不配占据它的位置，删掉它。

持续使用的具体动作：

- "这个段落为读者做了什么前一个段落没有做的事？"
- "如果我把这段删掉，什么会断裂？"
- "这是散文，还是应该是一个列表？为什么是散文？"
- "这句话在做两件事：拆开它或者选一个。"
- "开头承诺了 X。我们已经漂移到了 Y。要么重新穿线，要么改开头。"

### 从素材堆中提取

将原始材料视为采石场而非剧本。提取一个碎片，改造它以适应周围段落，然后放入。一个碎片可能被拆分到多个段落、与另一个合并、或被改写。素材堆的任务是被开采；文章的任务是以一种统一的声音来阅读。

如果素材堆缺少文章所需的内容，明确地指出这个空缺："我们这里需要一个例子，素材堆里没有。现在给我一个，否则我们删掉这部分。"

### 格式辩论

在选择如何呈现一个区块时，与用户一起权衡以下取舍，而不是默默决定：

- **散文 vs. 列表**：散文承载论证；列表承载并列项。如果项目不是真正并列的，散文更好。如果是，列表更易于扫描。
- **内联 vs. 标注框**：提示、警告和旁白放入标注框（`> [!TIP]`、`> [!NOTE]`），但仅在它们确实会打断主要论证内联流的情况下。否则留在内联中。
- **表格 vs. 重复结构**：如果相同结构重复 3 次以上且字段相同，用表格。否则用带粗体引导词的散文。
- **引用 vs. 改写**：当原始措辞本身是重点时用引用。当只有想法重要时用改写。
- **代码块 vs. 内联代码**：多行、可运行或说明性的用块级。单个标记或标识符用内联。

### 写作节奏

- 每个区块达成一致后立即追加到文章文件。
- 每次写入前重新从磁盘读取文件：用户可能在回合之间编辑过。绝不盲目覆盖。
- 如果用户想重写某个段落，原地编辑该特定段落；其余部分保持不变。

### 超出范围

- 挖掘不在素材堆中的新碎片（按"从素材堆中提取"的方式处理空缺）。
- 编辑原始素材文件。
- 发布、为特定平台格式化、或添加用户未要求的 frontmatter。

---

## 第二部分：原文与翻译对照

<!--原文-->
> The user has passed (or will pass) a markdown file of raw material. Treat it as the input pile: anything from a tidy list of fragments to a wall of unstructured prose to a transcript. The format does not matter. Read it end-to-end before doing anything else.
<!--翻译-->
> 用户已经传入（或将传入）一个原始素材的 Markdown 文件。将其视为输入素材堆：从整齐的碎片列表到非结构化的长篇文字再到对话记录都可以。格式不重要。在做任何事情之前，从头到尾完整阅读它。

<!--原文-->
> Then run a shaping session that produces a separate article document. This is **exploit**: the exploring is done, the pile is fixed: commit to a structure and mine the pile to fill it. Do not edit the raw material file: it is read-only to this skill.
<!--翻译-->
> 然后运行一个塑造会话，产生一篇独立的文章文档。这是**利用阶段**：探索已经完成，素材堆是固定的——选定结构并从素材堆中开采内容来填充它。不要编辑原始素材文件：对此技能而言它是只读的。

<!--原文-->
> **Read the pile.** Read the input file in full. Form a sense of what's in it.
<!--翻译-->
> **读取素材堆。** 完整阅读输入文件。形成对其中内容的感知。

<!--原文-->
> **Establish the prerequisites.** Settle with the user what the reader knows walking in: the concepts that are **grounded** from the start. Everything else must be grounded by a block before a later block can lean on it.
<!--翻译-->
> **建立前置条件。** 与用户敲定读者从一开始就知道什么：这些概念是**已经夯实的**。其他所有概念必须被某个段落夯实后，后面的段落才能使用它。

<!--原文-->
> **Draft 2–3 candidate openings.** Each opening should imply a different thesis or angle for the article. Show all of them. Force the user to pick or compose a hybrid. The chosen opening defines what the rest of the article must do.
<!--翻译-->
> **起草 2 到 3 个候选开头。** 每个开头应暗示文章的不同论点或角度。全部展示。要求用户选择或组合一个混合版本。选定的开头定义了文章其余部分必须完成的任务。

<!--原文-->
> **Grow paragraph by paragraph.** After the opening lands, ask "given this opening, what does the reader need to hear next?" Pull material from the pile to answer. The next block may only lean on grounded concepts, and grounds new ones as it lands. Argue about the form the next block takes: a paragraph, a list, a table, a callout, a quote, a code block. Each format choice should be deliberate and defensible.
<!--翻译-->
> **逐段推进。** 开头落地后，问"鉴于这个开头，读者下一步需要听到什么？"从素材堆中拉取材料来回答。下一个段落只能依赖于已夯实的概念，并在落地时夯实新的概念。就下一个段落的格式进行辩论：段落、列表、表格、标注框、引用、代码块。每个格式选择都应当有意识且可辩驳。

<!--原文-->
> **Append to the article file as you go.** Don't batch. Write each agreed paragraph or block immediately so the user can see the article taking shape.
<!--翻译-->
> **随行追加到文章文件。** 不要批量写入。在达成一致后立即写入每个段落或区块，让用户看到文章正在成型。

<!--原文-->
> **Loop step 4 until the article is done.** The user decides when it's done.
<!--翻译-->
> **循环步骤 4 直到文章完成。** 由用户决定何时完成。

<!--原文-->
> Every **concept** has to be **grounded** before a block can lean on it: the reader either walked in knowing it or met it in an earlier block. A block that reaches for an ungrounded concept loses the reader. The unit is the concept, not the word for it: a block can lean on an idea the reader lacks even with no jargon in sight. Where a concept has a name (a **term**), grounding it means landing the idea and the term together.
<!--翻译-->
> 每个**概念**必须在段落可以依赖它之前被**夯实**：读者要么一开始就知道它，要么在之前的段落中接触过它。一个试图使用未夯实概念的段落会让读者迷失方向。基本单位是概念本身，而不是表达它的词语：即使没有使用任何行话，一个段落也可能让读者面对一个他们缺乏的概念。当概念有名字（一个**术语**）时，夯实它意味着同时落地概念和术语。

<!--原文-->
> A concept gets grounded one of two ways: **Prerequisite**: grounded before the opening. The reader brings it. Fixed at the start. **Introduced**: a block establishes it, and from then on it's grounded for the rest of the article.
<!--翻译-->
> 一个概念可以通过两种方式被夯实：**前置条件**：在开头之前已夯实。读者自带。开始时固定。**引入**：某个段落建立了这个概念，此后它在文章其余部分都是夯实的。

<!--原文-->
> Keep a running list of what's grounded. When you ask "what does the reader need to hear next?", an ungrounded concept the next move needs is itself the answer: ground it first (here or in an earlier block) or you can't make the move.
<!--翻译-->
> 维护一个已夯实的运行列表。当你问"读者下一步需要听到什么？"时，下一个动作需要的但未夯实的概念本身就是答案：先夯实它（在这里或更早的段落），否则你无法做这个动作。

<!--原文-->
> The lever is what you make a prerequisite versus what you ground inside the article. Demand too much up front and you shut readers out; ground too much inside and the opening drowns in definitions. Settle it with the user when you establish prerequisites.
<!--翻译-->
> 关键杠杆在于：将什么设为前置条件，什么在文章内部夯实。前期要求太多会排斥读者；在内部夯实太多会让开头淹没在定义中。在建立前置条件时与用户敲定这一点。

<!--原文-->
> This is a grilling session inverted. In ideation, the question was "what are you actually noticing?" Here it's "what is this article actually arguing, and in what order does the reader need to hear it?" Push back. Refuse to let weak transitions slide. If a paragraph doesn't earn its place, cut it.
<!--翻译-->
> 这是一个倒置的盘问会话。在构思阶段，问题是"你实际上在注意什么？"而在这里是"这篇文章实际上在论证什么，读者需要以什么顺序听到它？"要推进。拒绝让薄弱的过渡蒙混过关。如果一个段落不配占据它的位置，删掉它。

<!--原文-->
> "What does this paragraph do for the reader that the previous one didn't?" "If I cut this, what breaks?" "Is this prose, or should it be a list? Why prose?" "This sentence is doing two jobs: split it or pick one." "The opening promised X. We've drifted to Y. Either re-thread it or change the opening."
<!--翻译-->
> "这个段落为读者做了什么前一个段落没有做的事？""如果我把这段删掉，什么会断裂？""这是散文，还是应该是一个列表？为什么是散文？""这句话在做两件事：拆开它或者选一个。""开头承诺了 X。我们已经漂移到了 Y。要么重新穿线，要么改开头。"

<!--原文-->
> Treat the raw material as a quarry, not a script. Pull a fragment, rework it to fit the surrounding paragraph, and place it. A fragment may be split across multiple paragraphs, merged with another, or paraphrased. The pile's job is to be mined; the article's job is to read as one voice.
<!--翻译-->
> 将原始材料视为采石场而非剧本。提取一个碎片，改造它以适应周围段落，然后放入。一个碎片可能被拆分到多个段落、与另一个合并、或被改写。素材堆的任务是被开采；文章的任务是以一种统一的声音来阅读。

<!--原文-->
> If the pile lacks something the article needs, name the gap explicitly: "We need an example here and the pile doesn't have one. Give me one now or we cut this section."
<!--翻译-->
> 如果素材堆缺少文章所需的内容，明确地指出这个空缺："我们这里需要一个例子，素材堆里没有。现在给我一个，否则我们删掉这部分。"

<!--原文-->
> When choosing how to render a block, weigh these tradeoffs out loud with the user, not silently: **Prose vs. list.** Prose carries argument; lists carry parallel items. If items aren't truly parallel, prose is better. If they are, a list is faster to scan. **Inline vs. callout.** Tips, warnings, and asides go in callouts (`> [!TIP]`, `> [!NOTE]`), but only if they'd genuinely derail the main argument inline. Otherwise leave them inline. **Table vs. repeated structure.** If the same shape repeats 3+ times with the same fields, a table. Otherwise prose with bold leads. **Quote vs. paraphrase.** Quote when the original wording is the point. Paraphrase when only the idea matters. **Code block vs. inline code.** Multi-line, runnable, or illustrative → block. Single token or identifier → inline.
<!--翻译-->
> 在选择如何呈现一个区块时，与用户一起权衡以下取舍，而不是默默决定：**散文 vs. 列表。** 散文承载论证；列表承载并列项。如果项目不是真正并列的，散文更好。如果是，列表更易于扫描。**内联 vs. 标注框。** 提示、警告和旁白放入标注框（`> [!TIP]`、`> [!NOTE]`），但仅在它们确实会打断主要论证内联流的情况下。否则留在内联中。**表格 vs. 重复结构。** 如果相同结构重复 3 次以上且字段相同，用表格。否则用带粗体引导词的散文。**引用 vs. 改写。** 当原始措辞本身是重点时用引用。当只有想法重要时用改写。**代码块 vs. 内联代码。** 多行、可运行或说明性的用块级。单个标记或标识符用内联。

<!--原文-->
> Append to the article file as each block is agreed. Re-read the file from disk before every write: the user may have edited between turns. Never overwrite blindly. If the user wants a paragraph rewritten, edit that specific paragraph in place; leave the rest alone.
<!--翻译-->
> 每个区块达成一致后立即追加到文章文件。每次写入前重新从磁盘读取文件：用户可能在回合之间编辑过。绝不盲目覆盖。如果用户想重写某个段落，原地编辑该特定段落；其余部分保持不变。

<!--原文-->
> Out of scope: Mining for new fragments that aren't in the pile (handle gaps as in "Pulling from the pile"). Editing the raw material file. Publishing, formatting for a specific platform, or adding frontmatter the user didn't ask for.
<!--翻译-->
> 超出范围：挖掘不在素材堆中的新碎片（按"从素材堆中提取"的方式处理空缺）。编辑原始素材文件。发布、为特定平台格式化、或添加用户未要求的 frontmatter。
