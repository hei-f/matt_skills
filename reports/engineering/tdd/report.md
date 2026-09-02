# TDD 技能详解报告

---

## 第一部分：中文详细总结

### 一、TDD 概述：红-绿-重构循环

TDD（测试驱动开发）本质上是一个"红 → 绿"的循环。这个技能（`tdd`）是一套参考标准，确保这个循环产出的测试是值得保留的。它解决的核心问题是：什么样的测试才是好测试、测试应该写在哪里、有哪些常见的反模式需要避免、以及循环本身的规则是什么。

整个技能的设计理念是：**每一节内容在每一个循环中都适用**——在循环开始前和循环进行中都要查阅，而不是在循环结束后才回头看。

当探索代码库时，应该先阅读 `CONTEXT.md`（如果存在），让测试名称和接口词汇与项目的领域语言保持一致，同时尊重所触及区域的架构决策记录（ADR）。

本技能的定位是一个**参考标准**，而非驱动程序。它定义了循环的规则，而由其他组件（用户或 `implement` 技能）来运行实际的 TDD 会话。

**关键原则：红-绿循环。** 先写一个会失败的测试，然后只写刚好能让它通过的代码。不要预判下一个测试，不要添加推测性的功能。重构步骤已从循环中移除（2026年6月），因为 AI Agent 几乎从不执行重构，而且将实现和评审分离到不同会话中效果更好。重构现在属于 `code-review` 技能的范畴。

### 二、什么是一个好的测试

好的测试通过**公共接口验证行为**，而不是验证实现细节。代码可以完全重写，但测试不应该随之改变。

一个优秀的测试读起来就像一份**规格说明书**：例如 `"user can checkout with valid cart"`（用户可以使用有效购物车结算）这个测试名称，直接告诉你系统具备什么能力。而且它能在重构中存活下来，因为它不关心内部结构。

好测试的特征：
- 测试用户/调用者关心的行为
- 只使用公共 API
- 能在内部重构中存活
- 描述的是"做什么"（WHAT），而不是"怎么做"（HOW）
- 每个测试只包含一个逻辑断言

### 三、接缝（Seams）：测试应该写在哪里

**接缝（Seam）** 是指你测试时所在的公共边界：即你在不触及内部的情况下观察行为的接口。测试必须放在接缝处，绝不能针对内部实现。

**核心规则：只在预先约定的接缝处测试。** 在写任何测试之前，必须先写下要测试的接缝，并与用户确认。没有经过确认的接缝，就不能写测试。你不可能测试一切，所以提前约定接缝，就是为了让测试精力集中在关键路径和复杂逻辑上，而不是每个边界情况。

应该问的问题是："什么是公共接口？我们应该在哪些接缝处测试？"

当接口本身的形态还存在疑问时（模块应该多深、接缝应该在哪里、接口应该暴露什么），应该调用 `codebase-design` 技能来获取相关词汇表。它是模块、接口、深度、接缝、适配器、杠杆效应和局部性等术语的共享来源。

**接缝的约束力。** 约定的接缝会贯穿整个流程：`tdd` 只在约定的接缝处工作，`code-review` 会检查是否只使用了约定的接缝。这意味着如果你试图在未约定的接缝上测试，代码审查会发现这个问题。

### 四、反模式（Anti-patterns）

#### 反模式一：与实现耦合的测试（Implementation-coupled tests）

**特征：** Mock 内部协作者、测试私有方法、或通过非接口的旁路来验证（比如直接查询数据库而不是使用接口）。

**识别标志：** 当你重构代码时测试会失败，但实际行为并没有改变。

**具体表现：**
- Mock 了你自己控制的类/模块
- 断言调用次数或调用顺序
- 测试私有方法
- 绕过接口直接验证（如直接查数据库来验证 `createUser` 的结果，而不是通过 `getUser` 接口来验证）

#### 反模式二：同义反复的测试（Tautological tests）

