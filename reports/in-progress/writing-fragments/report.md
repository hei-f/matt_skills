# writing-fragments 技能报告

## 第一部分：中文详细总结

### 概述

writing-fragments 是写作的**探索阶段**（explore）技能：从对话中挖掘原始碎片（fragments），不对结构做出任何承诺。这个阶段的目的是拓宽可能被写出的空间。承诺结构属于**利用阶段**（exploit），是其他技能的工作。writing-fragments 运行一个产生碎片的盘问会话，围绕用户想写的内容进行不懈的访谈。在此阶段施加阶段划分、大纲或文章结构是超出范围的。

当碎片从对话的任意一方产生时，将它们追加到一个单独的 Markdown 文件中。如果用户没有指定路径，先询问一次文档保存位置，然后在整个会话中记住它。从用户说的第一件事开始捕捉碎片，包括初始提示。首次写入时，在顶部放置一个带有工作标题的单一 H1（以后可以更改），不包含任何其他内容：无元数据、无目录、无日期。

### 什么是碎片（Fragment）

碎片是任何可能存活到最终文章中的文本片段。它必须**对作者可读**（作者能看出它是什么意思），但不需要定义其术语，也不需要让一个冷读者能够理解。标准是"这是不是一段好的文字？"，而不是"这是不是一个自包含的论证？"

碎片是刻意异构的。以下是可以成为碎片的例子：

- **精炼句子**：一句你想在某个地方使用但还不知道放在哪里的锐利句子。
- **主张**：附带一行理由的断言。
- **小品（Vignette）**：一件发生过的事、一段代码片段、一个场景、一个类比。
- **半成形想法**："关于 X 感觉像 Y 的某些东西，稍后展开。"
- **引用**：一段引文、一句对话、一行听到的话。
- **观察列表**：一组凭感觉相互关联的观察。
- **抱怨、坦白、妙语**。
- **引导词（Leading Word）**：一个紧凑的隐喻或新造词，整篇文章可以悬挂于其上（一个命名概念的术语，就像"曳光弹"或"战争迷雾"命名了整个模式）。

在这些之中，落地一个引导词是最有价值的碎片。它是承重的：在探索阶段命名了正确的词，它就会塑造后面的结构、过渡和标题，在整个利用阶段持续产生回报。当对话围绕一个反复出现的想法时，推动为它创造一个新词。

小说家的日记是这里的模型：多年非结构化的观察后来被挖掘作为原始材料。碎片就是这些观察。

### 文件格式

```markdown
# 工作标题

第一个碎片放在这里。

它可以是多个段落。可以包含列表、代码、引用：碎片自然采取的任何形状。

---

第二个碎片。

---

> 用户想保留的一句引文。

对它的反应。

---

- 一组相关的观察
- 凭感觉相互关联
- 想要彼此靠近
```

碎片之间用水平线（`\n---\n`）分隔。正文中没有标题。没有标签。没有超出添加顺序的排序逻辑。

### 写作节奏

- **静默追加**：不为每个碎片请求许可。顺带提及你添加了什么（"把这个加进去"），但不要用保存对话框打断对话。
- **每次写入前重新从磁盘读取文件**：用户可能在回合之间编辑、重新排序或删除了碎片，因此要保留他们的更改。绝不覆盖文件；仅追加（或者，如果用户要求，原地编辑特定碎片）。
- **响应用户指令**：用户可以随时说"删掉最后一个""重写那个，更锐利一点""合并这两个"。将这些视为一级指令。

---

## 第二部分：原文与翻译对照

<!--原文-->
> This is pure **explore**: widen the space of what could be written without committing to structure. Committing is *exploit*, a separate skill's job. Run a grilling session that produces fragments, interviewing the user relentlessly about whatever they want to write about. Imposing phases, outlines, or article structure is out of scope here.
<!--翻译-->
> 这是纯粹的**探索阶段**：拓宽可能被写出的空间，不对结构做出任何承诺。承诺结构属于*利用阶段*，是其他技能的工作。运行一个产生碎片的盘问会话，围绕用户想写的内容进行不懈的访谈。在此阶段施加阶段划分、大纲或文章结构是超出范围的。

