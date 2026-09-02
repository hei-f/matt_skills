# writing-beats 技能报告

## 第一部分：中文详细总结

### 概述

writing-beats 是写作的**利用阶段**（exploit）技能：将原始材料组装成一段"节拍之旅"（journey of beats），采用"选择你自己的冒险"（choose-your-own-adventure）风格。探索阶段已经完成，素材堆是固定的。你需要选定一条穿越素材的路径，从素材堆中开采内容来填充每个节拍。

### 核心概念：基础化（Grounding）

每篇文章都建立在读者已知的概念之上。**基础化**是 writing-beats 的核心机制——它定义了每个节拍可以安全地使用哪些概念。

每个**概念**必须在节拍可以依赖它之前被**夯实**：读者要么一开始就知道它，要么在之前的节拍中接触过它。一个试图使用未夯实概念的节拍会让读者迷失方向——这是写作之旅中不可逾越的规则。基本单位是概念本身，而不是表达它的词语：即使没有使用任何行话，一个节拍也可能让读者面对一个他们缺乏的概念。当概念有名字（一个**术语**）时，夯实它意味着同时落地概念和术语。

一个概念可以通过两种方式被夯实：

- **前置条件（Prerequisite）**：在第一个节拍之前就已经夯实。读者带着它进入文章。在开始时就固定下来。
- **引入（Introduced）**：某个节拍建立了这个概念，此后它对所有后续节拍都是夯实的。

因此，每个节拍做两件事：它**需要**已经夯实的概念，并且它**夯实**新的概念。维护一个不断更新的"已夯实列表"，并在每个节拍落地后更新它。

### 选择你自己的冒险机制

基础化机制塑造了"选择你自己的冒险"结构。某个待选节拍只有在它需要的所有概念都已夯实的情况下才是**可达的**；选择一个夯实了概念 X 的节拍，就会解锁所有之前在等待 X 的节拍。当你提供下一组节拍选项时，它们都必须从当前已夯实集合可达，并且你需要说明每个选项夯实了什么，这样用户就能看到它打开了哪些路径。

关键杠杆在于：将什么设为前置条件，什么在文章内部夯实。前期要求太多会排斥不具备这些知识的读者；在内部夯实太多会让早期节拍淹没在定义中。在建立前置条件时与用户敲定这一点，并在后续某个吸引人的节拍需要尚未夯实的概念时重新审视：解决方法是或者在它前面插入一个夯实节拍，或者将该概念提升为前置条件。

### 什么是节拍（Beat）

一个节拍是写作之旅中的一步。它做一件事：设置场景、落下一个观点、提出一个问题、插入一段旁白、转换角度。然后停止，让读者处在一个下一个节拍可以转折的位置。

节拍的大小由它需要什么决定：

- 如果整个动作只需要一句话，那就是一句话（"然后三个星期什么都没发生。"）。
- 如果动作需要铺垫，就是一个短段落。
- 如果节拍是一个自包含的小品、论证或示例，可以是多个段落。

如果一个"节拍"需要五个段落和三个子标题，它就不是一个节拍——它是两个节拍粘在一起。拆开它。

### 从素材堆中提取

从原始素材堆中拉取材料来填充每个节拍。可以改写、拆分、重组或引用。素材堆是一座采石场。

### 结束旅程

文章在旅程完成时结束，而不是在素材堆耗尽时结束。大多数素材堆会有没有进入文章的残余碎片。这没有关系——这正是拥有比需要更多的原始素材的意义所在。

### 写作节奏

- 一次只追加一个节拍。绝不提前写后面的内容。
- 每次写入前重新从磁盘读取文章文件。绝对保持用户所做的编辑。
- 如果用户对前面的节拍做了实质性修改，让这种修改影响接下来的内容。
- 如果用户说"重写那个节拍"或"回去尝试另一个节拍 3"，照做：原地编辑，其余部分保持不变。

---

## 第二部分：原文与翻译对照

<!--原文-->
> The user has passed (or will pass) a markdown file of raw material. This is **exploit**: the exploring is done, the pile is fixed. Commit to a path through it and mine the pile to fill each beat.
<!--翻译-->
> 用户已经传入（或将传入）一个原始素材的 Markdown 文件。这是**利用阶段**：探索已经完成，素材堆是固定的。选定一条穿越它的路径，从素材堆中开采内容来填充每个节拍。

