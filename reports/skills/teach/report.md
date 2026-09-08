# teach -- 有状态多会话教学

## 第一部分：详细总结

### 概述

`teach` 是一个**用户调用（User-invoked）**的 Skill，将你运行它的目录转化为一个**常设教学工作空间**，通过多个会话，以短小独立的自包含 HTML 课程来教授一个主题。

它不从模型已知的参数知识中教学。参数知识被视为不可信：在授课之前，它会去寻找高可信度资源，记录在 `RESOURCES.md` 中，并在每堂课中引用它们。另一个结构事实是它是**有状态的**：使命、资源、课程以及你已学内容的记录都以文件形式存在于目录中，因此下一个会话从这些文件中恢复，而非从上一次对话剩下的内容恢复。

### 调用方式与触发场景

通过输入 `/teach` 调用，Agent 不会自己触发。当学习本身就是项目时使用：一门语言、框架、刚加入的代码库、瑜伽、着色器、认证考试。它**不是**用来做一次性解释的工具。

| 需求 | 应使用 |
| --- | --- |
| 在数周内学习一个主题，会话间累积 | `teach` |
| 在当前会话中解释一个想法 | 直接在当前会话中提问 |
| Agent 的最后一条消息没理解，需要重新表述 | wait-what |
| 锐化已有的思考，而非获取新材料 | grill-me |
| 后台 Agent 阅读一手素材并留下引用文档 | research |
| 学习 grilling 中冒出的东西，不偏离 grilling | handoff 到教学工作空间，然后 teach |

### 前提条件与工作空间结构

`teach` 构建一个目录而非生成一个文件。该 Skill 假设每个工作空间一个使命，因此在一个你愿意交给单一主题的位置运行它。不要放在你工作的项目中：推荐使用独立仓库而非全局 `~/.learnings/` 文件夹或工作项目本身。独立仓库也使得课程可提交，这是团队分享课程的方式。

工作空间中累积的内容：

| 路径 | 内容 |
| --- | --- |
| `MISSION.md` | 你为什么要学这个。一切都挂在这上面；如果缺少它，teach 要做的第一件事就是访谈到它不再缺失 |
| `RESOURCES.md` | 经过审查的教学来源，分为知识（Knowledge）和智慧/社区（Wisdom） |
| `lessons/*.html` | 编号课程：教学的主要单元 |
| `reference/*.html` | 压缩的备忘单、算法、术语表：你实际会回来查阅的文档 |
| `learning-records/*.md` | 关于你已可证明学会的内容的 ADR 风格笔记，用于决定下一步教什么 |
| `assets/*` | 可复用组件，从共享样式表开始，使课程看起来像一个课程体系 |
| `NOTES.md` | 你声明的教学偏好 |

### 核心理念：存储强度与流畅性

关键思维用词是**存储强度（Storage Strength）**：长期记忆的保持，相对于**流畅性（Fluency）**——阅读时感觉像精通但一周后就消失的即时回忆。`teach` 通过合意困难（Desirable Difficulty）构建前者：检索练习、间隔学习、交错学习。**知识**先行（此时困难是敌人，因为它消耗你理解所需的工作记忆），然后通过紧密的反馈循环**训练技能**（此时困难是工具）。

两个因素引导教什么。**使命**（你想要的具体的真实世界原因）使每堂课有根基；没有它，课程会变得抽象，且没有什么能决定接下来教什么。从使命和学习记录中，`teach` 选择**最近发展区（Zone of Proximal Development）**内的下一课：足够有挑战性需要努力，但又不至于远到无法学会。

这也是为什么该 Skill 会推回而非顺从。一个需要**智慧**（真实世界判断力）的问题会得到尝试性回答，然后是指向一个你可以测试它的社区的指针。测验是一个门，而非形式：有用户报告说了"非常感谢"后被告诉训练仍在进行中。

### 课程、参考文献和组件

**课程（Lesson）**是一个自包含的 HTML 文件，短到可以一次学完，与使命绑定，提供一个切实的收获。它引用其来源，推荐一个一手来源让你自己去阅读，并链接到兄弟课程和参考文献。

值得知道的区分：课程很少被重温，参考文献则经常。因此课程的压缩精华（语法表、算法、姿势序列、术语表）属于 `reference/`，而非埋没在介绍它们的课程中。

课程由 `assets/` 中的**组件（Components）**构建：样式表、测验小部件、模拟器、图表辅助工具。复用是默认行为。Agent 在编写课程前先读取 `assets/`，从已存在的内容中构建；任何第二堂课也能使用的新内容都写为组件而非内联。共享样式表是每个工作空间获得的第一个组件；它确保输出不是一堆一次性作品。

### 常见问题

