# improve-codebase-architecture 技能报告

## 第一部分：技能内容详细总结

### 一、技能概述

`improve-codebase-architecture` 是一个**周期性维护工具**，扫描代码库寻找**深化机会（deepening opportunities）**——即将浅层模块（接口几乎与实现一样复杂）改造为深层模块（大量行为隐藏在小型稳定接口之后）的重构方向。它以可视化 HTML 报告呈现候选方案，然后对你选择的方案进行访谈式深入探讨。

该技能**从不修改代码**。整个运行只产出一个临时目录中的 HTML 文件和一个对话；实际的重构工作发生在之后的独立会话中，通过正常的构建流程进行。

### 二、核心概念：深度（Depth）

技能围绕一个核心思想运转：**深度**。

- **深层模块**：将大量行为隐藏在小型、稳定的接口之后。
- **浅层模块**：通过几乎与底层代码一样宽的接口泄漏其实现细节。

报告从三种形式寻找浅层性：
1. **缺乏局部性（locality）**：仅为可测试性而提取的纯函数，真正的 bug 隐藏在它们的调用方式中。
2. **接缝泄漏**：紧耦合的模块跨越其接缝互相泄漏。
3. **跨文件理解**：理解一个概念需要打开五个文件。

每个候选方案必须通过**删除测试（deletion test）**：删除这个模块会集中复杂性（好的信号），还是仅仅将其分散到调用者？只有"集中"的情况才能进入报告。

### 三、设计词汇体系

技能使用一套来自 `/codebase-design` 的共享设计词汇：

- **模块（module）**、**接口（interface）**、**深度（depth）**、**接缝（seam）**、**适配器（adapter）**、**杠杆（leverage）**、**局部性（locality）**
- 设计原则：删除测试、"接口即测试面"、"一个适配器=假设的接缝，两个=真实的"
- 领域语言来自 `GLOSSARY.md`：如果术语表中定义了"Order"，则谈论"订单接收模块"而非"FooBarHandler"

### 四、三阶段流程

#### 阶段一：探索（Explore）

**先定范围再扫描（YAGNI）**。对最近变更频繁的部分给予额外权重：

- 如果用户指明了方向（模块、子系统、痛点），直接采纳。
- 否则，回溯大量提交历史（`git log --oneline`）寻找热点文件。
- 先阅读项目的 `GLOSSARY.md` 和相关区域的 ADR。
- 然后启动一个子 Agent 遍历代码库，有机地探索并记录摩擦点。

#### 阶段二：呈现候选方案（HTML 报告）

将自包含的 HTML 文件写入操作系统临时目录，不在仓库中留下文件。每张卡片包含：涉及的文件、问题描述、简明解决方案、收益解释（局部性和杠杆）、前后对比图示、推荐强度徽章。

报告使用 **Tailwind CDN** 进行布局和样式，使用 **Mermaid CDN** 绘制关系图。报告末尾包含一个**最佳推荐（Top recommendation）**部分。

推荐强度含义：

| 徽章 | 含义 |
|-----|------|
| `Strong` | 删除测试明确通过，摩擦真实存在。应认真对待。 |
| `Worth exploring` | 深化合理，但收益取决于代码的未来走向。 |
| `Speculative` | 为完整性而列出。大部分可以安全忽略。 |

**ADR 冲突处理**：如果候选方案与现有 ADR 矛盾，仅在实际摩擦足以值得重新审视 ADR 时才提及，并明确标注。

#### 阶段三：访谈循环（Grilling Loop）

用户选定一个候选方案后，调用 `grilling` 技能遍历决策树。副作用在决策明确时即时发生，调用 `domain-modeling` 保持领域模型更新：

- 为深化模块命名了新概念？添加到 `GLOSSARY.md`。
- 对话中澄清了一个模糊术语？立即更新 `GLOSSARY.md`。
- 用户以充分理由拒绝了候选方案？提议记录为 ADR。
- 想探索深化模块的替代接口？调用 `codebase-design` 的 design-it-twice 模式。

### 五、使用场景

四个典型使用场景：

| 场景 | 使用方式 |
|------|---------|
| 日常维护 | 每几天运行一次，防止架构在功能迭代间腐化 |
| 大型构建之前 | 指向 spec："我们如何让这个变更变得容易？" |
| 遗留代码审计 | 对大型、无结构的仓库运行，了解其真实形态 |
| 为遗留代码编写测试 | 先找到缺失的接缝，再为不可测试的代码写测试 |

