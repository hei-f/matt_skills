# migrate-to-shoehorn — 迁移至 Shoehorn 类型安全断言库

## 第一部分：详细总结

### 概述

`migrate-to-shoehorn` 是一个代码迁移 Skill，专门将测试文件中的 `as` 类型断言替换为 `@total-typescript/shoehorn` 库提供的类型安全替代方案。该 Skill 仅在测试代码中使用，不可用于生产代码。

### 为什么使用 Shoehorn

`shoehorn` 允许在测试中传入部分数据的同时保持 TypeScript 类型检查的满意度。它解决了 `as` 断言的三个核心问题：需要手动指定目标类型、训练中习惯避免使用、以及对于故意传入错误数据需要双重断言（`as unknown as Type`）。

### 三种迁移模式

#### 1. 大对象仅需少量属性：`fromPartial()`

当函数接受一个大型类型但测试只需其中少量属性时，传统做法需要伪造整个对象的所有属性。`fromPartial()` 接受部分属性，自动完成类型检查。

迁移前：
```ts
getUser({
  body: { id: "123" },
  headers: {},
  cookies: {},
  // ...伪造所有 20 个属性
});
```

迁移后：
```ts
import { fromPartial } from "@total-typescript/shoehorn";
getUser(fromPartial({ body: { id: "123" } }));
```

#### 2. `as Type` → `fromPartial()`

直接的类型断言替换。将 `data as Type` 替换为 `fromPartial(data)`，同时添加 import。

迁移前：
```ts
getUser({ body: { id: "123" } } as Request);
```

迁移后：
```ts
import { fromPartial } from "@total-typescript/shoehorn";
getUser(fromPartial({ body: { id: "123" } }));
```

#### 3. `as unknown as Type` → `fromAny()`

当测试需要传入故意类型错误的数据以验证错误处理时使用。将 `as unknown as Type` 替换为 `fromAny()`，保留了自动补全功能。

迁移前：
```ts
getUser({ body: { id: 123 } } as unknown as Request);
```

迁移后：
```ts
import { fromAny } from "@total-typescript/shoehorn";
getUser(fromAny({ body: { id: 123 } }));
```

### 三个函数的用途对照

| 函数 | 用途 |
|------|------|
| `fromPartial()` | 传入部分数据，仍然通过类型检查 |
| `fromAny()` | 传入故意错误的数据（保留自动补全） |
| `fromExact()` | 强制传入完整对象（后续可从 `fromPartial()` 切换） |

### 工作流程

1. **收集需求**：询问用户哪些测试文件的 `as` 断言造成问题、是否处理大对象仅需部分属性、是否需要传入故意错误的数据
2. **安装并迁移**：安装 npm 包 → 使用 grep 查找测试文件中含 `as` 断言的代码 → 替换 `as Type` 为 `fromPartial()` → 替换 `as unknown as Type` 为 `fromAny()` → 添加 import → 运行类型检查验证

---

## 第二部分：原文与翻译

<!--原文-->
> # Migrate to Shoehorn
> `shoehorn` lets you pass partial data in tests while keeping TypeScript happy. It replaces `as` assertions with type-safe alternatives.
> **Test code only.** Never use shoehorn in production code.

<!--翻译-->
> # 迁移至 Shoehorn
> `shoehorn` 允许在测试中传入部分数据的同时保持 TypeScript 类型检查的满意度。它用类型安全的替代方案替换 `as` 断言。
> **仅测试代码。** 绝不在生产代码中使用 shoehorn。

<!--原文-->
> Problems with `as` in tests:
> - Trained not to use it
> - Must manually specify target type
> - Double-as (`as unknown as Type`) for intentionally wrong data

<!--翻译-->
> 测试中使用 `as` 的问题：
> - 训练中习惯避免使用
> - 必须手动指定目标类型
> - 对于故意错误的数据需要双重断言（`as unknown as Type`）