- **文件放在了哪里？我的跑到了 `~/.claude/skills` 下。** 一个真实的、尚未修复的 bug（#377）。SKILL.md 对两个不同的根使用了 `./`：`./MISSION-FORMAT.md` 确实位于已安装 Skill 中 SKILL.md 旁边，而 `./lessons/`、`./reference/` 等本应在你的目录中。将第一种路径解析为 Skill 安装目录的 Agent 会继续将第二种也解析到那里。在第一堂课前检查它落在了哪里，明确指定目录名开始。

- **留在同一个会话还是每堂课新开会话？** 三种方式都可行：留在同一会话、在新会话中重新调用 `/teach`、或在工作空间文件夹中打开新会话。每堂课是其自身的调用。文件夹是连续性，不是对话。常见做法是在工作空间中打开新会话并说 `/teach next lesson for <topic>`。

- **怎么知道它教的不是编造的？** 仅凭 Skill 的话你不知道。你阅读一手来源。`teach` 不足以可靠到不加检查就信任，任何基于 LLM 的 Skill 都不行。接地机制（RESOURCES.md、每课引用、每课推荐一手来源）的存在是为了让验证变得廉价，而非消除验证需求。失败不是假设性的：一个学 2x2 魔方的用户得到了无法解出魔方的捏造动作序列。

- **正确测验答案总是第一个选项。** 多人确认（Sonnet、Opus、GLM），仍未修复。SKILL.md 现在要求每个答案字数相同，但这解决了不同的暗示（正确答案曾是唯一有完整推理的那个），对位置却未提及。一个贡献者测试了位置层面的指令修复，报告正确答案在 9 课 33 次测验中仍 33 次落在 A 位置，指向 `assets/` 中的洗牌测验组件才是真正的修复。

- **它假设我已经知道了一些东西，并使用了从未定义的术语。** 最常见的实质性投诉。没有评估步骤：`teach` 从使命和学习记录推断你的水平，而在第零次会话中没有学习记录。两条帮助：在第一条消息中陈述你的先验知识和盲区，并在课程偏离时大声纠正水平——因为纠正会成为学习记录并引导下一课。显式知识评估步骤是一个长期的功能请求（#725）。

- **它做间隔重复吗？它知道何时停止教学吗？** 第一个不，第二个不总是可靠。间隔和交错是课程设计的原则，但没有东西安排复习，没有 Anki 或日历集成。相关的缺口是退出标准：正如一个用户所说，teach"擅长制作下一课，但不那么擅长知道何时停止并切换到复习或真实实践"。如果你想要复习或训练而非新材料，直接要求；该 Skill 不会自己提出切换。

- **只对编程有用吗？** 不，非编程用途是记录的更大部分：韩语、日语敬语、钢琴、吉他、桌游设计、OpenSCAD、电影情节、Azure 和 CCNA 认证、大学考试，以及给 8 岁和 10 岁孩子做的密室逃脱和火蝾螈可打印书籍。Skill 中没有任何内容是编程特定的。在编程中，最强的报告用途不是从零学习一门语言，而是在不熟悉的代码库或新团队的栈中获得方向。

- **用哪个模型运行？** 没有标准答案，报告的差异很大。更高的推理努力度已被报告能产生明显更好的课程。一个用户通过 Copilot CLI 用 Codex 运行同一 Skill，得到了一张 30 行的 HTML 卡片，而 Claude Code 生成了完整课程。

### 判断是否正常工作的标志

- 在空目录中做的第一件事是访谈你为什么想要这个，而非生成课程
- RESOURCES.md 在课程之前填满，每堂课指出一个值得自己去读的一手来源
- 课程中的断言携带外部链接。没有引用的课程是 Skill 凭记忆教学
- 一堂课一次学完，让你能做一件以前不会的事
- 在文件夹中打开新会话并说"next lesson"会继续课程而非重新开始
- learning-records/ 增长，课程停止重新教授你已证明掌握的内容
- 课程看起来像一个课程体系：它们链接 assets/ 中的样式表而非各自携带
- 需要判断力的问题会指向论坛、subreddit 或课程，而不仅是一个答案

### 在 Skill 体系中的位置

`teach` 是一个**随时可用的独立 Skill**。它不在构建链中，不与工程流程共享产物；它拥有自己的目录并在主题持续期间一直存在。它唯一的真正邻居是 handoff，通过组合用法：如果你在 grilling 中被问到不懂的东西，不要停止 grilling 去学——用 `/handoff` 交接给教学工作空间，在那里用 `/teach` 学习，然后回来继续。附近的替代方案是 research，用于你想要一份引用文档而非课程和记忆保留时。

---

## 第二部分：原文与翻译

<!--原文-->
> `teach` turns the directory you run it in into a standing teaching workspace and teaches you one topic across many sessions, in short self-contained HTML lessons.

<!--翻译-->
> `teach` 将你运行它的目录转化为一个常设教学工作空间，通过多个会话，以短小独立的自包含 HTML 课程来教授一个主题。