<!--原文-->
> As fragments emerge from either side of the conversation, append them to a single markdown file. If the user did not pass a path, ask once where to save the document, then remember it for the rest of the session. Capture fragments from the very first thing the user says, including the initial prompt. On first write, put a single H1 at the top with a working title (it can change later) and nothing else: no metadata, no TOC, no date.
<!--翻译-->
> 当碎片从对话的任意一方产生时，将它们追加到一个单独的 Markdown 文件中。如果用户没有指定路径，先询问一次文档保存位置，然后在整个会话中记住它。从用户说的第一件事开始捕捉碎片，包括初始提示。首次写入时，在顶部放置一个带有工作标题的单一 H1（以后可以更改），不包含任何其他内容：无元数据、无目录、无日期。

<!--原文-->
> A fragment is any piece of text that might survive into the final article. It must be *readable by the author* (the author can tell what it means), but it does not need to define its terms or be comprehensible to a cold reader. The bar is "is this a piece of good writing?", not "is this a self-contained argument?"
<!--翻译-->
> 碎片是任何可能存活到最终文章中的文本片段。它必须*对作者可读*（作者能看出它是什么意思），但不需要定义其术语，也不需要让一个冷读者能够理解。标准是"这是不是一段好的文字？"，而不是"这是不是一个自包含的论证？"

<!--原文-->
> Fragments are deliberately heterogeneous. Examples of what could be a fragment: A sharp sentence you'd want to deploy somewhere but don't yet know where. A claim with a one-line justification. A vignette: a thing that happened, a code snippet, a scenario, an analogy. A half-thought: "something about how X feels like Y, work this out later." A quote, a piece of dialogue, an overheard line. A list of related observations that hang together by feel. A complaint, a confession, a punchline. A **leading word**: a compact metaphor or coinage the whole piece can hang on (one term that names the idea, the way *tracer bullets* or *fog of war* names a whole pattern).
<!--翻译-->
> 碎片是刻意异构的。以下是可以成为碎片的例子：一句你想在某个地方使用但还不知道放在哪里的锐利句子。附带一行理由的主张。小品：一件发生过的事、一段代码片段、一个场景、一个类比。半成形想法："关于 X 感觉像 Y 的某些东西，稍后展开。"一段引文、一句对话、一行听到的话。一组凭感觉相互关联的观察。一句抱怨、一次坦白、一句妙语。**引导词**：一个紧凑的隐喻或新造词，整篇文章可以悬挂于其上（一个命名概念的术语，就像*曳光弹*或*战争迷雾*命名了整个模式）。

<!--原文-->
> Of these, the leading word is the most valuable fragment to land. It is load-bearing: name the right one in explore and it shapes the structure, the transitions, and the title later, paying dividends through the entire exploit phase. When the conversation circles a recurring idea, push to coin a word for it.
<!--翻译-->
> 在这些之中，落地一个引导词是最有价值的碎片。它是承重的：在探索阶段命名了正确的词，它就会塑造后面的结构、过渡和标题，在整个利用阶段持续产生回报。当对话围绕一个反复出现的想法时，推动为它创造一个新词。

<!--原文-->
> The novelist's diary is the model: years of unstructured noticings that later get mined for raw material. Fragments are noticings.
<!--翻译-->
> 小说家的日记是这里的模型：多年非结构化的观察后来被挖掘作为原始材料。碎片就是这些观察。

<!--原文-->
> Fragments are separated by a horizontal rule (`\n---\n`). No headings inside the body. No tags. No order beyond the order they were added.
<!--翻译-->
> 碎片之间用水平线（`\n---\n`）分隔。正文中没有标题。没有标签。没有超出添加顺序的排序逻辑。

<!--原文-->
> Append silently. Don't ask permission for each fragment. Mention what you added in passing ("adding that"), but don't interrupt the conversation with save dialogs.
<!--翻译-->
> 静默追加。不为每个碎片请求许可。顺带提及你添加了什么（"把这个加进去"），但不要用保存对话框打断对话。

<!--原文-->
> Before every write: re-read the file from disk. The user may have edited, reordered, or deleted fragments between turns, so preserve their changes. Never overwrite the file; only append (or, if the user asks, edit a specific fragment in place).
<!--翻译-->
> 每次写入前：重新从磁盘读取文件。用户可能在回合之间编辑、重新排序或删除了碎片，因此要保留他们的更改。绝不覆盖文件；仅追加（或者，如果用户要求，原地编辑特定碎片）。

<!--原文-->
> The user can say "cut the last one", "rewrite that one sharper", "merge those two" at any time. Treat those as first-class instructions.
<!--翻译-->
> 用户可以随时说"删掉最后一个""重写那个，更锐利一点""合并这两个"。将这些视为一级指令。
