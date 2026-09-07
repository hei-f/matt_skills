# domain-modeling — 领域建模

## 第一部分：详细总结

### 概述

`domain-modeling` 是一个模型调用的 Skill，用于在编码过程中**主动构建和优化**项目的领域模型。它不只是被动地读取 `CONTEXT.md`，而是积极地挑战术语、发明边缘场景、并实时记录术语表和决策。

### 文件结构

**单上下文（大多数仓库）**：
```
/CONTEXT.md          ← 术语表
/docs/adr/           ← 架构决策记录
/src/
```

**多上下文（通过 CONTEXT-MAP.md 存在来判断）**：
```
/CONTEXT-MAP.md      ← 指向各上下文的映射
/src/ordering/CONTEXT.md
/src/billing/CONTEXT.md
```

惰性创建文件：仅在需要写内容时才创建。

### 会话中的行为

1. **对照术语表挑战**：当用户使用的术语与 `CONTEXT.md` 冲突时立即指出
2. **锐化模糊语言**：当用户使用模糊或重载术语时，提出精确的标准术语
3. **讨论具体场景**：发明探测边缘情况的场景，迫使用户精确界定概念边界
4. **与代码交叉引用**：检查代码是否与用户的说法一致，发现矛盾时立即指出
5. **内联更新 CONTEXT.md**：术语一旦解决就立即更新，不要批处理
6. **谨慎提供 ADR**：仅当同时满足三个条件时才建议创建 ADR：
   - 难以逆转
   - 没有上下文会令人惊讶
   - 是真实权衡的结果

### 关键原则

- `CONTEXT.md` 应**完全不含实现细节**，只是术语表
- 不要将 `CONTEXT.md` 当作 Spec、草稿本或实现决策仓库
- ADR 可以只有一段话；价值在于记录决策及其原因

### 补充文件

- **ADR-FORMAT.md**：ADR 编号格式、模板（标题 + 背景/决策/原因）、可选字段（Status、Considered Options、Consequences）
- **CONTEXT-FORMAT.md**：术语表格式——每个术语加粗，一到两句定义，列出应避免的同义词

---

## 第二部分：原文与翻译

<!--原文-->
> # Domain Modeling
> Actively build and sharpen the project's domain model as you design. This is the *active* discipline: challenging terms, inventing edge-case scenarios, and writing the glossary and decisions down the moment they crystallise.

<!--翻译-->
> # 领域建模
> 在设计过程中主动构建和优化项目的领域模型。这是*主动*的纪律：挑战术语、发明边缘场景，并在术语和决策结晶的瞬间将其记录下来。

<!--原文-->
> ### Challenge against the glossary
> When the user uses a term that conflicts with the existing language in `CONTEXT.md`, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?"

<!--翻译-->
> ### 对照术语表挑战
> 当用户使用的术语与 `CONTEXT.md` 中现有语言冲突时，立即指出。"你的术语表将'取消'定义为 X，但你似乎指的是 Y。是哪一种？"

<!--原文-->
> ### Sharpen fuzzy language
> When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account': do you mean the Customer or the User? Those are different things."

<!--翻译-->
> ### 锐化模糊语言
> 当用户使用模糊或重载术语时，提出精确的标准术语。"你说的'account'：是指 Customer 还是 User？这是不同的事物。"

<!--原文-->
> ### Cross-reference with code
> When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible. Which is right?"

<!--翻译-->
> ### 与代码交叉引用
> 当用户陈述某事物的工作方式时，检查代码是否一致。如果发现矛盾，揭露它："你的代码取消整个订单，但你刚才说部分取消是可能的。哪种说法正确？"

<!--原文-->
> `CONTEXT.md` should be totally devoid of implementation details. Do not treat `CONTEXT.md` as a spec, a scratch pad, or a repository for implementation decisions. It is a glossary and nothing else.

<!--翻译-->
> `CONTEXT.md` 应完全不含实现细节。不要将 `CONTEXT.md` 当作 Spec、草稿本或实现决策仓库。它只是术语表，别无其他。

<!--原文-->
> ### Offer ADRs sparingly
> Only offer to create an ADR when all three are true:
> 1. **Hard to reverse**
> 2. **Surprising without context**
> 3. **The result of a real trade-off**
> If any of the three is missing, skip the ADR.

<!--翻译-->
> ### 谨慎提供 ADR
> 仅当三者同时成立时才建议创建 ADR：
> 1. **难以逆转**
> 2. **没有上下文会令人惊讶**
> 3. **是真实权衡的结果**
> 如果三者中缺少任何一个，跳过 ADR。