**特征：** 断言中的期望值以和代码相同的方式重新计算。例如 `expect(add(a, b)).toBe(a + b)`，或者手工用相同方式推导出的快照，或者一个常量等于它自己。这样的测试"构造即通过"，永远不会与代码产生分歧。

**正确做法：** 期望值必须来自独立的事实来源：一个已知正确的字面量、一个手工计算的例子、或者规格说明书。

**示例：**
```typescript
// 错误：期望值以相同方式重新计算
test("calculateTotal sums line items", () => {
  const items = [{ price: 10 }, { price: 5 }];
  const expected = items.reduce((sum, i) => sum + i.price, 0);
  expect(calculateTotal(items)).toBe(expected);
});

// 正确：期望值是独立的已知字面量
test("calculateTotal sums line items", () => {
  expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
});
```

#### 反模式三：水平切片（Horizontal slicing）

**特征：** 先写完所有测试，再写所有实现代码。批量测试验证的是"想象中"的行为：你在测试事物的"形状"而非面向用户的行为，测试对真实变化变得不敏感，而且你在理解实现之前就已经锁定了测试结构。

**正确做法：** 使用**垂直切片**的方式：一个测试 → 一个实现 → 重复。每个测试都是一颗**曳光弹**（tracer bullet），它响应上一个循环教给你的东西。

### 五、循环的规则

1. **红先于绿。** 先写失败的测试，然后只写刚好能通过它的代码。不要预判未来的测试或添加推测性功能。

2. **一次一个切片。** 每个循环一个接缝、一个测试、一个最小实现。

3. **重构不属于这个循环。** 重构属于代码审查阶段（参见 `code-review` 技能），不属于红 → 绿的实现循环。这是 2026 年 6 月的有意设计决策：Agent 几乎从不执行重构，将实现和评审分开效果更好。无论这是否符合传统意义上的 TDD，关键是这个循环能产出更好的代码。

### 六、补充文件详解

#### mocking.md：何时使用 Mock

**核心原则：只在系统边界处 Mock。**

需要 Mock 的场景：
- 外部 API（支付、邮件等）
- 数据库（有时——更倾向于使用测试数据库）
- 时间/随机性
- 文件系统（有时）

不应该 Mock 的：
- 你自己的类/模块
- 内部协作者
- 任何你控制的东西

**为可 Mock 性而设计：**

1. **使用依赖注入。** 将外部依赖从外部传入，而不是在内部创建：
```typescript
// 易于 Mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// 难以 Mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

2. **优先使用 SDK 风格的接口，而非通用 fetcher。** 为每个外部操作创建特定的函数，而非一个带条件逻辑的通用函数。SDK 方式的优势在于：每个 Mock 返回一种特定形状、测试设置中不需要条件逻辑、更容易看出测试使用了哪些端点、每个端点的类型安全。

```typescript
// 好的：每个函数可以独立 Mock
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// 不好的：Mock 需要条件逻辑
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

#### tests.md：好测试与坏测试

**好的测试——集成式风格：** 通过真实接口测试，不 Mock 内部组件。测试的是调用者和用户关心的可观察行为。只使用公共 API，能在内部重构中存活。描述"做什么"（WHAT）而非"怎么做"（HOW）。每个测试一个逻辑断言。

**坏的测试——实现细节式测试：** 与内部结构耦合。Mock 内部协作者、测试私有方法、断言调用次数/顺序、重构时测试会失败但行为未变、测试名称描述"怎么做"而非"做什么"、通过非接口的外部方式验证。

**坏的测试——同义反复式测试：** 期望值只是复述了实现逻辑，所以测试在构造上必然通过。期望值必须是独立的、已知正确的字面量。

---

## 第二部分：原文与翻译

以下为 SKILL.md、mocking.md 和 tests.md 三个文件的逐段原文与中文翻译。

### SKILL.md

---

