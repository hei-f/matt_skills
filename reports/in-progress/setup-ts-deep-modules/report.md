# setup-ts-deep-modules 技能报告

## 第一部分：中文详细总结

### 概述

setup-ts-deep-modules 是一个用户手动调用的技能，用于将 dependency-cruiser 接入 TypeScript 仓库，确保每个包都遵循"深度模块"（deep module）架构：大量行为隐藏在小接口之后。包的公开表面是其**入口点**（包根目录下的文件），子文件夹中的一切对外不可见。该技能安装 dependency-cruiser 并配置规则，使入口点成为唯一的访问途径，然后验证这些规则确实生效。

### 深度模块架构形态

```
src/packages/
  <name>/
    index.ts        ← 入口点（公开）。从外部导入此文件。
    client.ts       ← 另一个入口点。包可以暴露多个入口点。
    lib/            ← 实现：对外隐藏，内部可自由互导。
    tests/          ← 与实现同层的测试和测试数据（子文件夹，因此私有）。
```

公开表面是包**根目录下的所有文件**，而非只指定一个 `index.ts`。按照约定，实现放在 `lib/`，测试放在 `tests/`，使每个包都保持相同的两层文件夹结构。规则本质上是通用的：**任何**子文件夹中的**任何**内容都是私有的，因此添加新文件夹不需要更改配置。

### 四条规则（全部为 error 级别）

1. **入口点边界**：包外代码（应用代码或其他包）只能导入该包的入口点（根目录文件），绝不能导入其子文件夹中的任何内容。
2. **包内自由**：包内文件之间可以自由互导。
3. **通过入口点测试**：`<pkg>/tests/` 下的文件可以导入任何包的入口点及其自己的 `tests/` 测试数据文件，但绝不能导入任何包的子文件夹内部实现（即使是自己的也不行）。跨包集成测试可以；深层导入不可以。
4. **禁止循环**：不允许任何依赖循环。

**入口点而非桶文件**：由于公开表面是**每个**根文件，包可以暴露多个小型入口点（`index.ts`、`client.ts`、`server.ts`），而不是将所有内容通过一个巨大的 `index.ts` 导出。强烈不鼓励使用桶文件（barrel files）重新导出整个子树；保持入口点精简，将实现隐藏在子文件夹中。

分层（哪些包可以依赖哪些包）是另一个不同的问题，在配置中以注释桩的形式留给仓库自行填写。

### 七个步骤

#### 步骤 1：检测环境

- **包管理器**：`pnpm-lock.yaml` → pnpm，`yarn.lock` → yarn，`bun.lockb` → bun，否则 npm。用于后续所有命令。
- **包根目录**：如果存在 `src/`，则使用 `src/packages`，否则用 `packages`。如果仓库已有不同的明显约定，与用户确认。
- **现有配置**：检查是否存在 `.dependency-cruiser.*` 文件。如果存在，不要覆盖它：将四条规则和选项合并进去，并告知用户添加了什么。

**完成标准**：包管理器、包根目录和现有配置状态全部确认。

#### 步骤 2：安装 dependency-cruiser

使用已检测到的包管理器将 `dependency-cruiser` 安装为 devDependency。

**完成标准**：`dependency-cruiser` 出现在 `devDependencies` 中。

#### 步骤 3：编写配置

将 `dependency-cruiser.config.cjs` 复制到仓库根目录，命名为 `.dependency-cruiser.cjs`。将 `PACKAGES_ROOT` 设置为步骤 1 中检测到的根目录。规则基于路径深度且与扩展名无关，因此无需其他调整。

**完成标准**：`.dependency-cruiser.cjs` 存在，`PACKAGES_ROOT` 正确，四条禁止规则均已配置。

#### 步骤 4：接入检查命令

- 添加 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。
- 将其整合到仓库的伞形检查命令中，该命令应已包含类型检查（例如 `check`、`ci` 或 `validate` 脚本）。不要修改 `tsconfig` 或添加路径别名。
- 如果没有伞形脚本，添加 `lint:boundaries` 并告知用户将其纳入 CI。

**完成标准**：`lint:boundaries` 存在且作为与类型检查相同的命令的一部分运行。

#### 步骤 5：搭建示例包

创建一个已提交的 `<packages-root>/example/` 作为可复制模板：

- `index.ts` 为入口点。导出一个委托给内部文件的函数（使包明显是**深层**的，而非全透传）。
- `lib/impl.ts`：子文件夹中的内部文件，由 `index.ts` 导入，外部无法访问。
- `tests/example.test.ts`：**仅**导入 `../index`（一个入口点），并对公开函数进行断言。

