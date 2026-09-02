# scaffold-exercises — 创建课程练习目录结构

## 第一部分：详细总结

### 概述

`scaffold-exercises` 是一个自动化 Skill，用于创建通过 `pnpm ai-hero-cli internal lint` 验证的课程练习目录结构，然后用 `git commit` 提交。适用于需要批量搭建课程章节和练习题骨架的教学内容开发场景。

### 目录命名规范

- **章节（Sections）**：格式为 `XX-章节名/`，位于 `exercises/` 目录下（如 `01-retrieval-skill-building`）
- **练习（Exercises）**：格式为 `XX.YY-练习名/`，位于章节目录下（如 `01.03-retrieval-with-bm25`）
- 章节编号为 `XX`，练习编号为 `XX.YY`
- 名称使用短横线命名法（dash-case）：全小写、连字符分隔

### 练习变体

每个练习至少需要以下子文件夹之一：

- `problem/` —— 学生工作区，含 TODO 标记
- `solution/` —— 参考实现
- `explainer/` —— 概念材料，无 TODO

创建骨架时默认使用 `explainer/`，除非计划另有指定。

### 必需文件

每个子文件夹（`problem/`、`solution/`、`explainer/`）需要一个 `readme.md`，要求：
- **非空**（必须有实际内容，即使只有一行标题也可以）
- 无断链

创建骨架时，创建一个最小化的 readme，包含标题和描述：

```md
# 练习标题
描述在此
```

如果子文件夹有代码，还需要一个超过一行的 `main.ts`。但对于骨架，仅 readme 的练习是可以的。

### 工作流程

1. **解析计划**：提取章节名称、练习名称和变体类型
2. **创建目录**：使用 `mkdir -p` 为每个路径创建目录
3. **创建骨架 readme**：为每个变体文件夹创建一个带有标题的 `readme.md`
4. **运行 Lint**：`pnpm ai-hero-cli internal lint` 验证
5. **修复错误**：迭代直到 Lint 通过

### Lint 规则摘要

Linter (`pnpm ai-hero-cli internal lint`) 检查：

- 每个练习有子文件夹（`problem/`、`solution/`、`explainer/`）
- 至少存在 `problem/`、`explainer/` 或 `explainer.1/` 之一
- 主子文件夹中存在非空的 `readme.md`
- 无 `.gitkeep` 文件
- 无 `speaker-notes.md` 文件
- readme 中无断链
- readme 中无 `pnpm run exercise` 命令
- 每个子文件夹需要 `main.ts`，除非是纯 readme 练习

### 移动/重命名练习

当重新编号或移动练习时：

1. 使用 `git mv`（不是 `mv`）重命名目录，保留 Git 历史
2. 更新数字前缀以维持顺序
3. 移动后重新运行 Lint

---

## 第二部分：原文与翻译

<!--原文-->
> # Scaffold Exercises
> Create exercise directory structures that pass `pnpm ai-hero-cli internal lint`, then commit with `git commit`.

<!--翻译-->
> # 创建练习骨架
> 创建通过 `pnpm ai-hero-cli internal lint` 验证的练习目录结构，然后用 `git commit` 提交。

<!--原文-->
> ## Directory naming
> - **Sections**: `XX-section-name/` inside `exercises/` (e.g., `01-retrieval-skill-building`)
> - **Exercises**: `XX.YY-exercise-name/` inside a section (e.g., `01.03-retrieval-with-bm25`)
> - Section number = `XX`, exercise number = `XX.YY`
> - Names are dash-case (lowercase, hyphens)

<!--翻译-->
> ## 目录命名
> - **章节**：`XX-章节名/`，位于 `exercises/` 目录下（如 `01-retrieval-skill-building`）
> - **练习**：`XX.YY-练习名/`，位于章节目录下（如 `01.03-retrieval-with-bm25`）
> - 章节编号 = `XX`，练习编号 = `XX.YY`
> - 名称使用短横线命名法（全小写、连字符分隔）