<!--原文-->
> It does not teach from what the model already knows. Parametric knowledge is treated as untrusted: before it teaches, it goes and finds high-trust resources, records them in `RESOURCES.md`, and cites them inside every lesson. The other structural fact is that it is stateful: the mission, the resources, the lessons and the record of what you have learned all live in the directory as files, so the next session picks up from those files rather than from whatever is left of the last conversation.

<!--翻译-->
> 它不从模型已知的内容中教学。参数知识被视为不可信：在教学之前，它会去寻找高可信度资源，记录在 `RESOURCES.md` 中，并在每堂课中引用它们。另一个结构事实是它是**有状态的**：使命、资源、课程以及你已学内容的记录都以文件形式存在于目录中，因此下一个会话从这些文件中恢复，而非从上一次对话剩下的内容恢复。

<!--原文-->
> Reach for it when the learning is the project: a language, a framework, a codebase you have just joined, yoga, shaders, a certification. It is not the tool for one explanation in passing.

<!--翻译-->
> 当学习本身就是项目时使用它：一门语言、框架、刚加入的代码库、瑜伽、着色器、认证考试。它不是用来做一次性解释的工具。

<!--原文-->
> What accumulates in that directory: `MISSION.md` -- Why you are learning this. Everything else hangs off it; if it is missing, the first thing `teach` does is interview you until it isn't. `RESOURCES.md` -- The vetted sources it teaches from, split into Knowledge and Wisdom (communities). `lessons/*.html` -- The numbered lessons: the primary unit of teaching. `reference/*.html` -- Compressed cheat-sheets, algorithms, glossaries: the documents you actually return to. `learning-records/*.md` -- ADR-style notes on what you have demonstrably learned, used to decide what to teach next. `assets/*` -- Reusable components, starting with a shared stylesheet, so the lessons look like one course. `NOTES.md` -- Your stated teaching preferences.

<!--翻译-->
> 该目录中累积的内容：`MISSION.md`——你为什么要学这个。一切都挂在这上面；如果缺少它，`teach` 要做的第一件事就是访谈到它不再缺失。`RESOURCES.md`——经过审查的教学来源，分为知识（Knowledge）和智慧/社区（Wisdom）。`lessons/*.html`——编号课程：教学的主要单元。`reference/*.html`——压缩的备忘单、算法、术语表：你实际会回来查阅的文档。`learning-records/*.md`——关于你已可证明学会的内容的 ADR 风格笔记，用于决定下一步教什么。`assets/*`——可复用组件，从共享样式表开始，使课程看起来像一个课程体系。`NOTES.md`——你声明的教学偏好。

<!--原文-->
> The word to think with is **storage strength**: long-term retention, as opposed to **fluency**, the in-the-moment recall that feels like mastery while you are reading and is gone a week later. `teach` builds the former through desirable difficulty: retrieval practice, spacing, interleaving. Knowledge comes first, where difficulty is the enemy because it eats the working memory you need in order to understand; then the skill is drilled through a tight feedback loop, where difficulty is the tool.

<!--翻译-->
> 关键思维用词是**存储强度**：长期记忆的保持，相对于**流畅性**——阅读时感觉像精通但一周后就消失的即时回忆。`teach` 通过合意困难构建前者：检索练习、间隔学习、交错学习。知识先行（此时困难是敌人，因为它消耗你理解所需的工作记忆），然后通过紧密的反馈循环训练技能（此时困难是工具）。

<!--原文-->
> Two things steer what you get taught. The **mission** (the concrete real-world reason you want this) grounds every lesson; without it the lessons drift abstract and nothing decides what comes next. From the mission and the learning records, `teach` picks the next lesson inside your **zone of proximal development**: challenging enough to take effort, not so far ahead that it stops being learnable.

<!--翻译-->
> 两个因素引导你被教什么。**使命**（你想要的具体的真实世界原因）使每堂课有根基；没有它，课程会变得抽象，且没有什么能决定接下来教什么。从使命和学习记录中，`teach` 选择**最近发展区**内的下一课：足够有挑战性需要努力，但又不至于远到无法学会。

<!--原文-->
> A **lesson** is one self-contained HTML file, short enough to finish in a sitting, tied to the mission, giving one tangible win. It cites its sources, recommends one primary source to go and read yourself, and links to sibling lessons and reference documents. The split worth knowing: lessons are rarely revisited, reference documents are. So the compressed essence of a lesson (the syntax table, the algorithm, the pose sequence, the glossary) belongs in `reference/`, not buried in the lesson that introduced it.

