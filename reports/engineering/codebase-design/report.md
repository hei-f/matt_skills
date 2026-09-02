# codebase-design — 代码库设计深层模块词汇

## 第一部分：详细总结

### 概述

`codebase-design` 是模型调用的 Skill，提供设计**深层模块（deep modules）**的共享词汇和原则——在干净接缝处的小接口背后承载大量行为。目标是调用者的杠杆（leverage）、维护者的局部性（locality）和所有人的可测试性（testability）。

### 核心词汇表

| 术语 | 定义 | 避免使用的替代词 |
|------|------|------------------|
| **Module（模块）** | 任何有接口和实现的东西，尺度无关：函数、类、包等 | unit, component, service |
| **Interface（接口）** | 调用者正确使用模块需要知道的一切：类型签名、不变量、排序约束、错误模式、性能特征 | API, signature（太窄） |
| **Implementation（实现）** | 模块内部的代码体 | — |
| **Depth（深度）** | 接口处的杠杆：调用者每学习一单位接口能行使的行为量 | — |
| **Seam（接缝）** | 可以在不编辑该位置的情况下改变行为的地方（Michael Feathers 术语） | boundary（与 DDD 的限界上下文重载） |
| **Adapter（适配器）** | 在接缝处满足接口的具体事物，描述角色而非实质 | — |
| **Leverage（杠杆）** | 调用者从深度中获得的好处：更多能力/每单位接口 | — |
| **Locality（局部性）** | 维护者从深度中获得的好处：变更、Bug、知识、验证集中一处 | — |

### 深层 vs 浅层

- **深层模块** = 小接口 + 大量实现（理想形态）
- **浅层模块** = 大接口 + 少量实现（应避免）

### 核心原则

1. **深度是接口的属性**，而非实现的属性。深层模块内部可以由小的、可模拟、可替换的部分组成。
2. **删除测试**：想象删除该模块——如果复杂性消失了，它只是一个透传；如果复杂性在 N 个调用者中重新出现，它在发挥作用。
3. **接口就是测试面**：调用者和测试跨越同一个接缝。
4. **一个适配器 = 假设接缝，两个适配器 = 真实接缝**：除非确实有东西在接缝上变化，否则不要引入接缝。

### 可测试性设计

1. **接受依赖，不要创建依赖**：通过参数传入而非内部 new
2. **返回结果，不要产生副作用**：纯函数优于 void 修改
3. **小表面积**：更少的方法 = 更少的测试，更少的参数 = 更简单的测试设置

### 关系链

Module 有唯一 Interface → Depth 是 Module 相对其 Interface 的属性 → Seam 是 Interface 所在位置 → Adapter 位于 Seam 处 → Depth 为调用者产生 Leverage，为维护者产生 Locality

### 补充文件

- **DEEPENING.md**：依赖分类（进程内、本地可替代、远程但自有、真正外部）和接缝纪律
- **DESIGN-IT-TWICE.md**：并行子 Agent 模式，以多种截然不同的方式设计接口，然后比较深度、局部性和接缝位置

### 拒绝的框架

- 深度作为实现行数与接口行数的比率（奖励填充实现）
- "接口"作为 TypeScript `interface` 关键字（太窄）
- "边界（boundary）"作为术语（与 DDD 重载）

---

## 第二部分：原文与翻译

<!--原文-->
> # Codebase Design
> Design **deep modules**: a lot of behaviour behind a small interface, placed at a clean seam, testable through that interface.

<!--翻译-->
> # 代码库设计
> 设计**深层模块**：在干净接缝处的小接口背后承载大量行为，可通过该接口测试。

<!--原文-->
> **Module**: anything with an interface and an implementation. Deliberately scale-agnostic: a function, class, package, or tier-spanning slice.

<!--翻译-->
> **模块（Module）**：任何有接口和实现的东西。刻意与尺度无关：函数、类、包或跨层切片。

<!--原文-->
> **Depth**: leverage at the interface. The amount of behaviour a caller (or test) can exercise per unit of interface they have to learn.

<!--翻译-->
> **深度（Depth）**：接口处的杠杆。调用者（或测试）每学习一单位接口所能行使的行为量。

<!--原文-->
> **Seam** _(Michael Feathers)_: a place where you can alter behaviour without editing in that place; the *location* at which a module's interface lives.

<!--翻译-->
> **接缝（Seam）**（Michael Feathers 术语）：一个可以在不编辑该位置的情况下改变行为的地方；模块接口所在的*位置*。

<!--原文-->
> ## Deep vs shallow
> **Deep module** = small interface + lots of implementation
> **Shallow module** = large interface + little implementation (avoid)

<!--翻译-->
> ## 深层 vs 浅层
> **深层模块** = 小接口 + 大量实现
> **浅层模块** = 大接口 + 少量实现（应避免）

<!--原文-->
> - **The deletion test.** Imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.
> - **The interface is the test surface.** Callers and tests cross the same seam.
> - **One adapter means a hypothetical seam. Two adapters means a real one.**

<!--翻译-->
> - **删除测试**：想象删除该模块。如果复杂性消失，它是透传。如果复杂性在 N 个调用者中重新出现，它在发挥作用。
> - **接口就是测试面**：调用者和测试跨越同一个接缝。
> - **一个适配器 = 假设接缝，两个适配器 = 真实接缝。**

<!--原文-->
> ## Designing for testability
> 1. **Accept dependencies, don't create them.**
> 2. **Return results, don't produce side effects.**
> 3. **Small surface area.**

<!--翻译-->
> ## 可测试性设计
> 1. **接受依赖，不要创建依赖。**
> 2. **返回结果，不要产生副作用。**
> 3. **小表面积。**