告知用户这是一个可复制或删除的起始模板。

**完成标准**：示例包存在，通过根入口点暴露行为，将 `impl` 隐藏在子文件夹中。

#### 步骤 6：证明规则生效

这是整个技能的完成标准：不能在实际违规上报错的配置毫无价值。

1. 运行 `lint:boundaries`。在干净的示例上**必须通过**。
2. 临时向 `tests/example.test.ts` 添加一个深层导入（如 `import { thing } from "../lib/impl"`）。再次运行 `lint:boundaries`；**必须失败**，报错 `tests-through-entrypoints`。
3. 撤回深层导入。再次运行，**必须通过**。

**完成标准**：观察到通过 → 深层导入时失败 → 再次通过。如果步骤 2 没有失败，说明规则接线不正确，需要在完成前修复。

#### 步骤 7：文档化约定

在**包文件夹内**编写 `README.md`（`<packages-root>/README.md`，紧邻其管理的包），内容包括：`src/packages/<name>/` 布局（入口点在根目录，`lib/` 放实现，`tests/` 放测试）、"仅通过包的入口点（根文件）导入"、以及如何运行 `lint:boundaries`。**明确地不鼓励桶文件**：暴露多个小型入口点，而不是通过一个索引文件重新导出整个子树。保持为可复制的代码片段外加四条规则各一段。

然后在仓库的智能体指令文件（如果已有 `CLAUDE.md` 则用它，否则用 `AGENTS.md`；如果两者都不存在则创建 `AGENTS.md`）中添加一个**上下文指针**指向该文件。一行即可，例如 `Packages are deep modules: see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` 这使智能体能够发现边界规则，而不是在不知情的情况下违反它。

**完成标准**：`<packages-root>/README.md` 存在且不鼓励桶文件，仓库的 `CLAUDE.md`/`AGENTS.md` 链接到它。

### 注意事项

- 配置中的 `$1` 反向引用（dependency-cruiser 的分组匹配机制）使得包可以访问自己的内部实现，而外部代码不能。不要将其展开为每个包单独的规则。
- 公开与私有的判断标准是**深度**：包的根文件是入口点；子文件夹中的任何内容均为私有。约定的子文件夹是 `lib/`（实现）和 `tests/`，但规则没有硬编码它们：任何子文件夹都是私有的，因此新增文件夹不需要修改配置。添加入口点只需添加一个根文件（而非桶文件）。
- 包是**扁平的**：根目录下只有一层直接子包。包的内部可以任意嵌套深度；包不能包含另一个包。
- 使用 `.cjs`（而非 `.js`），这样即使在 `"type": "module"` 的仓库中，配置中的 `module.exports` 也能正常工作。

---

## 第二部分：原文与翻译对照

<!--原文-->
> Make every package in this repo a **deep module**: a lot of behaviour behind a small interface. A package's public surface is its **entry points** (the files at the package root), and everything in its subfolders is hidden. This skill installs dependency-cruiser and the rules that make the entry points the only way in, then proves the rules bite.
<!--翻译-->
> 使本仓库中的每个包都成为**深度模块**：大量行为隐藏在小接口之后。包的公开表面是其**入口点**（包根目录下的文件），子文件夹中的一切对外隐藏。本技能安装 dependency-cruiser 以及使入口点成为唯一访问途径的规则，然后证明这些规则确实生效。

<!--原文-->
> The public surface is the package's **root files**, not one designated `index.ts`. By convention implementation lives in `lib/` and tests in `tests/`, giving every package the same two-folder shape. The rule itself is general, though: *anything* in *any* subfolder is private, so you never extend the config to add a folder.
<!--翻译-->
> 公开表面是包**根目录下的所有文件**，而非指定一个 `index.ts`。按照约定，实现放在 `lib/`，测试放在 `tests/`，使每个包都保持相同的两层文件夹结构。但规则本质上是通用的：**任何**子文件夹中的**任何**内容均为私有，因此你永远不需要扩展配置来添加文件夹。

<!--原文-->
> **Entry-point boundary**: code outside a package (app code or another package) may import only that package's entry points (its root files), never anything in its subfolders.
<!--翻译-->
> **入口点边界**：包外代码（应用代码或其他包）只能导入该包的入口点（根文件），绝不能导入其子文件夹中的任何内容。

<!--原文-->
> **Intra-package freedom**: a package's own files import each other freely.
<!--翻译-->
> **包内自由**：包内文件之间可以自由互导。

