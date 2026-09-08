# resolving-merge-conflicts 技能报告

## 第一部分：技能内容详细总结

### 一、技能概述

`resolving-merge-conflicts` 是一个独立的、无依赖的冲突解决工具，专门处理 git 合并（merge）或变基（rebase）过程中的冲突。它逐块（hunk by hunk）解决冲突，运行项目自身的自动化检查，并以一次提交完成整个操作。

该技能拒绝将冲突视为文本问题。在触碰任何冲突块之前，它追溯每一方的变更为其**一手来源（primary source）**（提交消息、PR、原始工单），因此始终在两个**意图**之间而非两个文本块之间进行选择。它在双方兼容的地方保留双方意图；在确实不兼容的地方，选择与合并声明目标匹配的一方并明确指出权衡。它**绝不发明新行为**来掩盖冲突，且 `--abort` 不在其选项之内——合并总是被推进到完成提交。

### 二、五步流程

#### 步骤 1：查看当前状态

检查合并/变基的当前状态，包括 git 历史记录和冲突文件。

#### 步骤 2：寻找每个冲突的一手来源

深入理解每个变更是**为什么**做出的，原始意图是什么。阅读提交消息、检查 PR、检查原始工单。不能仅通过对比 diff 块来判断——语法上完美的解决可能仍然静默地丢弃某人有意做出的更改。

#### 步骤 3：解决每个冲突块

- 在可能的地方**保留双方的意图**。
- 在双方不兼容时，**选择与合并声明目标匹配的一方**，并明确指出所做的权衡。
- **绝不发明新行为**。
- 始终坚持解决，**绝不 `--abort`**。

#### 步骤 4：发现并运行自动检查

找到项目自身的自动化检查并运行它们。典型顺序是：类型检查（typecheck）→ 测试（tests）→ 格式化（format）。修复合并造成的任何破坏。合并是 git 中最容易产生虽满足两个分支语法但无法通过任一方测试的代码的地方。

#### 步骤 5：完成合并/变基

暂存所有内容并提交。如果是变基，继续变基过程，直到所有提交都被变基完成。

### 三、设计哲学

#### 一手来源优于 `--ours` 和 `--theirs`

该技能旨在消除的失败模式是**按标志解决**：`--ours`、`--theirs`，或手动删除看起来不太重要的那一块，使冲突标记消失并让构建通过。这种解决可能在语法上完美，但仍然静默地丢弃了某人有意做出的更改。你无法保留你没有阅读过的意图，因此工作从历史（提交、PR、工单）开始，然后才进入 diff。

#### 永远不 `--abort`

放弃（abort）扔掉解决工作，并在下次尝试时回到相同的、未改变的冲突。技能是为"合并将要发生"的情况编写的。如果你已决定不应发生合并，那是在调用之前做出的决定，而不是循环内的一个分支。

### 四、使用场景

| 当前状况 | 应选择的技能 |
|---------|------------|
| 合并或变基中途，树中有冲突标记 | 本技能 |
| 合并已完成，现在某些行为出现异常 | `diagnosing-bugs` |
| 规划如何切片工作以减少分支碰撞 | 两者都不适用 |

#### 并行工作与冲突避免

对于"是否应该将并行 Agent 远离相同文件以避免冲突"的问题，回答是**大部分情况下不应该**。按任务划分文件区域的成本高于收益。一个值得遵守的原则：**先做大型重构**——一个在十个分支都已分叉之后落地的大型重命名是唯一保持昂贵的案例。

来自并行工作树用户报告的注意事项：当兄弟会话各自在其自己的树中构建一个工单时，合并回主干的冲突最好由编写了变更的那个会话来处理，因为它是唯一已经知道意图的会话。

### 五、与 Claude Code 自身能力的对比

Claude Code 本身就能相当好地解决冲突。该技能的附加值在于"寻找一手来源"和"运行反馈循环"两个步骤——这些步骤在没有技能的情况下需要每次手动提示。一个未经提示的 Agent 通常会仅从 diff 中产生一个看似合理的解决方案并就此停止。技能的价值在于它不会让 Agent 跳过的两个步骤：阅读每一方存在的原因，以及事后运行检查。

### 六、工作判定标准

- Agent 在解决冲突时引用的是提交消息、PR 或工单，而不仅仅是 diff 块。
- 每个冲突块最终包含双方的行为，或明确的注释说明舍弃了什么以及为什么。
- 结果中没有任何东西是来自两个分支之外的。
- 类型检查、测试和格式化在提交**之前**被定位并运行通过，而不是在你注意到某些东西损坏之后。
- 以一个干净的树结束，操作已完成，包括多提交变基中每个剩余的提交。

---

## 第二部分：SKILL.md 原文及翻译

```
<!--原文-->
> ---
> name: resolving-merge-conflicts
> description: "Use when you need to resolve an in-progress git merge/rebase conflict."
> ---
```
```
<!--翻译-->
> ---
> name: resolving-merge-conflicts
> description: "当你需要解决正在进行的 git merge/rebase 冲突时使用。"
> ---
```

```
<!--原文-->
> 1. **See the current state** of the merge/rebase. Check git history, and the conflicting files.
```
```
<!--翻译-->
> 1. **查看合并/变基的当前状态。** 检查 git 历史记录和冲突文件。
```

```
<!--原文-->
> 2. **Find the primary sources** for each conflict. Understand deeply why each change was made, and what the original intent was. Read the commit messages, check the PRs, check original issues/tickets.
```
```
<!--翻译-->
> 2. **为每个冲突寻找一手来源。** 深入理解每个变更是为什么做出的，原始意图是什么。阅读提交消息，检查 PR，检查原始工单/工单。
```

```
<!--原文-->
> 3. **Resolve each hunk.** Preserve both intents where possible. Where incompatible, pick the one matching the merge's stated goal and note the trade-off. Do **not** invent new behaviour. Always resolve; never `--abort`.
```
```
<!--翻译-->
> 3. **解决每个冲突块。** 在可能的地方保留双方的意图。在双方不兼容时，选择与合并的声明目标匹配的一方并明确指出权衡。**不要**发明新行为。始终坚持解决；绝不 `--abort`。
```

```
<!--原文-->
> 4. Discover the project's **automated checks** and run them, typically typecheck, then tests, then format. Fix anything the merge broke.
```
```
<!--翻译-->
> 4. 发现项目自身的**自动化检查**并运行它们，典型顺序是类型检查、测试，最后是格式化。修复合并造成的任何破坏。
```

```
<!--原文-->
> 5. **Finish the merge/rebase.** Stage everything and commit. If rebasing, continue the rebase process until all commits are rebased.
```
```
<!--翻译-->
> 5. **完成合并/变基。** 暂存所有内容并提交。如果是变基，继续变基过程直到所有提交都被变基完成。
```
