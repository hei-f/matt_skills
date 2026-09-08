# diagnosing-bugs — 纪律性 Bug 诊断循环

## 第一部分：详细总结

### 概述

`diagnosing-bugs` 是一个模型调用的 Skill，为困难 Bug 和性能回归提供**纪律性的六阶段诊断循环**。核心理念：**在没有紧反馈循环之前，拒绝进行任何理论推测**。

### 六阶段流程

#### Phase 1: 构建反馈循环（核心）

这是整个 Skill 的灵魂。构建一个对当前 Bug **变红**的、紧致的通过/失败信号：

- **10种构建方式**（按优先级）：失败测试 → Curl/HTTP 脚本 → CLI 调用 → 无头浏览器脚本 → 重放捕获的追踪 → 一次性测试框架 → 属性/模糊测试循环 → 二分查找 → 差分循环 → HITL bash 脚本

- **紧致化循环**：加速、增强信号精度、提高确定性。2 秒确定循环是调试超能力；30 秒不稳定循环勉强可用。

- **非确定性 Bug**：目标不是干净重现，而是**更高的复现率**。

- **完成标准**：你能指出**一个命令**，它已经运行过至少一次，能够驱动实际 Bug 代码路径并断言用户的精确症状，是确定的、快速的、Agent 可运行的。在此之前绝不进入 Phase 2。

#### Phase 2: 重现 + 最小化

运行循环确认复现用户描述的故障模式。然后**最小化**——将复现场景缩小到仍然变红的最小场景，每次只削减一个元素，保留每个对故障**承载性（load-bearing）**的元素。

#### Phase 3: 假设

生成 **3-5 个排序假设**，每个必须是**可证伪的**（格式："如果 X 是原因，那么改变 Y 会让 Bug 消失/改变 Z 会让它更糟"）。**在测试前向用户展示排序列表**——他们可能有领域知识立即重排。

#### Phase 4: 插桩

每个探针必须映射到 Phase 3 的具体预测。**一次只改变一个变量**。工具偏好：调试器/REPL > 定向日志 > 绝不用"记录一切然后 grep"。用唯一前缀标记调试日志，便于清理。

#### Phase 5: 修复 + 回归测试

**在修复之前写回归测试**，但只在存在**正确接缝**时。如果不存在正确接缝，这本身就是发现——代码库架构阻止了 Bug 被锁定。

#### Phase 6: 清理

- 原始复现不再复现
- 回归测试通过
- 所有 `[DEBUG-...]` 插桩已删除
- 一次性原型已删除
- 正确的假设在 commit/PR 消息中说明

### 关键约束

- 在反馈循环存在之前，禁止阅读代码构建理论
- 必须脱敏所有密钥
- 跳过阶段必须有明确理由

---

## 第二部分：原文与翻译

<!--原文-->
> # Diagnosing Bugs
> A discipline for hard bugs. Skip phases only when explicitly justified.

<!--翻译-->
> # 诊断 Bug
> 针对困难 Bug 的纪律。仅在明确有理由时才跳过阶段。

<!--原文-->
> ## Phase 1: Build a feedback loop
> **This is the skill.** Everything else is mechanical. If you have a **tight** pass/fail signal for the bug (one that goes red on _this_ bug), you will find the cause... If you don't have one, no amount of staring at code will save you.
> Spend disproportionate effort here. **Be aggressive. Be creative. Refuse to give up.**

<!--翻译-->
> ## Phase 1: 构建反馈循环
> **这就是整个 Skill。**其他一切都是机械的。如果你有一个针对此 Bug 的**紧致**通过/失败信号（一个在此 Bug 上变红的信号），你就会找到原因...如果没有，看再多代码也救不了你。
> 在这里投入不成比例的努力。**要激进。要有创意。拒绝放弃。**

<!--原文-->
> Build the right feedback loop, and the bug is 90% fixed.

<!--翻译-->
> 构建正确的反馈循环，Bug 就已经修复了 90%。

<!--原文-->
> A 30-second flaky loop is barely better than no loop; a 2-second deterministic one is tight, a debugging superpower.

<!--翻译-->
> 30 秒的不稳定循环勉强比没有循环好一点；2 秒的确定性循环是紧致的，是调试的超能力。

<!--原文-->
> If you catch yourself reading code to build a theory before this command exists, **stop: jumping straight to a hypothesis is the exact failure this skill prevents.** No red-capable command, no Phase 2.

<!--翻译-->
> 如果你发现自己在反馈命令存在之前阅读代码构建理论，**停下来：直接跳到假设正是这个 Skill 要防止的失败**。没有能变红的命令，就没有 Phase 2。

<!--原文-->
> ## Phase 3: Hypothesise
> Generate **3–5 ranked hypotheses** before testing any of them. Single-hypothesis generation anchors on the first plausible idea.
> Each hypothesis must be **falsifiable**: state the prediction it makes.
> > Format: "If <X> is the cause, then <changing Y> will make the bug disappear / <changing Z> will make it worse."

<!--翻译-->
> ## Phase 3: 假设
> 在测试任何假设之前生成 **3-5 个排序假设**。单一假设生成会锚定在第一个貌似合理的想法上。
> 每个假设必须是**可证伪的**：陈述它做出的预测。
> > 格式："如果 X 是原因，那么改变 Y 会让 Bug 消失 / 改变 Z 会让它更糟。"

<!--原文-->
> ## Phase 5: Fix + regression test
> **If no correct seam exists, that itself is the finding.** Note it. The codebase architecture is preventing the bug from being locked down. Flag this for the next phase.

<!--翻译-->
> ## Phase 5: 修复 + 回归测试
> **如果不存在正确的接缝，这本身就是发现。**记录它。代码库架构正在阻止 Bug 被锁定。标记此问题以供下一阶段处理。