<!--原文-->
> ### Large objects with few needed properties
> Before:
> ```ts
> getUser({
>   body: { id: "123" },
>   headers: {},
>   cookies: {},
>   // ...20 more properties
> });
> ```
> After:
> ```ts
> import { fromPartial } from "@total-typescript/shoehorn";
> getUser(fromPartial({ body: { id: "123" } }));
> ```

<!--翻译-->
> ### 大对象仅需少量属性
> 迁移前：
> ```ts
> getUser({
>   body: { id: "123" },
>   headers: {},
>   cookies: {},
>   // ...20 多个属性
> });
> ```
> 迁移后：
> ```ts
> import { fromPartial } from "@total-typescript/shoehorn";
> getUser(fromPartial({ body: { id: "123" } }));
> ```

<!--原文-->
> ### `as Type` → `fromPartial()`
> Before: `getUser({ body: { id: "123" } } as Request);`
> After:
> ```ts
> import { fromPartial } from "@total-typescript/shoehorn";
> getUser(fromPartial({ body: { id: "123" } }));
> ```

<!--翻译-->
> ### `as Type` → `fromPartial()`
> 迁移前：`getUser({ body: { id: "123" } } as Request);`
> 迁移后：
> ```ts
> import { fromPartial } from "@total-typescript/shoehorn";
> getUser(fromPartial({ body: { id: "123" } }));
> ```

<!--原文-->
> ### `as unknown as Type` → `fromAny()`
> Before: `getUser({ body: { id: 123 } } as unknown as Request); // wrong type on purpose`
> After:
> ```ts
> import { fromAny } from "@total-typescript/shoehorn";
> getUser(fromAny({ body: { id: 123 } }));
> ```

<!--翻译-->
> ### `as unknown as Type` → `fromAny()`
> 迁移前：`getUser({ body: { id: 123 } } as unknown as Request); // 故意传入错误类型`
> 迁移后：
> ```ts
> import { fromAny } from "@total-typescript/shoehorn";
> getUser(fromAny({ body: { id: 123 } }));
> ```

<!--原文-->
> ## When to use each
> | Function | Use case |
> |---|---|
> | `fromPartial()` | Pass partial data that still type-checks |
> | `fromAny()` | Pass intentionally wrong data (keeps autocomplete) |
> | `fromExact()` | Force full object (swap with fromPartial later) |

<!--翻译-->
> ## 各函数用途
> | 函数 | 用途 |
> |------|------|
> | `fromPartial()` | 传入部分数据，仍然通过类型检查 |
> | `fromAny()` | 传入故意错误的数据（保留自动补全） |
> | `fromExact()` | 强制传入完整对象（后续可从 fromPartial 切换） |

<!--原文-->
> ## Workflow
> 1. **Gather requirements** - ask user:
>    - What test files have `as` assertions causing problems?
>    - Are they dealing with large objects where only some properties matter?
>    - Do they need to pass intentionally wrong data for error testing?
> 2. **Install and migrate**:
>    - [ ] Install: `npm i @total-typescript/shoehorn`
>    - [ ] Find test files with `as` assertions: `grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
>    - [ ] Replace `as Type` with `fromPartial()`
>    - [ ] Replace `as unknown as Type` with `fromAny()`
>    - [ ] Add imports from `@total-typescript/shoehorn`
>    - [ ] Run type check to verify

<!--翻译-->
> ## 工作流程
> 1. **收集需求**——询问用户：
>    - 哪些测试文件的 `as` 断言造成问题？
>    - 是否处理大对象仅需部分属性的场景？
>    - 是否需要传入故意错误的数据以测试错误处理？
> 2. **安装并迁移**：
>    - [ ] 安装：`npm i @total-typescript/shoehorn`
>    - [ ] 查找含 `as` 断言的测试文件：`grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
>    - [ ] 将 `as Type` 替换为 `fromPartial()`
>    - [ ] 将 `as unknown as Type` 替换为 `fromAny()`
>    - [ ] 添加从 `@total-typescript/shoehorn` 的 import
>    - [ ] 运行类型检查验证
