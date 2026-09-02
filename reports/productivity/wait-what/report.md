# wait-what -- 消息没理解时立即触发

## 第一部分：详细总结

### 概述

`wait-what` 是一个**用户调用（User-invoked）**的 Skill，当你无法理解一条消息时输入它。Agent 随后会重新表述它刚说的话：添加你缺失的上下文，用通俗英语写作，并使用你项目的 CONTEXT.md 中的词汇。

该 Skill 只有三行长。这是设计，不是未完成的草稿。与啰嗦作斗争的 Skill 会因为自身膨胀而失败：一个四百行的简洁 Skill 仍然会使模型啰嗦，因为模型读取的是体量，而非请求。这个 Skill 携带一个精确的主导词，别无其他。

### 调用方式与触发场景

通过输入 `/wait-what` 调用。Agent 不会自己触发，也不应该自己触发——只有你知道你何时跟不上了。

在注意到自己在略读的瞬间就使用它。Agent 已经漂移到了它发明的行话中、堆叠了五个缩写、或解释了一个你从未见过前提的决策。它修复你所在的当前对话。要完全阻止行话出现，使用 grill-with-docs，它提前构建共享语言。

### 核心设计：主导词是机制

主导词是 **wait**。"Be concise"（更简洁点）是关于 Agent 输出的指令，模型通过裁剪词语来服从，让你进一步迷失。**Wait** 关乎的是*你的*状态。它表示在这里理解失败了。听到"be brief"（简短点）的 Agent 写电报。听到"wait, you lost me"（等等，我没跟上）的 Agent 会退后一步并解释。

这个区别就是整个 Skill。每一个针对啰嗦的流行修复都命名*输出*：`/tldr`、`/no-fluff`、`/talk-normal`。模型过度纠正为原始人语域，更短但不会更清晰。命名*听众*同时要求两半：更少的词语**和**你缺失的上下文。

Skill 说的是重新表述**那个**（that），而不是"上一条消息"。让你迷失的通常比一个段落更大，所以 Agent 自行决定回溯多远。

### 与已有语言的对接

Skill 正文复用你全局 CLAUDE.md 和项目 CONTEXT.md 中已有的主导词。ASD-STE100 简化技术英语设置语域。通用语言提供名词。Skill、CLAUDE.md 和 CONTEXT.md 触及相同的 token，因此调用它不是一个新指令——它是对 Agent 已经同意的某条指令的提醒。

如果你没有 CONTEXT.md（也没有 CONTEXT-MAP.md 指向当前上下文的），Skill 仍然有效。你只失去领域词汇那一半。

### 常见问题

由于该 Skill 极短（三行）且单一用途，公开文档中未列出常见问题部分。其设计本身就是对过度复杂化的明确拒绝。

### 判断是否正常工作的标志

- 重新表述**更短且更清晰**，而非更短且更生硬
- 它添加了你缺失的前提，而非仅仅删除词语
- 项目名词替换了发明的名词。你 CONTEXT.md 中的术语重新出现
- 你可以连续使用两次，它不会退化为简洁到生硬

### 在 Skill 体系中的位置

你可以在任何时刻、任何对话中、任何其他 Skill 内部使用 `wait-what`。它事后修复一条消息。真正的治愈是提前达成一致的共享语言，那就是 grill-with-docs：一个在运行中执行 domain-modeling 的 grilling 会话，使你们都使用的词语落到 CONTEXT.md 中。如果你不确定哪个 Skill 适合当前时刻，ask-matt 为你导航。

---

## 第二部分：原文与翻译

<!--原文-->
> `wait-what` is what you type when a message didn't land. The agent then re-pitches what it just said. It adds the context you were missing, writes in plain English, and uses the vocabulary from your project's `CONTEXT.md`.

<!--翻译-->
> `wait-what` 是当一条消息没理解时你输入的东西。Agent 随后重新表述它刚说的话。它添加你缺失的上下文，用通俗英语写作，并使用你项目的 `CONTEXT.md` 中的词汇。

<!--原文-->
> The skill is three lines long. That is the design, not an unfinished draft. Skills that fight verbosity fail by growing: a four-hundred-line concision skill still leaves the model verbose, because the model reads the volume, not the plea. This one carries a single precise leading word and nothing else.

<!--翻译-->
> 该 Skill 只有三行长。这是设计，不是未完成的草稿。与啰嗦作斗争的 Skill 会因为自身膨胀而失败：一个四百行的简洁 Skill 仍然会使模型啰嗦，因为模型读取的是体量，而非请求。这个 Skill 携带一个精确的主导词，别无其他。