<!--原文-->
> TDD is the red → green loop. This skill is the reference that makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop. Every section applies on every cycle: consult them before and during the loop, not after.
<!--翻译-->
> TDD 是红 → 绿的循环。这个技能是一套参考标准，确保这个循环产出的测试值得保留：什么样的测试是好测试、测试应该写在哪里、有哪些反模式、以及循环的规则。每一节在每一个循环中都适用：在循环开始前和循环进行中查阅，而非循环结束后。

---

<!--原文-->
> When exploring the codebase, read `CONTEXT.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching.
<!--翻译-->
> 当探索代码库时，先阅读 `CONTEXT.md`（如果存在），让测试名称和接口词汇与项目的领域语言保持一致，同时尊重所触及区域的架构决策记录（ADR）。

---

<!--原文-->
> Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't. A good test reads like a specification: "user can checkout with valid cart" tells you exactly what capability exists, and it survives refactors because it doesn't care about internal structure.
<!--翻译-->
> 测试通过公共接口验证行为，而非验证实现细节。代码可以完全重写，测试不应该随之改变。一个好的测试读起来就像一份规格说明书："user can checkout with valid cart"（用户可以使用有效购物车结算）精确地告诉你系统具备什么能力，而且它能在重构中存活，因为它不关心内部结构。

---

<!--原文-->
> See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.
<!--翻译-->
> 参见 tests.md 获取示例，参见 mocking.md 获取 Mock 使用指南。

---

<!--原文-->
> A **seam** is the public boundary you test at: the interface where you observe behavior without reaching inside. Tests live at seams, never against internals.
<!--翻译-->
> **接缝（seam）** 是你测试时所在的公共边界：即你在不触及内部的情况下观察行为的接口。测试必须放在接缝处，绝不能针对内部实现。

---

<!--原文-->
> **Test only at pre-agreed seams.** Before writing any test, write down the seams under test and confirm them with the user. No test is written at an unconfirmed seam. You can't test everything, so agreeing the seams up front is how testing effort lands on the critical paths and complex logic instead of every edge case.
<!--翻译-->
> **只在预先约定的接缝处测试。** 在写任何测试之前，先写下要测试的接缝，并与用户确认。没有经过确认的接缝，就不能写测试。你不可能测试一切，所以提前约定接缝，就是为了让测试精力集中在关键路径和复杂逻辑上，而不是每个边界情况。

---

<!--原文-->
> Ask: "What's the public interface, and which seams should we test?"
<!--翻译-->
> 应该问："什么是公共接口？我们应该在哪些接缝处测试？"

---

<!--原文-->
> When the shape of that interface is itself in question (how deep the module is, where the seam belongs, what the interface should expose), call the Skill tool with "codebase-design" for the vocabulary. It is the shared source of the module, interface, depth, seam, adapter, leverage and locality terms, and it is a reference to consult, not a session to run.
<!--翻译-->
> 当接口本身的形态还存在疑问时（模块应该多深、接缝应该在哪里、接口应该暴露什么），调用 Skill 工具并指定 "codebase-design" 来获取相关词汇表。它是模块、接口、深度、接缝、适配器、杠杆效应和局部性等术语的共享来源，而且它是一个参考标准可供查阅，而非一个需要运行的会话。

---

<!--原文-->
> - **Implementation-coupled**: mocks internal collaborators, tests private methods, or verifies through a side channel (querying the database instead of using the interface). The tell: the test breaks when you refactor but behavior hasn't changed.
<!--翻译-->
> - **与实现耦合的测试**：Mock 内部协作者、测试私有方法、或通过非接口的旁路来验证（直接查询数据库而不是使用接口）。识别标志：当你重构代码时测试会失败，但实际行为并没有改变。

---

<!--原文-->
> - **Tautological**: the assertion recomputes the expected value the way the code does (`expect(add(a, b)).toBe(a + b)`, a snapshot derived by hand the same way, a constant asserted equal to itself), so it passes by construction and can never disagree with the code. Expected values must come from an independent source of truth: a known-good literal, a worked example, the spec.
<!--翻译-->
> - **同义反复的测试**：断言中的期望值以和代码相同的方式重新计算（如 `expect(add(a, b)).toBe(a + b)`，手工用相同方式推导出的快照，一个常量断言等于它自己），所以它"构造即通过"，永远不会与代码产生分歧。期望值必须来自独立的事实来源：一个已知正确的字面量、一个手工计算的例子、或者规格说明书。

---

<!--原文-->
> - **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests verify _imagined_ behavior: you test the _shape_ of things rather than user-facing behavior, the tests go insensitive to real changes, and you commit to test structure before understanding the implementation. Work in **vertical slices** instead: one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.
<!--翻译-->
> - **水平切片**：先写完所有测试，再写所有实现代码。批量测试验证的是"想象中"的行为：你在测试事物的"形状"而非面向用户的行为，测试对真实变化变得不敏感，而且你在理解实现之前就已经锁定了测试结构。应该使用**垂直切片**的方式：一个测试 → 一个实现 → 重复，每个测试都是一颗**曳光弹**（tracer bullet），它响应上一个循环教给你的东西。

---

<!--原文-->
> - **Red before green.** Write the failing test first, then only enough code to pass it. Don't anticipate future tests or add speculative features.
<!--翻译-->
> - **红先于绿。** 先写失败的测试，然后只写刚好能通过它的代码。不要预判未来的测试或添加推测性功能。

---

<!--原文-->
> - **One slice at a time.** One seam, one test, one minimal implementation per cycle.
<!--翻译-->
> - **一次一个切片。** 每个循环一个接缝、一个测试、一个最小实现。

---

<!--原文-->
> - **Refactoring is not part of the loop.** It belongs to the review stage (see the `code-review` skill), not the red → green implementation cycle.
<!--翻译-->
> - **重构不属于这个循环。** 重构属于代码审查阶段（参见 `code-review` 技能），不属于红 → 绿的实现循环。

---

### mocking.md

---

<!--原文-->
> # When to Mock
<!--翻译-->
> # 何时使用 Mock

---

<!--原文-->
> Mock at **system boundaries** only:
<!--翻译-->
> 只在**系统边界**处 Mock：

---

<!--原文-->
> - External APIs (payment, email, etc.)
<!--翻译-->
> - 外部 API（支付、邮件等）

---

<!--原文-->
> - Databases (sometimes - prefer test DB)
<!--翻译-->
> - 数据库（有时——更倾向于使用测试数据库）

---

<!--原文-->
> - Time/randomness
<!--翻译-->
> - 时间/随机性

---

<!--原文-->
> - File system (sometimes)
<!--翻译-->
> - 文件系统（有时）

---

<!--原文-->
> Don't mock:
<!--翻译-->
> 不应该 Mock：

---

<!--原文-->
> - Your own classes/modules
<!--翻译-->
> - 你自己的类/模块

---

<!--原文-->
> - Internal collaborators
<!--翻译-->
> - 内部协作者

---

<!--原文-->
> - Anything you control
<!--翻译-->
> - 任何你控制的东西

---

<!--原文-->
> ## Designing for Mockability
<!--翻译-->
> ## 为可 Mock 性而设计

---

<!--原文-->
> At system boundaries, design interfaces that are easy to mock:
<!--翻译-->
> 在系统边界处，设计易于 Mock 的接口：

---

<!--原文-->
> **1. Use dependency injection**
<!--翻译-->
> **1. 使用依赖注入**

---

<!--原文-->
> Pass external dependencies in rather than creating them internally:
<!--翻译-->
> 将外部依赖从外部传入，而不是在内部创建：

---

<!--原文-->
> ```typescript
> // Easy to mock
> function processPayment(order, paymentClient) {
>   return paymentClient.charge(order.total);
> }
> 
> // Hard to mock
> function processPayment(order) {
>   const client = new StripeClient(process.env.STRIPE_KEY);
>   return client.charge(order.total);
> }
> ```
<!--翻译-->
> ```typescript
> // 易于 Mock
> function processPayment(order, paymentClient) {
>   return paymentClient.charge(order.total);
> }
> 
> // 难以 Mock
> function processPayment(order) {
>   const client = new StripeClient(process.env.STRIPE_KEY);
>   return client.charge(order.total);
> }
> ```
---

<!--原文-->
> **2. Prefer SDK-style interfaces over generic fetchers**
<!--翻译-->
> **2. 优先使用 SDK 风格的接口，而非通用 fetcher**

---

<!--原文-->
> Create specific functions for each external operation instead of one generic function with conditional logic:
<!--翻译-->
> 为每个外部操作创建特定的函数，而非一个带条件逻辑的通用函数：

---

<!--原文-->
> ```typescript
> // GOOD: Each function is independently mockable
> const api = {
>   getUser: (id) => fetch(`/users/${id}`),
>   getOrders: (userId) => fetch(`/users/${userId}/orders`),
>   createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
> };
> 
> // BAD: Mocking requires conditional logic inside the mock
> const api = {
>   fetch: (endpoint, options) => fetch(endpoint, options),
> };
> ```
<!--翻译-->
> ```typescript
> // 好的：每个函数可以独立 Mock
> const api = {
>   getUser: (id) => fetch(`/users/${id}`),
>   getOrders: (userId) => fetch(`/users/${userId}/orders`),
>   createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
> };
> 
> // 不好的：Mock 需要条件逻辑
> const api = {
>   fetch: (endpoint, options) => fetch(endpoint, options),
> };
> ```
---

<!--原文-->
> The SDK approach means:
<!--翻译-->
> SDK 方式的优势是：

---

<!--原文-->
> - Each mock returns one specific shape
<!--翻译-->
> - 每个 Mock 返回一种特定形状

---

<!--原文-->
> - No conditional logic in test setup
<!--翻译-->
> - 测试设置中不需要条件逻辑

---

<!--原文-->
> - Easier to see which endpoints a test exercises
<!--翻译-->
> - 更容易看出测试使用了哪些端点

---

<!--原文-->
> - Type safety per endpoint
<!--翻译-->
> - 每个端点的类型安全

---

### tests.md

---

<!--原文-->
> # Good and Bad Tests
<!--翻译-->
> # 好的测试与坏的测试

---

<!--原文-->
> ## Good Tests
<!--翻译-->
> ## 好的测试

---

<!--原文-->
> **Integration-style**: Test through real interfaces, not mocks of internal parts.
<!--翻译-->
> **集成式风格**：通过真实接口测试，不 Mock 内部组件。

---

<!--原文-->
> ```typescript
> // GOOD: Tests observable behavior
> test("user can checkout with valid cart", async () => {
>   const cart = createCart();
>   cart.add(product);
>   const result = await checkout(cart, paymentMethod);
>   expect(result.status).toBe("confirmed");
> });
> ```
<!--翻译-->
> ```typescript
> // 好的：测试可观察的行为
> test("user can checkout with valid cart", async () => {
>   const cart = createCart();
>   cart.add(product);
>   const result = await checkout(cart, paymentMethod);
>   expect(result.status).toBe("confirmed");
> });
> ```
---

<!--原文-->
> Characteristics:
<!--翻译-->
> 特征：

---

<!--原文-->
> - Tests behavior users/callers care about
<!--翻译-->
> - 测试用户/调用者关心的行为

---

<!--原文-->
> - Uses public API only
<!--翻译-->
> - 只使用公共 API

---

<!--原文-->
> - Survives internal refactors
<!--翻译-->
> - 能在内部重构中存活

---

<!--原文-->
> - Describes WHAT, not HOW
<!--翻译-->
> - 描述"做什么"（WHAT），而非"怎么做"（HOW）

---

<!--原文-->
> - One logical assertion per test
<!--翻译-->
> - 每个测试一个逻辑断言

---

<!--原文-->
> ## Bad Tests
<!--翻译-->
> ## 坏的测试

---

<!--原文-->
> **Implementation-detail tests**: Coupled to internal structure.
<!--翻译-->
> **实现细节式测试**：与内部结构耦合。

---

<!--原文-->
> ```typescript
> // BAD: Tests implementation details
> test("checkout calls paymentService.process", async () => {
>   const mockPayment = jest.mock(paymentService);
>   await checkout(cart, payment);
>   expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
> });
> ```
<!--翻译-->
> ```typescript
> // 不好的：测试实现细节
> test("checkout calls paymentService.process", async () => {
>   const mockPayment = jest.mock(paymentService);
>   await checkout(cart, payment);
>   expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
> });
> ```
---

<!--原文-->
> Red flags:
<!--翻译-->
> 危险信号：

---

<!--原文-->
> - Mocking internal collaborators
<!--翻译-->
> - Mock 内部协作者

---

<!--原文-->
> - Testing private methods
<!--翻译-->
> - 测试私有方法

---

<!--原文-->
> - Asserting on call counts/order
<!--翻译-->
> - 断言调用次数/顺序

---

<!--原文-->
> - Test breaks when refactoring without behavior change
<!--翻译-->
> - 重构时测试失败但行为未变

---

<!--原文-->
> - Test name describes HOW not WHAT
<!--翻译-->
> - 测试名称描述"怎么做"而非"做什么"

---

<!--原文-->
> - Verifying through external means instead of interface
<!--翻译-->
> - 通过非接口的外部方式验证

---

<!--原文-->
> ```typescript
> // BAD: Bypasses interface to verify
> test("createUser saves to database", async () => {
>   await createUser({ name: "Alice" });
>   const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
>   expect(row).toBeDefined();
> });
> 
> // GOOD: Verifies through interface
> test("createUser makes user retrievable", async () => {
>   const user = await createUser({ name: "Alice" });
>   const retrieved = await getUser(user.id);
>   expect(retrieved.name).toBe("Alice");
> });
> ```
<!--翻译-->
> ```typescript
> // 不好的：绕过接口直接验证
> test("createUser saves to database", async () => {
>   await createUser({ name: "Alice" });
>   const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
>   expect(row).toBeDefined();
> });
> 
> // 好的：通过接口验证
> test("createUser makes user retrievable", async () => {
>   const user = await createUser({ name: "Alice" });
>   const retrieved = await getUser(user.id);
>   expect(retrieved.name).toBe("Alice");
> });
> ```
---

<!--原文-->
> **Tautological tests**: Expected value restates the implementation, so the test passes by construction.
<!--翻译-->
> **同义反复式测试**：期望值只是复述了实现逻辑，所以测试在构造上必然通过。

---

<!--原文-->
> ```typescript
> // BAD: Expected value is recomputed the way the code computes it
> test("calculateTotal sums line items", () => {
>   const items = [{ price: 10 }, { price: 5 }];
>   const expected = items.reduce((sum, i) => sum + i.price, 0);
>   expect(calculateTotal(items)).toBe(expected);
> });
> 
> // GOOD: Expected value is an independent, known literal
> test("calculateTotal sums line items", () => {
>   expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
> });
> ```
<!--翻译-->
> ```typescript
> // 不好的：期望值以和代码相同的方式重新计算
> test("calculateTotal sums line items", () => {
>   const items = [{ price: 10 }, { price: 5 }];
>   const expected = items.reduce((sum, i) => sum + i.price, 0);
>   expect(calculateTotal(items)).toBe(expected);
> });
> 
> // 好的：期望值是独立的、已知正确的字面量
> test("calculateTotal sums line items", () => {
>   expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
> });
> ```