<!--原文-->
> **Establish the prerequisites.** Before any beats, settle with the user what the audience already knows walking in: the concepts that are **grounded** from the start. Everything else must be grounded by a beat before a later beat can use it.
<!--翻译-->
> **建立前置条件。** 在任何节拍之前，与用户敲定读者从一开始就知道什么：这些概念是**已经夯实的**。其他所有概念必须被某个节拍夯实后，后面的节拍才能使用它。

<!--原文-->
> Write 2–3 candidate **starting beats**, drawn from the raw material. Each is a different entry point into the article. Each may only lean on grounded concepts; note what new concepts each one grounds. Show the user the beats before writing to the article file. The user picks one. Preview what beats that pick unlocks, as if the user is seeing a little way down the path.
<!--翻译-->
> 从原始素材中写出 2 到 3 个候选的**起始节拍**。每一个是进入文章的不同入口点。每个节拍只能依赖于已夯实的概念；标注每个节拍夯实了哪些新概念。在写入文章文件之前向用户展示这些节拍。用户选择一个。预览该选择解锁了哪些节拍，就像让用户看到前方路径的一小段。

<!--原文-->
> Once the user picks a starting beat, write **only that beat** to the article file. A beat may be one sentence or several paragraphs, whatever that beat naturally is. Stop there.
<!--翻译-->
> 一旦用户选择了起始节拍，将**仅这一个节拍**写入文章文件。一个节拍可以是一句话或多个段落，以该节拍自然需要的形式为准。就此打住。

<!--原文-->
> Re-read the article file from disk. Then offer 2–3 candidate **next beats**: different directions the journey could pivot to from where the article now stands. Each must be reachable from the current grounded set; note what each one grounds.
<!--翻译-->
> 从磁盘重新读取文章文件。然后提供 2 到 3 个候选的**下一个节拍**：从文章当前位置出发，旅程可以转向的不同方向。每个节拍必须从当前已夯实集合可达；标注每个节拍夯实了什么。

<!--原文-->
> Loop steps 3–5 until the article reaches a natural end.
<!--翻译-->
> 循环步骤 3 到 5，直到文章到达自然终点。

<!--原文-->
> Every **concept** has to be **grounded** before a beat can lean on it: the audience either walked in knowing it or met it in an earlier beat. A beat that reaches for an ungrounded concept loses the reader; that is the one move the journey can't make. The unit is the concept, not the word for it: a beat can lean on an idea the reader lacks even with no jargon in sight. Where a concept has a name (a **term**), grounding it means landing the idea and the term together.
<!--翻译-->
> 每个**概念**必须在节拍可以依赖它之前被**夯实**：读者要么一开始就知道它，要么在之前的节拍中接触过它。一个试图使用未夯实概念的节拍会让读者迷失方向——这是写作之旅中不可逾越的一步。基本单位是概念本身，而不是表达它的词语：即使没有使用任何行话，一个节拍也可能让读者面对一个他们缺乏的概念。当概念有名字（一个**术语**）时，夯实它意味着同时落地概念和术语。

<!--原文-->
> A concept gets grounded one of two ways: **Prerequisite**: grounded before the first beat. The audience brings it. Fixed at the start. **Introduced**: a beat establishes it, and from then on it's grounded for every later beat.
<!--翻译-->
> 一个概念可以通过两种方式被夯实：**前置条件**：在第一个节拍之前就已经夯实。读者带着它进入文章。在开始时就固定下来。**引入**：某个节拍建立了这个概念，此后它对所有后续节拍都是夯实的。

<!--原文-->
> So each beat does two jobs: it **requires** concepts that are already grounded, and it **grounds** new ones. Keep a running list of what's grounded so far, and update it each time a beat lands.
<!--翻译-->
> 因此，每个节拍做两件事：它**需要**已经夯实的概念，并且它**夯实**新的概念。维护一个不断更新的"已夯实列表"，并在每个节拍落地后更新它。

