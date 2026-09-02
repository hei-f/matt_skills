# setup-pre-commit — 设置 Pre-commit 钩子

## 第一部分：详细总结

### 概述

`setup-pre-commit` 是一个配置型 Skill，用于在当前仓库中设置 Husky pre-commit 钩子，配合 lint-staged（运行 Prettier）、类型检查和测试。适用于希望添加提交时代码格式化、类型检查和测试验证的开发者。

### 设置内容

该 Skill 自动配置以下内容：
- **Husky** pre-commit 钩子框架
- **lint-staged** 在所有暂存文件上运行 Prettier
- **Prettier** 配置文件（如果缺失则创建默认配置）
- **typecheck** 和 **test** 脚本集成到 pre-commit 钩子中

### 安装步骤

#### 1. 检测包管理器

检查锁文件：`package-lock.json`（npm）、`pnpm-lock.yaml`（pnpm）、`yarn.lock`（yarn）、`bun.lockb`（bun）。使用检测到的包管理器，不明时默认 npm。

#### 2. 安装依赖

以 devDependencies 安装 `husky`、`lint-staged` 和 `prettier`。

#### 3. 初始化 Husky

运行 `npx husky init`。这将创建 `.husky/` 目录并在 `package.json` 中添加 `"prepare": "husky"` 脚本。

#### 4. 创建 `.husky/pre-commit`

写入 pre-commit 钩子文件（Husky v9+ 不需要 shebang）：

```
npx lint-staged
npm run typecheck
npm run test
```

**适配**：将 `npm` 替换为检测到的包管理器。如果仓库的 `package.json` 中没有 `typecheck` 或 `test` 脚本，省略对应行并告知用户。

#### 5. 创建 `.lintstagedrc`

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

`--ignore-unknown` 标志跳过 Prettier 无法解析的文件（如图片等）。

#### 6. 创建 `.prettierrc`（如不存在）

仅在不存在 Prettier 配置时创建。默认配置：使用空格缩进、tab 宽度 2、每行 80 字符、双引号、ES5 尾逗号、分号、箭头函数始终加括号。

#### 7. 验证

检查清单：
- `.husky/pre-commit` 存在且可执行
- `.lintstagedrc` 存在
- `package.json` 中的 `prepare` 脚本为 `"husky"`
- `prettier` 配置文件存在
- 运行 `npx lint-staged` 验证其正常工作

#### 8. 提交

暂存所有更改/创建的文件并提交，提交信息：`Add pre-commit hooks (husky + lint-staged + prettier)`。这将通过新的 pre-commit 钩子运行，是一次良好的冒烟测试。

### 注意事项

- Husky v9+ 不需要在钩子文件中写 shebang
- `prettier --ignore-unknown` 跳过 Prettier 无法解析的文件
- pre-commit 钩子先运行 lint-staged（快，仅暂存文件），然后运行完整类型检查和测试

---

## 第二部分：原文与翻译

<!--原文-->
> # Setup Pre-Commit Hooks
> ## What This Sets Up
> - **Husky** pre-commit hook
> - **lint-staged** running Prettier on all staged files
> - **Prettier** config (if missing)
> - **typecheck** and **test** scripts in the pre-commit hook

<!--翻译-->
> # 设置 Pre-commit 钩子
> ## 设置内容
> - **Husky** pre-commit 钩子
> - **lint-staged** 在所有暂存文件上运行 Prettier
> - **Prettier** 配置（如果缺失）
> - pre-commit 钩子中的 **typecheck** 和 **test** 脚本

<!--原文-->
> ### 1. Detect package manager
> Check for `package-lock.json` (npm), `pnpm-lock.yaml` (pnpm), `yarn.lock` (yarn), `bun.lockb` (bun). Use whichever is present. Default to npm if unclear.

<!--翻译-->
> ### 1. 检测包管理器
> 检查 `package-lock.json`（npm）、`pnpm-lock.yaml`（pnpm）、`yarn.lock`（yarn）、`bun.lockb`（bun）。使用存在的那个。不明时默认 npm。

