# code-review — 双轴代码审查

## 第一部分：详细总结

### 概述

`code-review` 是一个模型调用的 Skill，用于对自某个固定点以来的代码变更进行**双轴审查**。两个轴分别由并行子 Agent 独立执行，互不污染彼此的上下文。

### 双轴设计

- **Standards（标准轴）**：代码是否遵循仓库的编码标准？
- **Spec（规格轴）**：代码是否忠实地实现了原始 Issue/Spec？

两个轴的结果分别报告，故意不合并排名——防止一个轴掩盖另一个轴的问题。例如：
- 代码符合所有标准但实现了错误的功能 → Standards 通过，Spec 失败
- 代码完全符合需求但违反了项目约定 → Spec 通过，Standards 失败

### 流程

1. **确定固定点**：用户指定的 commit SHA、分支名、tag 等。使用 `git diff <fixed-point>...HEAD`（三点语法，与 merge-base 比较）。验证固定点可解析且 diff 非空。
2. **识别 Spec 来源**：按优先级查找——commit 消息中的 Issue 引用 → 用户参数中的路径 → `docs/`/`specs/`/`.scratch/` 下的 Spec 文件 → 询问用户。找不到则 Spec 轴跳过。
3. **识别标准来源**：仓库中的 `CODING_STANDARDS.md`、`CONTRIBUTING.md` 等。此外始终携带 **Fowler 代码坏味道基线**（12 种来自《重构》第3章的坏味道）：神秘命名、重复代码、依恋情结、数据泥团、基本类型偏执、重复 Switch、霰弹式修改、发散式变化、夸夸其谈通用性、消息链、中间人、拒绝遗赠。
4. **并行生成子 Agent**：两个子 Agent 的 prompt 都包含完整 diff 命令和 commit 列表，各自不超过 400 词。
5. **聚合结果**：分别在 `## Standards` 和 `## Spec` 标题下呈现。

### 关键规则

- 仓库文档标准覆盖基线坏味道
- 坏味道始终是判断性调用（judgement call），不是硬性违反
- 跳过工具已强制执行的规则（如 linter）

---

## 第二部分：原文与翻译

<!--原文-->
> Two-axis review of the diff between `HEAD` and a fixed point the user supplies:
> - **Standards**: does the code conform to this repo's documented coding standards?
> - **Spec**: does the code faithfully implement the originating issue / spec?

<!--翻译-->
> 对 `HEAD` 与用户提供的固定点之间的 diff 进行双轴审查：
> - **Standards（标准轴）**：代码是否符合此仓库的编码标准？
> - **Spec（规格轴）**：代码是否忠实实现了原始 Issue/Spec？

<!--原文-->
> Both axes run as **parallel sub-agents** so they don't pollute each other's context, then this skill aggregates their findings.

<!--翻译-->
> 两个轴作为**并行子 Agent**运行，互不污染彼此的上下文，然后本 Skill 聚合它们的发现。

<!--原文-->
> ### 3. Identify the standards sources
> Anything in the repo that documents how code should be written...
> On top of whatever the repo documents, the Standards axis always carries the **smell baseline** below: a fixed set of Fowler code smells

<!--翻译-->
> ### 3. 识别标准来源
> 仓库中任何记录代码编写方式的文档...
> 在仓库文档之上，标准轴始终携带以下**坏味道基线**：一组固定的 Fowler 代码坏味道

<!--原文-->
> - **Mysterious Name**: a function, variable, or type whose name doesn't reveal what it does → rename it
> - **Duplicated Code**: the same logic shape appears in more than one hunk → extract the shared shape
> - **Feature Envy**: a method that reaches into another object's data more than its own → move the method
> - **Data Clumps**: the same few fields or params keep travelling together → bundle them into one type
> - **Primitive Obsession**: a primitive standing in for a domain concept → give it its own type
> - **Repeated Switches**: same switch/if-cascade recurs → replace with polymorphism
> - **Shotgun Surgery**: one logical change forces scattered edits → gather into one module
> - **Divergent Change**: one module edited for several unrelated reasons → split
> - **Speculative Generality**: abstraction added for needs the spec doesn't have → delete it
> - **Message Chains**: long a.b().c().d() navigation → hide behind one method
> - **Middle Man**: a class that mostly just delegates → cut it, call the real target
> - **Refused Bequest**: subclass ignores most of what it inherits → use composition

<!--翻译-->
> - **神秘命名**：名称不揭示其作用的函数/变量/类型 → 重命名
> - **重复代码**：相同逻辑形状出现在多个位置 → 提取共享形状
> - **依恋情结**：方法访问另一个对象的数据多于自身 → 移动方法
> - **数据泥团**：相同字段/参数总是一起出现 → 捆绑为一个类型
> - **基本类型偏执**：基本类型代替领域概念 → 赋予其自己的类型
> - **重复 Switch**：相同 switch/if 级联重复出现 → 用多态替代
> - **霰弹式修改**：一个逻辑变更导致散落编辑 → 集中到一个模块
> - **发散式变化**：一个模块因不相关原因被编辑 → 拆分
> - **夸夸其谈通用性**：为 Spec 不需要的需求添加抽象 → 删除
> - **消息链**：长链式 a.b().c().d() 导航 → 隐藏在第一个对象的一个方法后
> - **中间人**：主要只是委托的类 → 移除，直接调用真正目标
> - **拒绝遗赠**：子类忽略/覆盖大部分继承内容 → 用组合替代继承

<!--原文-->
> ## Why two axes
> A change can pass one axis and fail the other... Reporting them separately stops one axis from masking the other.

<!--翻译-->
> ## 为什么双轴
> 一个变更可能通过一个轴而失败另一个...分开报告防止一个轴掩盖另一个轴的问题。