<!--原文-->
> **Tests through the entry points**: files under `<pkg>/tests/` may import any package's entry points and their own `tests/` fixtures, but never any package's subfolder internals (not even their own). Integration tests across packages are fine; deep imports are not.
<!--翻译-->
> **通过入口点测试**：`<pkg>/tests/` 下的文件可以导入任何包的入口点及其自己的 `tests/` 固定数据文件，但绝不能导入任何包的子文件夹内部实现（即使是自己的也不行）。跨包集成测试可以；深层导入不可以。

<!--原文-->
> **No cycles**: no dependency cycles.
<!--翻译-->
> **禁止循环**：不允许任何依赖循环。

<!--原文-->
> **Entry points, not a barrel.** Because the public surface is *every* root file, a package can expose several small entry points (`index.ts`, `client.ts`, `server.ts`) instead of funnelling everything through one giant `index.ts`. Barrel files that re-export a whole subtree are discouraged; keep entry points small and hide implementation in subfolders.
<!--翻译-->
> **入口点，而非桶文件。** 由于公开表面是*每个*根文件，包可以暴露多个小型入口点（`index.ts`、`client.ts`、`server.ts`），而不是将所有内容通过一个巨大的 `index.ts` 汇集。不鼓励重新导出整个子树的桶文件；保持入口点精简，将实现隐藏在子文件夹中。

<!--原文-->
> Layering (which packages may depend on which) is a *different* concern and is left as a commented stub in the config for this repo to fill in.
<!--翻译-->
> 分层（哪些包可以依赖哪些包）是*另一个*不同的问题，在配置中以注释桩的形式留给仓库自行填写。