<!--原文-->
> ### 2. Install dependencies
> Install as devDependencies: `husky lint-staged prettier`

<!--翻译-->
> ### 2. 安装依赖
> 以 devDependencies 安装：`husky lint-staged prettier`

<!--原文-->
> ### 3. Initialize Husky
> `npx husky init`
> This creates `.husky/` dir and adds `prepare: "husky"` to package.json.

<!--翻译-->
> ### 3. 初始化 Husky
> `npx husky init`
> 这将创建 `.husky/` 目录并在 package.json 中添加 `prepare: "husky"`。

<!--原文-->
> ### 4. Create `.husky/pre-commit`
> Write this file (no shebang needed for Husky v9+):
> ```
> npx lint-staged
> npm run typecheck
> npm run test
> ```
> **Adapt**: Replace `npm` with detected package manager. If repo has no `typecheck` or `test` script in package.json, omit those lines and tell the user.

<!--翻译-->
> ### 4. 创建 `.husky/pre-commit`
> 写入此文件（Husky v9+ 不需要 shebang）：
> ```
> npx lint-staged
> npm run typecheck
> npm run test
> ```
> **适配**：将 `npm` 替换为检测到的包管理器。如果仓库的 package.json 中没有 `typecheck` 或 `test` 脚本，省略这些行并告知用户。

<!--原文-->
> ### 5. Create `.lintstagedrc`
> ```json
> { "*": "prettier --ignore-unknown --write" }
> ```

<!--翻译-->
> ### 5. 创建 `.lintstagedrc`
> ```json
> { "*": "prettier --ignore-unknown --write" }
> ```

<!--原文-->
> ### 6. Create `.prettierrc` (if missing)
> Only create if no Prettier config exists. Use these defaults:
> ```json
> {
>   "useTabs": false,
>   "tabWidth": 2,
>   "printWidth": 80,
>   "singleQuote": false,
>   "trailingComma": "es5",
>   "semi": true,
>   "arrowParens": "always"
> }
> ```

<!--翻译-->
> ### 6. 创建 `.prettierrc`（如不存在）
> 仅在不存在 Prettier 配置时创建。使用以下默认值：
> ```json
> {
>   "useTabs": false,
>   "tabWidth": 2,
>   "printWidth": 80,
>   "singleQuote": false,
>   "trailingComma": "es5",
>   "semi": true,
>   "arrowParens": "always"
> }
> ```

<!--原文-->
> ### 7. Verify
> - [ ] `.husky/pre-commit` exists and is executable
> - [ ] `.lintstagedrc` exists
> - [ ] `prepare` script in package.json is `"husky"`
> - [ ] `prettier` config exists
> - [ ] Run `npx lint-staged` to verify it works

<!--翻译-->
> ### 7. 验证
> - [ ] `.husky/pre-commit` 存在且可执行
> - [ ] `.lintstagedrc` 存在
> - [ ] package.json 中的 `prepare` 脚本为 `"husky"`
> - [ ] `prettier` 配置存在
> - [ ] 运行 `npx lint-staged` 验证其正常工作

<!--原文-->
> ### 8. Commit
> Stage all changed/created files and commit with message: `Add pre-commit hooks (husky + lint-staged + prettier)`
> This will run through the new pre-commit hooks: a good smoke test that everything works.

<!--翻译-->
> ### 8. 提交
> 暂存所有更改/创建的文件并提交，提交信息：`Add pre-commit hooks (husky + lint-staged + prettier)`
> 这将通过新的 pre-commit 钩子运行：一次良好的冒烟测试。

<!--原文-->
> ## Notes
> - Husky v9+ doesn't need shebangs in hook files
> - `prettier --ignore-unknown` skips files Prettier can't parse (images, etc.)
> - The pre-commit runs lint-staged first (fast, staged-only), then full typecheck and tests

<!--翻译-->
> ## 注意事项
> - Husky v9+ 不需要在钩子文件中写 shebang
> - `prettier --ignore-unknown` 跳过 Prettier 无法解析的文件（图片等）
> - pre-commit 钩子先运行 lint-staged（快，仅暂存文件），然后运行完整类型检查和测试