<!--原文-->
> This is what shapes the choose-your-own-adventure. A candidate beat is only reachable if everything it requires is already grounded; picking a beat that grounds concept X unlocks every beat that was waiting on X. When you offer next beats, they must all be reachable from the current grounded set, and say what each one grounds, so the user can see which paths it opens.
<!--翻译-->
> 这就是塑造"选择你自己的冒险"结构的机制。某个待选节拍只有在它需要的所有概念都已夯实的情况下才是可达的；选择一个夯实了概念 X 的节拍，就会解锁所有之前在等待 X 的节拍。当你提供下一组节拍选项时，它们都必须从当前已夯实集合可达，并且你需要说明每个选项夯实了什么，这样用户就能看到它打开了哪些路径。

<!--原文-->
> The big lever is what you make a prerequisite versus what you ground inside the piece. Demand too much up front and you shut out readers who don't have it; ground too much inside and the early beats drown in definitions. Settle this with the user when you establish prerequisites, and revisit it whenever a tempting beat turns out to require a concept nothing has grounded yet: the fix is either a grounding beat before it, or promoting the concept to a prerequisite.
<!--翻译-->
> 关键杠杆在于：将什么设为前置条件，什么在文章内部夯实。前期要求太多会排斥不具备这些知识的读者；在内部夯实太多会让早期节拍淹没在定义中。在建立前置条件时与用户敲定这一点，并在后续某个吸引人的节拍需要尚未夯实的概念时重新审视：解决方法是或者在它前面插入一个夯实节拍，或者将该概念提升为前置条件。

<!--原文-->
> A beat is one move in the journey. It does one thing: sets a scene, lands a point, asks a question, drops an aside, twists the angle. Then it stops, leaving the reader at a place where the next beat can pivot.
<!--翻译-->
> 一个节拍是写作之旅中的一步。它做一件事：设置场景、落下一个观点、提出一个问题、插入一段旁白、转换角度。然后停止，让读者处在一个下一个节拍可以转折的位置。

<!--原文-->
> A beat is sized by what it needs: A single sentence if that's all the move is ("And then nothing happened for three weeks."). A short paragraph if the move needs setup. Multiple paragraphs if the beat is a self-contained vignette, argument, or example.
<!--翻译-->
> 节拍的大小由它需要什么决定：如果整个动作只需要一句话，那就是一句话（"然后三个星期什么都没发生。"）。如果动作需要铺垫，就是一个短段落。如果节拍是一个自包含的小品、论证或示例，可以是多个段落。

<!--原文-->
> If a "beat" needs five paragraphs and three subheadings, it's not a beat; it's two beats glued together. Split it.
<!--翻译-->
> 如果一个"节拍"需要五个段落和三个子标题，它就不是一个节拍——它是两个节拍粘在一起。拆开它。

<!--原文-->
> Pull material from the raw pile to populate each beat. You can paraphrase, split, recombine, or quote. The pile is a quarry.
<!--翻译-->
> 从原始素材堆中拉取材料来填充每个节拍。可以改写、拆分、重组或引用。素材堆是一座采石场。

<!--原文-->
> The article ends when the journey is complete, not when the pile is empty. Most piles will have leftover fragments that don't make it in. That is fine; that is the point of having more raw material than you need.
<!--翻译-->
> 文章在旅程完成时结束，而不是在素材堆耗尽时结束。大多数素材堆会有没有进入文章的残余碎片。这没有关系——这正是拥有比需要更多的原始素材的意义所在。

<!--原文-->
> Append one beat at a time. Never write ahead. Re-read the article file from disk before every write. Preserve user edits absolutely. If the user edits a previous beat substantially, let it change what comes next. If the user says "rewrite that beat" or "go back and try a different beat 3", do it: edit in place, leave the rest alone.
<!--翻译-->
> 一次只追加一个节拍。绝不提前写后面的内容。每次写入前重新从磁盘读取文章文件。绝对保持用户所做的编辑。如果用户对前面的节拍做了实质性修改，让这种修改影响接下来的内容。如果用户说"重写那个节拍"或"回去尝试另一个节拍 3"，照做：原地编辑，其余部分保持不变。