<!--原文-->
> **Detect the environment**: Package manager (`pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `bun.lockb` → bun, else npm). Use it for every command below. Packages root: if `src/` exists use `src/packages`, else `packages`. Confirm with the user if the repo already has a different obvious convention. Existing config: check for a `.dependency-cruiser.*` file. If one exists, do **not** overwrite it: merge the four rules and the options in, and tell the user what you added.
<!--翻译-->
> **检测环境**：包管理器（`pnpm-lock.yaml` → pnpm，`yarn.lock` → yarn，`bun.lockb` → bun，否则 npm）。用于后续所有命令。包根目录：如果存在 `src/`，则使用 `src/packages`，否则用 `packages`。如果仓库已有不同的明显约定，与用户确认。现有配置：检查是否存在 `.dependency-cruiser.*` 文件。如果存在，**不要**覆盖它：将四条规则和选项合并进去，并告知用户添加了什么。

<!--原文-->
> Install `dependency-cruiser` as a devDependency with the detected package manager.
<!--翻译-->
> 使用已检测到的包管理器将 `dependency-cruiser` 安装为 devDependency。

<!--原文-->
> Copy `dependency-cruiser.config.cjs` to the repo root as `.dependency-cruiser.cjs`. Set `PACKAGES_ROOT` to the root detected in step 1. The rules are path-depth based and extension-agnostic, so nothing else needs adapting.
<!--翻译-->
> 将 `dependency-cruiser.config.cjs` 复制到仓库根目录，命名为 `.dependency-cruiser.cjs`。将 `PACKAGES_ROOT` 设置为步骤 1 中检测到的根目录。规则基于路径深度且与扩展名无关，因此无需其他调整。

<!--原文-->
> Add a `lint:boundaries` script: `depcruise <packages-root>` (or `depcruise src`). Fold it into the repo's umbrella check command, the one that already runs typecheck (e.g. a `check` / `ci` / `validate` script). Do **not** touch `tsconfig` or add path aliases. If there is no umbrella script, add `lint:boundaries` and tell the user to include it in CI.
<!--翻译-->
> 添加 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。将其整合到仓库的伞形检查命令中，该命令应已包含类型检查（例如 `check`、`ci` 或 `validate` 脚本）。**不要**修改 `tsconfig` 或添加路径别名。如果没有伞形脚本，添加 `lint:boundaries` 并告知用户将其纳入 CI。

<!--原文-->
> Create a committed `<packages-root>/example/` as a copy-me template. `index.ts` is an entry point. Export one function that delegates to an internal file (so the package is visibly *deep*, not a pass-through). `lib/impl.ts`: an internal file in a **subfolder**, imported by `index.ts`, not reachable from outside. `tests/example.test.ts` imports **only** `../index` (an entry point) and asserts against the public function.
<!--翻译-->
> 创建一个已提交的 `<packages-root>/example/` 作为可复制模板。`index.ts` 为入口点。导出一个委托给内部文件的函数（使包明显是*深层*的，而非全透传）。`lib/impl.ts`：**子文件夹**中的内部文件，由 `index.ts` 导入，外部无法访问。`tests/example.test.ts` **仅**导入 `../index`（一个入口点），并对公开函数进行断言。

<!--原文-->
> This is the completion criterion for the whole skill: a config that doesn't fail on a violation is worthless. Run `lint:boundaries`. It must **pass** on the clean example. Temporarily add a deep import to `tests/example.test.ts` (e.g. `import { thing } from "../lib/impl"`). Run `lint:boundaries` again; it must **fail** with `tests-through-entrypoints`. Revert the deep import. Run once more, and it must **pass**.
<!--翻译-->
> 这是整个技能的完成标准：不能在实际违规上报错的配置毫无价值。运行 `lint:boundaries`。在干净的示例上**必须通过**。临时向 `tests/example.test.ts` 添加一个深层导入（如 `import { thing } from "../lib/impl"`）。再次运行 `lint:boundaries`；**必须失败**，报错 `tests-through-entrypoints`。撤回深层导入。再次运行，**必须通过**。

<!--原文-->
> Write a `README.md` **in the packages folder** (`<packages-root>/README.md`, next to the packages it governs) covering: the `src/packages/<name>/` layout (entry points at the root, `lib/` for implementation, `tests/` for tests), "import only through a package's entry points (its root files)", and how to run `lint:boundaries`. **Discourage barrel files** explicitly: expose several small entry points instead of re-exporting a whole subtree through one index.
<!--翻译-->
> 在**包文件夹内**编写 `README.md`（`<packages-root>/README.md`，紧邻其管理的包），内容包括：`src/packages/<name>/` 布局（入口点在根目录，`lib/` 放实现，`tests/` 放测试）、"仅通过包的入口点（根文件）导入"、以及如何运行 `lint:boundaries`。**明确地不鼓励桶文件**：暴露多个小型入口点，而不是通过一个索引文件重新导出整个子树。

<!--原文-->
> Then add a **context pointer** to it from the repo's agent-instructions file (`CLAUDE.md` if present, else `AGENTS.md`, creating `AGENTS.md` if neither exists). One line is enough, e.g. `Packages are deep modules: see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` This is what makes an agent discover the boundary rule instead of tripping over it.
<!--翻译-->
> 然后在仓库的智能体指令文件（如果已有 `CLAUDE.md` 则用它，否则用 `AGENTS.md`；如果两者都不存在则创建 `AGENTS.md`）中添加一个**上下文指针**指向该文件。一行即可，例如 `Packages are deep modules: see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` 这使智能体能够发现边界规则，而不是在不知情的情况下违反它。

<!--原文-->
> The config's `$1` back-references (dependency-cruiser's group matching) are what let a package reach its own internals while outsiders can't. Don't flatten them into separate per-package rules.
<!--翻译-->
> 配置中的 `$1` 反向引用（dependency-cruiser 的分组匹配机制）使得包可以访问自己的内部实现，而外部代码不能。不要将其展开为每个包单独的规则。

<!--原文-->
> Public vs private is decided by **depth**: a package's root files are entry points; anything in a subfolder is private. The conventional subfolders are `lib/` (implementation) and `tests/`, but the rule doesn't hardcode them: any subfolder is private, so a new folder never needs a config change. Adding an entry point is just adding a root file (no barrel).
<!--翻译-->
> 公开与私有的判断标准是**深度**：包的根文件是入口点；子文件夹中的任何内容均为私有。约定的子文件夹是 `lib/`（实现）和 `tests/`，但规则没有硬编码它们：任何子文件夹都是私有的，因此新增文件夹不需要修改配置。添加入口点只需添加一个根文件（而非桶文件）。

<!--原文-->
> Packages are **flat**: one tier of immediate children under the root. A package's internals may nest as deep as you like; a package may not contain another package.
<!--翻译-->
> 包是**扁平的**：根目录下只有一层直接子包。包的内部可以任意嵌套深度；包不能包含另一个包。

<!--原文-->
> Use `.cjs` (not `.js`) so the config's `module.exports` works even in `"type": "module"` repos.
<!--翻译-->
> 使用 `.cjs`（而非 `.js`），这样即使在 `"type": "module"` 的仓库中，配置中的 `module.exports` 也能正常工作。