### 六、已知问题

1. **访谈过度**：较弱的模型会跳过报告直接开始访谈。解决方法是在调用时明确说明"不要访谈，只展示报告"。
2. **HTML 报告无法渲染**：报告依赖 CDN，需要网络访问。离线环境会导致显示为无样式的原始 HTML。
3. **大型遗留代码库效果有限**：建议先用 `grill-with-docs` 建立共享词汇。
4. **TypeScript 实现指导缺失**：技能会告诉你深化应该在哪里进行，但如何转化为包或目录结构需要你自己完成。

### 七、工作判定标准

- 候选方案使用你的领域概念命名，而非编造的类名。
- 候选方案聚集在你近期编辑的文件中，而非代码库的死角。
- 运行期间没有代码变更，唯一的文件是临时目录中的 HTML 报告。
- 它在报告后停下来询问你想要哪个候选方案，而非自行继续。
- 每张卡片以局部性或杠杆来解释收益，并说明哪些测试变得更简单。

---

## 第二部分：SKILL.md 原文及翻译

```
<!--原文-->
> ---
> name: improve-codebase-architecture
> description: Scan a codebase for deepening opportunities, present them as a visual HTML report, then grill through whichever one you pick.
> disable-model-invocation: true
> ---
```
```
<!--翻译-->
> ---
> name: improve-codebase-architecture
> description: 扫描代码库寻找深化机会，以可视化 HTML 报告呈现，然后对你选择的方案进行访谈式深入探讨。
> disable-model-invocation: true
> ---
```

```
<!--原文-->
> # Improve Codebase Architecture
>
> Surface architectural friction and propose **deepening opportunities**: refactors that turn shallow modules into deep ones. The aim is testability and AI-navigability.
```
```
<!--翻译-->
> # 改进代码库架构
>
> 揭示架构摩擦并提出**深化机会**：将浅层模块改造为深层模块的重构方向。目标是可测试性和 AI 可导航性。
```

```
<!--原文-->
> This command is _informed_ by the project's domain model and built on a shared design vocabulary:
>
> - Call the Skill tool with "codebase-design" for the architecture vocabulary (**module**, **interface**, **depth**, **seam**, **adapter**, **leverage**, **locality**) and its principles (the deletion test, "the interface is the test surface", "one adapter = hypothetical seam, two = real"). Use these terms exactly in every suggestion, and don't drift into "component," "service," "API," or "boundary."
> - The domain language in `GLOSSARY.md` gives names to good seams; ADRs in `docs/adr/` record decisions this command should not re-litigate.
```
```
<!--翻译-->
> 此命令由项目的领域模型提供信息，并构建在共享的设计词汇之上：
>
> - 调用 Skill 工具使用 "codebase-design" 获取架构词汇（**模块**、**接口**、**深度**、**接缝**、**适配器**、**杠杆**、**局部性**）及其原则（删除测试、"接口即测试面"、"一个适配器=假设的接缝，两个=真实的"）。在每条建议中精确使用这些术语，不要滑向"组件"、"服务"、"API"或"边界"等词汇。
> - `GLOSSARY.md` 中的领域语言为好的接缝命名；`docs/adr/` 中的 ADR 记录了此命令不应重新争论的决策。
```

```
<!--原文-->
> ## Process
>
> ### 1. Explore
>
> **Scope before you scan: YAGNI.** Deepening a module pays off by making future changes to it easier, so put extra weight on the parts of the codebase that have recently changed. Decide *where* to look before you look:
>
> - If the user named a direction (a module, a subsystem, a pain point), take it, and skip the inference below.
> - Otherwise, walk back a good stretch of the commit history (`git log --oneline`) to find the codebase's hot spots, the files and areas that keep coming up, and let those paths pull your attention first. If the changes are scattered with no clear hot spot, widen the net.
```
```
<!--翻译-->
> ## 流程
>
> ### 1. 探索
>
> **先定范围再扫描：YAGNI。** 深化模块的回报在于使未来对其的变更更容易，因此对代码库中近期变更的部分给予额外权重。在查看之前先决定*在哪里*查看：
>
> - 如果用户指明了方向（一个模块、一个子系统、一个痛点），直接采纳，跳过以下推断。
> - 否则，回溯足够长的提交历史（`git log --oneline`）来找到代码库的热点——那些反复出现的文件和区域——让这些路径优先吸引你的注意力。如果变更分散没有明确的热点，扩大搜索范围。
```