<!--原文-->
> ## Exercise variants
> Each exercise needs at least one of these subfolders:
> - `problem/` - student workspace with TODOs
> - `solution/` - reference implementation
> - `explainer/` - conceptual material, no TODOs
> When stubbing, default to `explainer/` unless the plan specifies otherwise.

<!--翻译-->
> ## 练习变体
> 每个练习至少需要以下子文件夹之一：
> - `problem/` —— 学生工作区，含 TODO 标记
> - `solution/` —— 参考实现
> - `explainer/` —— 概念材料，无 TODO
> 创建骨架时默认使用 `explainer/`，除非计划另有指定。

<!--原文-->
> ## Required files
> Each subfolder (`problem/`, `solution/`, `explainer/`) needs a `readme.md` that:
> - Is **not empty** (must have real content, even a single title line works)
> - Has no broken links
> When stubbing, create a minimal readme with a title and a description:
> ```md
> # Exercise Title
> Description here
> ```
> If the subfolder has code, it also needs a `main.ts` (>1 line). But for stubs, a readme-only exercise is fine.

<!--翻译-->
> ## 必需文件
> 每个子文件夹（`problem/`、`solution/`、`explainer/`）需要一个 `readme.md`，要求：
> - **非空**（必须有实际内容，即使只有一行标题也可以）
> - 无断链
> 创建骨架时，创建一个最小化的 readme，包含标题和描述：
> ```md
> # 练习标题
> 描述在此
> ```
> 如果子文件夹有代码，还需要一个超过一行的 `main.ts`。但对于骨架，仅 readme 的练习是可以的。

<!--原文-->
> ## Workflow
> 1. **Parse the plan** - extract section names, exercise names, and variant types
> 2. **Create directories** - `mkdir -p` for each path
> 3. **Create stub readmes** - one `readme.md` per variant folder with a title
> 4. **Run lint** - `pnpm ai-hero-cli internal lint` to validate
> 5. **Fix any errors** - iterate until lint passes

<!--翻译-->
> ## 工作流程
> 1. **解析计划**——提取章节名称、练习名称和变体类型
> 2. **创建目录**——使用 `mkdir -p` 为每个路径创建目录
> 3. **创建骨架 readme**——为每个变体文件夹创建一个带有标题的 `readme.md`
> 4. **运行 Lint**——`pnpm ai-hero-cli internal lint` 验证
> 5. **修复错误**——迭代直到 Lint 通过

<!--原文-->
> ## Lint rules summary
> The linter checks:
> - Each exercise has subfolders (`problem/`, `solution/`, `explainer/`)
> - At least one of `problem/`, `explainer/`, or `explainer.1/` exists
> - `readme.md` exists and is non-empty in the primary subfolder
> - No `.gitkeep` files
> - No `speaker-notes.md` files
> - No broken links in readmes
> - No `pnpm run exercise` commands in readmes
> - `main.ts` required per subfolder unless it's readme-only

<!--翻译-->
> ## Lint 规则摘要
> Linter 检查：
> - 每个练习有子文件夹（`problem/`、`solution/`、`explainer/`）
> - 至少存在 `problem/`、`explainer/` 或 `explainer.1/` 之一
> - 主子文件夹中存在非空的 `readme.md`
> - 无 `.gitkeep` 文件
> - 无 `speaker-notes.md` 文件
> - readme 中无断链
> - readme 中无 `pnpm run exercise` 命令
> - 每个子文件夹需要 `main.ts`，除非是纯 readme 练习

<!--原文-->
> ## Moving/renaming exercises
> When renumbering or moving exercises:
> 1. Use `git mv` (not `mv`) to rename directories - preserves git history
> 2. Update the numeric prefix to maintain order
> 3. Re-run lint after moves

<!--翻译-->
> ## 移动/重命名练习
> 当重新编号或移动练习时：
> 1. 使用 `git mv`（不是 `mv`）重命名目录——保留 Git 历史
> 2. 更新数字前缀以维持顺序
> 3. 移动后重新运行 Lint