<!--原文-->
> Use it the second you notice you're skimming. The agent has drifted into jargon it invented, stacked five acronyms, or explained a decision whose premise you never saw. It fixes the conversation you're already in. To stop the jargon arriving at all, use grill-with-docs, which builds the shared language upfront.

<!--翻译-->
> 在注意到自己在略读的瞬间就使用它。Agent 已经漂移到了它发明的行话中、堆叠了五个缩写、或解释了一个你从未见过前提的决策。它修复你所在的当前对话。要完全阻止行话出现，使用 grill-with-docs，它提前构建共享语言。

<!--原文-->
> The leading word is **wait**. "Be concise" is an instruction about the agent's output, and the model obeys it by clipping words and losing you further. **Wait** is about *your* state. It says comprehension failed here. An agent that hears "be brief" writes telegrams. An agent that hears "wait, you lost me" backs up and explains.

<!--翻译-->
> 主导词是 **wait**。"Be concise"（更简洁点）是关于 Agent 输出的指令，模型通过裁剪词语来服从，让你进一步迷失。**Wait** 关乎的是*你的*状态。它表示在这里理解失败了。听到"be brief"（简短点）的 Agent 写电报。听到"wait, you lost me"（等等，我没跟上）的 Agent 会退后一步并解释。

<!--原文-->
> That difference is the whole skill. Every popular fix for verbosity names the *output*: `/tldr`, `/no-fluff`, `/talk-normal`. The model over-corrects into a caveman register that is shorter and no clearer. Naming the *listener* asks for both halves at once: fewer words **and** the context you were missing.

<!--翻译-->
> 这个区别就是整个 Skill。每一个针对啰嗦的流行修复都命名*输出*：`/tldr`、`/no-fluff`、`/talk-normal`。模型过度纠正为原始人语域，更短但不会更清晰。命名*听众*同时要求两半：更少的词语**和**你缺失的上下文。

<!--原文-->
> The skill says re-pitch **that**, not "that last message". What lost you is usually bigger than one paragraph, so the agent decides how far back to go.

<!--翻译-->
> Skill 说的是重新表述**那个**，而不是"上一条消息"。让你迷失的通常比一个段落更大，所以 Agent 自行决定回溯多远。

<!--原文-->
> The body reuses the leading words already in your global `CLAUDE.md` and your project's `CONTEXT.md`. ASD-STE100 Simplified Technical English sets the register. The ubiquitous language supplies the nouns. The skill, `CLAUDE.md` and `CONTEXT.md` reach for the same tokens, so invoking it is not a new instruction. It is a reminder of one the agent already agreed to.

<!--翻译-->
> Skill 正文复用你全局 `CLAUDE.md` 和项目 `CONTEXT.md` 中已有的主导词。ASD-STE100 简化技术英语设置语域。通用语言提供名词。Skill、`CLAUDE.md` 和 `CONTEXT.md` 触及相同的 token，因此调用它不是一个新指令——它是对 Agent 已经同意的某条指令的提醒。

<!--原文-->
> If you have no `CONTEXT.md` (and no `CONTEXT-MAP.md` pointing to one for the context at hand), the skill still works. You lose only the domain-vocabulary half.

<!--翻译-->
> 如果你没有 `CONTEXT.md`（也没有 `CONTEXT-MAP.md` 指向当前上下文的），Skill 仍然有效。你只失去领域词汇那一半。

<!--原文-->
> It's working if: The re-pitch is **shorter and clearer**, not shorter and blunter. It adds the premise you were missing, instead of only deleting words. Project nouns replace invented ones. The terms in your `CONTEXT.md` come back. You can use it twice in a row, and it does not degrade into terseness.

<!--翻译-->
> 判断是否正常工作的标志：重新表述**更短且更清晰**，而非更短且更生硬。它添加了你缺失的前提，而非仅仅删除词语。项目名词替换了发明的名词。你 `CONTEXT.md` 中的术语重新出现。你可以连续使用两次，它不会退化为简洁到生硬。

<!--原文-->
> You can use `wait-what` at any point, in any conversation, inside any other skill. It repairs one message after the fact. The real cure is a shared language agreed upfront, and that is grill-with-docs: a grilling session that runs domain-modeling as it goes, so the words you both use land in your `CONTEXT.md`.

<!--翻译-->
> 你可以在任何时刻、任何对话中、任何其他 Skill 内部使用 `wait-what`。它事后修复一条消息。真正的治愈是提前达成一致的共享语言，那就是 grill-with-docs：一个在运行中执行 domain-modeling 的 grilling 会话，使你们都使用的词语落到 `CONTEXT.md` 中。