```
<!--原文-->
> Read the project's domain glossary (`GLOSSARY.md`) and any ADRs in the area you're touching first.
>
> Then spawn a sub-agent to walk the codebase. Don't follow rigid heuristics; explore organically and note where you experience friction:
>
> - Where does understanding one concept require bouncing between many small modules?
> - Where are modules **shallow**, with an interface nearly as complex as the implementation?
> - Where have pure functions been extracted just for testability, but the real bugs hide in how they're called (no **locality**)?
> - Where do tightly-coupled modules leak across their seams?
> - Which parts of the codebase are untested, or hard to test through their current interface?
```
```
<!--翻译-->
> 首先阅读项目的领域术语表（`GLOSSARY.md`）和你要接触的区域中任何相关的 ADR。
>
> 然后启动一个子 Agent 遍历代码库。不要遵循僵化的启发式规则；有机地探索并记录你在何处遇到摩擦：
>
> - 在何处理解一个概念需要在许多小模块之间跳转？
> - 哪些模块是**浅层**的，其接口几乎与实现一样复杂？
> - 哪些纯函数仅为可测试性而提取，但真正的 bug 隐藏在它们的调用方式中（缺乏**局部性**）？
> - 哪些紧耦合的模块跨越其接缝互相泄漏？
> - 代码库的哪些部分未经测试，或难以通过其当前接口进行测试？
```

```
<!--原文-->
> Apply the **deletion test** to anything you suspect is shallow: would deleting it concentrate complexity, or just move it? A "yes, concentrates" is the signal you want.
```
```
<!--翻译-->
> 对你怀疑是浅层的任何东西应用**删除测试**：删除它会集中复杂性，还是仅仅转移它？"是的，会集中"就是你想要的信号。
```

```
<!--原文-->
> ### 2. Present candidates as an HTML report
>
> Write a self-contained HTML file to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows), and write to `<tmpdir>/architecture-review-<timestamp>.html` so each run gets a fresh file. Open it for the user (`xdg-open <path>` on Linux, `open <path>` on macOS, `start <path>` on Windows) and tell them the absolute path.
```
```
<!--翻译-->
> ### 2. 以 HTML 报告呈现候选方案
>
> 将自包含的 HTML 文件写入操作系统临时目录，确保仓库中不留下任何文件。从 `$TMPDIR` 解析临时目录，回退到 `/tmp`（Windows 上为 `%TEMP%`），写入 `<tmpdir>/architecture-review-<timestamp>.html` 使每次运行获得一个新文件。为用户打开它（Linux 上 `xdg-open <path>`，macOS 上 `open <path>`，Windows 上 `start <path>`）并告知他们绝对路径。
```

```
<!--原文-->
> The report uses **Tailwind via CDN** for layout and styling, and **Mermaid via CDN** for diagrams where a graph/flow/sequence reliably communicates the structure. Mix Mermaid with hand-crafted CSS/SVG visuals: use Mermaid when relationships are graph-shaped (call graphs, dependencies, sequences), and hand-built divs/SVG when you want something more editorial (mass diagrams, cross-sections, collapse animations). Each candidate gets a **before/after visualisation**. Be visual.
```
```
<!--翻译-->
> 报告使用通过 CDN 引入的 **Tailwind** 进行布局和样式，使用通过 CDN 引入的 **Mermaid** 在图形/流程/时序能可靠传达结构的地方绘制图表。将 Mermaid 与手工制作的 CSS/SVG 视觉效果混合使用：当关系是图形状的（调用图、依赖关系、时序）使用 Mermaid，当你想要更具社论性的内容（质量图、剖面图、折叠动画）时使用手工构建的 div/SVG。每个候选方案都有一个**前后对比可视化**。要注重视觉呈现。
```

```
<!--原文-->
> For each candidate, render a card with:
>
> - **Files**: which files/modules are involved
> - **Problem**: why the current architecture is causing friction
> - **Solution**: plain English description of what would change
> - **Benefits**: explained in terms of locality and leverage, and how tests would improve
> - **Before / After diagram**: side-by-side, custom-drawn, illustrating the shallowness and the deepening
> - **Recommendation strength**: one of `Strong`, `Worth exploring`, `Speculative`, rendered as a badge
```
```
<!--翻译-->
> 为每个候选方案渲染一张卡片，包含：
>
> - **文件**：涉及哪些文件/模块
> - **问题**：当前架构为何造成摩擦
> - **解决方案**：用简明英语描述变更内容
> - **收益**：以局部性和杠杆来解释，以及测试将如何改进
> - **前后对比图**：并排的自定义图示，说明浅层性和深化效果
> - **推荐强度**：`Strong`（强烈推荐）、`Worth exploring`（值得探索）、`Speculative`（推测性）之一，渲染为徽章
```

