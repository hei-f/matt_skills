# research 技能报告

## 第一部分：技能内容详细总结

### 一、技能概述

`research` 解决一个研究问题，通过让后台 Agent 阅读拥有答案的**一手来源（primary sources）**，然后将附有引用的 Markdown 文件写入仓库。它的核心理念是：研究是你可以委托的阅读跑腿工作，而不是你可以外包的思考工作。你得到的是一份可供审查、引用、交给他人的文档，而不是一个会在会话结束时消失的答案。

该技能仅在**一手来源**（官方文档、源代码、规格说明、第一方 API）上工作，而不是对其的二手总结。它追溯每个声明回到拥有该声明的源头，因此当 API 的官方文档可达时，它不会复述博客文章对该 API 的描述。

### 二、核心机制：后台委托

技能的决定性动作是将阅读工作作为**后台 Agent** 运行。你继续工作；它独自去追踪每个声明回到一手来源，写入一个 Markdown 文件，然后汇报。

后台 Agent 的工作：
1. 针对**一手来源**调查问题（官方文档、源代码、规格说明、第一方 API），而非对它们的二手叙述。将每个声明追溯到拥有该声明的源头。
2. 将发现写入单个 Markdown 文件，为每个声明引用其来源。
3. 将文件保存到仓库已有的笔记保存位置；匹配现有的约定，如果没有约定则放在合理的地方并说明位置。

### 三、一手来源约束

技能命名了合格来源的*种类*（官方文档、源代码、规格说明、第一方 API），但没有信任名单、没有领域门槛、没有验证环节。这是该技能在最初提出时最响亮的反对意见："五个研究子 Agent 指向垃圾信息，只会更快地给你五个自信的错误答案。你如何把关什么才算高信任来源？"

实际的缓解措施是每个声明上的**引用**。追踪两三个引用，如果它们落到了事物的摘要而非事物本身，说明那次运行在其唯一的工作上失败了。

### 四、使用场景与选择

区分 `research` 与类似技能的关键是**返回内容的保质期**：

| 你需要什么 | 应选择的技能 |
|-----------|------------|
| 一个外部事实，决策在等待它 | `research` |
| 通过访谈共同做出的决策 | `grilling` |
| 持久的架构决策，写入 `CONTEXT.md` 和 ADR | `grill-with-docs` |
| 找出某个方法在你的代码库中是否有效 | `prototype` |
| 一个太大无法在一个会话中持有的计划 | `wayfinder` |

`research` 与 `grill-with-docs` 的分界线是返回内容的保质期：研究产出**短命的资产**（例如"这个库本周的认证机制是什么"），而 ADR 记录的是你**保留的决策**。如果正在产出的是一份决策而非一个事实，那就是在访谈而非研究。

### 五、产出物及其管理

- 文件放置位置由仓库决定，不由技能决定：它匹配已存在的笔记存放约定。
- 每次运行写入一个文件。
- 社区共识：ADR 保留，研究文件不保留。研究文件记录的是写入当天为真的事实，因此一份过时的研究文件比没有更糟。

### 六、已知问题

1. **嵌套后台 Agent（issue #530）**：这是该技能文档最完善的已知缺陷。技能告诉调用者启动后台 Agent 但不限制 Agent 类型，导致它生成的 Agent 可能再次触发。有报告称单次任务消耗约 450k token，三个重复运行重叠。修复方案是在本地副本中添加指令告知已是子 Agent 的 Agent 自行完成工作。相反，如果全局指令禁止 Agent 重新委托工作，后台 Agent 会礼貌地拒绝任务。

2. **缺乏停止条件**：技能中没有停止标准。表现为 Agent 深入得过深，或广泛覆盖主题但遗漏关键细节。范围界定取决于你——一个狭窄的、可回答的问题比"研究 X"好得多。

3. **后续会话不自动复用**：没有什么会自动加载过去的研究文件。文件的价值在于被有意识地送入下一步：附加到规格、引用到访谈会话、指向工单。

4. **wayfinder 研究工单的处理**：wayfinder 的未发布变更中会为每个研究工单生成 `/research` 子 Agent 并并行消耗。已知问题：子 Agent 从永不打算合并的分支打开草稿 PR（issue #576），以及删除分支会破坏工单的上下文指针。

### 七、工作判定标准

- 你自己的会话继续运行。如果坐着看它阅读，说明委托没有发生。
- 恰好一个后台任务出现。第二个名称几乎相同的任务是嵌套 bug。
- 一个新的 Markdown 文件出现在仓库已用于笔记的文件夹中，Agent 告诉你路径。
- 文件中的每个声明都带有链接，随机追踪两个会落在官方文档、规格说明或实际源文件上，而非某人的叙述。
- 仅凭该文件就能做出之前卡住的决策，无需自己回到源头。

---

## 第二部分：SKILL.md 原文及翻译

```
<!--原文-->
> ---
> name: research
> description: Investigate a question against high-trust primary sources and capture the findings as a Markdown file in the repo. Use when the user wants a topic researched, docs or API facts gathered, or reading legwork delegated to a background agent.
> ---
```
```
<!--翻译-->
> ---
> name: research
> description: 针对高信任度的一手来源调查一个问题，并将发现捕获为仓库中的 Markdown 文件。当用户想要研究某个主题、收集文档或 API 事实、或将阅读跑腿工作委托给后台 Agent 时使用。
> ---
```

```
<!--原文-->
> Spin up a **background agent** to do the research, so you keep working while it reads.
```
```
<!--翻译-->
> 启动一个**后台 Agent** 来进行研究，这样你可以在它阅读时继续工作。
```

```
<!--原文-->
> Its job:
>
> 1. Investigate the question against **primary sources** (official docs, source code, specs, first-party APIs), not a secondary write-up of them. Follow every claim back to the source that owns it.
> 2. Write the findings to a single Markdown file, citing each claim's source.
> 3. Save it where the repo already keeps such notes; match the existing convention, and if there is none, put it somewhere sensible and say where.
```
```
<!--翻译-->
> 它的工作：
>
> 1. 针对**一手来源**（官方文档、源代码、规格说明、第一方 API）调查问题，而非对它们的二手叙述。将每个声明追溯到拥有该声明的源头。
> 2. 将发现写入单个 Markdown 文件，为每个声明引用其来源。
> 3. 将文件保存到仓库已有的此类笔记保存位置；匹配现有的约定，如果没有约定则放在合理的地方并说明位置。
```