<!--翻译-->
> **课程**是一个自包含的 HTML 文件，短到可以一次学完，与使命绑定，提供一个切实的收获。它引用其来源，推荐一个一手来源让你自己去阅读，并链接到兄弟课程和参考文献。值得知道的区分：课程很少被重温，参考文献则经常。因此课程的压缩精华（语法表、算法、姿势序列、术语表）属于 `reference/`，而非埋没在介绍它们的课程中。

<!--原文-->
> Lessons are built from **components** in `assets/`: stylesheets, quiz widgets, simulators, diagram helpers. Reuse is the default. The agent reads `assets/` before authoring a lesson and builds from what is there, and anything new that a second lesson could use is written as a component rather than inlined. The shared stylesheet is the first component every workspace earns; it is what stops the output being a pile of one-offs.

<!--翻译-->
> 课程由 `assets/` 中的**组件**构建：样式表、测验小部件、模拟器、图表辅助工具。复用是默认行为。Agent 在编写课程前先读取 `assets/`，从已存在的内容中构建；任何第二堂课也能使用的新内容都写为组件而非内联。共享样式表是每个工作空间获得的第一个组件；它确保输出不是一堆一次性作品。

<!--原文-->
> You don't, on the skill's word alone. You read the primary sources. `teach` is not reliable enough to trust unchecked, and no skill built on an LLM is. The grounding machinery (`RESOURCES.md`, citations in every lesson, one recommended primary source per lesson) exists to make verification cheap, not to remove the need for it.

<!--翻译-->
> 仅凭 Skill 的话你无法知道。你阅读一手来源。`teach` 不足以可靠到不加检查就信任，任何基于 LLM 的 Skill 都不行。接地机制（`RESOURCES.md`、每课引用、每课推荐一手来源）的存在是为了让验证变得廉价，而非消除验证需求。

<!--原文-->
> The correct quiz answer is always the first option. Confirmed by several people, on Sonnet, on Opus and on GLM, and still unfixed. One contributor tested an instruction-level fix for position and reported the correct answer still landing in slot A 33 times out of 33 across nine lessons, which points at a shuffling quiz component in `assets/` as the real fix rather than better wording.

<!--翻译-->
> 正确测验答案总是第一个选项。多人确认（Sonnet、Opus、GLM），仍未修复。一个贡献者测试了位置层面的指令修复，报告正确答案在 9 课 33 次测验中仍 33 次落在 A 位置，指向 `assets/` 中的洗牌测验组件才是真正的修复，而非更好的措辞。

<!--原文-->
> It assumed I already knew things, and used terms it never defined. The commonest substantive complaint. There is no assessment step: `teach` infers your level from the mission and the learning records, and in session one there are no learning records. Two things help: state your prior knowledge and your gaps in the first message, and correct the level out loud when a lesson misses, because the correction becomes a learning record and steers the next one.

<!--翻译-->
> 它假设我已经知道了一些东西，并使用了从未定义的术语。最常见的实质性投诉。没有评估步骤：`teach` 从使命和学习记录推断你的水平，而在第一次会话中没有学习记录。两条帮助：在第一条消息中陈述你的先验知识和盲区，并在课程偏离时大声纠正水平——因为纠正会成为学习记录并引导下一课。

<!--原文-->
> Is it only useful for code? No, and the non-coding use is the larger part of the record: Korean, Japanese formal register, piano, guitar, board game design, OpenSCAD, film plots, Azure and CCNA certifications, university exams, and children of eight and ten getting printable books on escape rooms and fire salamanders. Nothing in the skill is programming-specific: mission, resources, zone of proximal development and drill work the same way in any domain.

<!--翻译-->
> 只对编程有用吗？不，非编程用途是记录的更大部分：韩语、日语敬语、钢琴、吉他、桌游设计、OpenSCAD、电影情节、Azure 和 CCNA 认证、大学考试，以及给 8 岁和 10 岁孩子做的密室逃脱和火蝾螈可打印书籍。Skill 中没有任何内容是编程特定的：使命、资源、最近发展区和训练在任何领域都以相同方式运作。

<!--原文-->
> `teach` is a **reach-for-it-anytime standalone**. It is not a step in a build chain and shares no artifacts with the engineering flow; it owns its directory and lives there for as long as the topic lasts. Its one real neighbour is handoff, through the composition Matt named as the answer to "what do I do if I'm being grilled about something I don't understand?": don't stop the grilling to learn: `/handoff` to a teaching workspace, learn it there with `/teach`, then go back and pick up where you left off.

<!--翻译-->
> `teach` 是一个**随时可用的独立 Skill**。它不在构建链中，不与工程流程共享产物；它拥有自己的目录并在主题持续期间一直存在。它唯一的真正邻居是 handoff，通过 Matt 对"如果我在 grilling 中被问到不懂的东西该怎么办？"的回答所命名的组合用法：不要停止 grilling 去学习——用 `/handoff` 交接给教学工作空间，在那里用 `/teach` 学习，然后回来从断点继续。