```
<!--原文-->
> End the report with a **Top recommendation** section: which candidate you'd tackle first and why.
>
> **Use GLOSSARY.md vocabulary for the domain, and the `/codebase-design` vocabulary for the architecture.** If `GLOSSARY.md` defines "Order," talk about "the Order intake module," not "the FooBarHandler," and not "the Order service."
```
```
<!--翻译-->
> 报告以**最佳推荐**部分结束：你会首先处理哪个候选方案以及原因。
>
> **使用 GLOSSARY.md 的词汇处理领域，使用 `/codebase-design` 的词汇处理架构。** 如果 `GLOSSARY.md` 定义了 "Order"，就谈论"订单接收模块"，而非 "FooBarHandler"，也非"订单服务"。
```

```
<!--原文-->
> **ADR conflicts**: if a candidate contradicts an existing ADR, only surface it when the friction is real enough to warrant revisiting the ADR. Mark it clearly in the card (e.g. a warning callout: _"contradicts ADR-0007, but worth reopening because…"_). Don't list every theoretical refactor an ADR forbids.
```
```
<!--翻译-->
> **ADR 冲突**：如果候选方案与现有 ADR 矛盾，仅在实际摩擦足够真实、值得重新审视 ADR 时才提及。在卡片中清晰标注（例如警告提示：_"与 ADR-0007 矛盾，但值得重新讨论因为……"_）。不要列出 ADR 禁止的每一个理论上的重构。
```

```
<!--原文-->
> See [HTML-REPORT.md](HTML-REPORT.md) for the full HTML scaffold, diagram patterns, and styling guidance.
```
```
<!--翻译-->
> 完整的 HTML 脚手架、图表模式和样式指南请参阅 [HTML-REPORT.md](HTML-REPORT.md)。
```

```
<!--原文-->
> Do NOT propose interfaces yet. After the file is written, ask the user: "Which of these would you like to explore?"
```
```
<!--翻译-->
> 暂时不要提出接口。文件写入后，询问用户："你想探索哪一个？"
```

```
<!--原文-->
> ### 3. Grilling loop
>
> Once the user picks a candidate, call the Skill tool with "grilling" to walk the decision tree with them: constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive.
>
> Side effects happen inline as decisions crystallize; call the Skill tool with "domain-modeling" to keep the domain model current as you go:
>
> - **Naming a deepened module after a concept not in `GLOSSARY.md`?** Add the term to `GLOSSARY.md`. Create the file lazily if it doesn't exist.
> - **Sharpening a fuzzy term during the conversation?** Update `GLOSSARY.md` right there.
> - **User rejects the candidate with a load-bearing reason?** Offer an ADR, framed as: _"Want me to record this as an ADR so future architecture reviews don't re-suggest it?"_ Only offer when the reason would actually be needed by a future explorer to avoid re-suggesting the same thing; skip ephemeral reasons ("not worth it right now") and self-evident ones.
> - **Want to explore alternative interfaces for the deepened module?** Call the Skill tool with "codebase-design" and use its design-it-twice parallel sub-agent pattern.
```
```
<!--翻译-->
> ### 3. 访谈循环
>
> 一旦用户选定一个候选方案，调用 Skill 工具使用 "grilling" 与他们一起遍历决策树：约束条件、依赖关系、深化模块的形态、接缝背后的内容、哪些测试能存活。
>
> 副作用在决策明确时即时发生；调用 Skill 工具使用 "domain-modeling" 在过程中保持领域模型最新：
>
> - **为深化模块以 `GLOSSARY.md` 中不存在的概念命名？** 将该术语添加到 `GLOSSARY.md`。如果文件不存在则懒创建它。
> - **在对话中澄清了一个模糊术语？** 立即更新 `GLOSSARY.md`。
> - **用户以充分理由拒绝了候选方案？** 提供 ADR，措辞为：_"需要我将此记录为 ADR 以便未来的架构审查不会重新建议它吗？"_ 仅在理由确实会被未来探索者需要以避免重新建议同一事物时才提供；跳过短暂的（"现在不值得"）和不言自明的理由。
> - **想探索深化模块的替代接口？** 调用 Skill 工具使用 "codebase-design" 并使用其 design-it-twice 并行子 Agent 模式。
```
