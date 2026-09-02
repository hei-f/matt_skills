# git-guardrails-claude-code — Git 安全护栏（Claude Code 版）

## 第一部分：详细总结

### 概述

`git-guardrails-claude-code` 是一个配置型 Skill，用于在 Claude Code 中设置 PreToolUse Hook，在危险 Git 命令执行前拦截并阻止它们。适用于希望防止破坏性 Git 操作（如强制推送、硬重置、清理等）的开发者。该 Skill 通过部署一个 Shell 脚本来检查每条 Bash 命令，匹配危险模式时阻止执行。

### 被阻止的命令

该 Hook 默认阻止以下 Git 操作：

- `git push`（包括 `--force` 的所有变体）
- `git reset --hard`
- `git clean -f` / `git clean -fd`
- `git branch -D`
- `git checkout .` / `git restore .`

当命令被阻止时，Claude 会看到一条提示消息，告知它无权执行这些命令。

### 安装步骤

#### 1. 询问作用域

首先询问用户：仅本项目安装（`.claude/settings.json`）还是全局安装（`~/.claude/settings.json`）？

#### 2. 复制 Hook 脚本

捆绑的 Hook 脚本位于 `scripts/block-dangerous-git.sh`。根据作用域复制到目标位置：

- **项目级**：`.claude/hooks/block-dangerous-git.sh`
- **全局**：`~/.claude/hooks/block-dangerous-git.sh`

使用 `chmod +x` 赋予可执行权限。

#### 3. 将 Hook 添加到设置文件

在对应作用域的 settings.json 中配置 PreToolUse Hook。配置 JSON 中指定 `matcher` 为 `"Bash"`，`type` 为 `"command"`。如果设置文件已存在，将 Hook 合并到现有的 `hooks.PreToolUse` 数组中，不要覆盖其他设置。

#### 4. 询问自定义需求

询问用户是否需要添加或移除阻止列表中的某些模式。根据需求编辑已复制的脚本。

#### 5. 验证

运行快速测试，将模拟的 Git 命令通过管道传给脚本。脚本应以退出码 2 退出，并向 stderr 输出 BLOCKED 消息。

### Hook 脚本工作方式

脚本读取标准输入中的 JSON，使用 `jq` 提取 `tool_input.command` 字段，然后将其与危险模式列表进行匹配。危险模式包括：`git push`、`git reset --hard`、`git clean -fd`、`git clean -f`、`git branch -D`、`git checkout \.`、`git restore \.`、`push --force`、`reset --hard`。匹配成功时向 stderr 输出阻止消息并以退出码 2 退出；否则以退出码 0 退出，允许命令执行。

---

## 第二部分：原文与翻译

<!--原文-->
> # Setup Git Guardrails
> Sets up a PreToolUse hook that intercepts and blocks dangerous git commands before Claude executes them.

<!--翻译-->
> # 设置 Git 安全护栏
> 设置一个 PreToolUse Hook，在 Claude 执行危险 Git 命令之前拦截并阻止它们。

<!--原文-->
> ## What Gets Blocked
> - `git push` (all variants including `--force`)
> - `git reset --hard`
> - `git clean -f` / `git clean -fd`
> - `git branch -D`
> - `git checkout .` / `git restore .`
> When blocked, Claude sees a message telling it that it does not have authority to access these commands.

<!--翻译-->
> ## 被阻止的命令
> - `git push`（包括 `--force` 的所有变体）
> - `git reset --hard`
> - `git clean -f` / `git clean -fd`
> - `git branch -D`
> - `git checkout .` / `git restore .`
> 当被阻止时，Claude 会看到一条消息，告知它无权执行这些命令。

<!--原文-->
> ## Steps
> ### 1. Ask scope
> Ask the user: install for **this project only** (`.claude/settings.json`) or **all projects** (`~/.claude/settings.json`)?

<!--翻译-->
> ## 步骤
> ### 1. 询问作用域
> 询问用户：仅**此项目**安装（`.claude/settings.json`）还是**所有项目**安装（`~/.claude/settings.json`）？

<!--原文-->
> ### 2. Copy the hook script
> The bundled script is at: [scripts/block-dangerous-git.sh](scripts/block-dangerous-git.sh)
> Copy it to the target location based on scope:
> - **Project**: `.claude/hooks/block-dangerous-git.sh`
> - **Global**: `~/.claude/hooks/block-dangerous-git.sh`
> Make it executable with `chmod +x`.

<!--翻译-->
> ### 2. 复制 Hook 脚本
> 捆绑的脚本位于：[scripts/block-dangerous-git.sh](scripts/block-dangerous-git.sh)
> 根据作用域复制到目标位置：
> - **项目级**：`.claude/hooks/block-dangerous-git.sh`
> - **全局**：`~/.claude/hooks/block-dangerous-git.sh`
> 使用 `chmod +x` 赋予可执行权限。

<!--原文-->
> ### 3. Add hook to settings
> Add to the appropriate settings file. If the settings file already exists, merge the hook into the existing `hooks.PreToolUse` array. Don't overwrite other settings.

<!--翻译-->
> ### 3. 将 Hook 添加到设置文件
> 添加到对应的设置文件。如果设置文件已存在，将 Hook 合并到现有的 `hooks.PreToolUse` 数组中。不要覆盖其他设置。

<!--原文-->
> ### 4. Ask about customization
> Ask if user wants to add or remove any patterns from the blocked list. Edit the copied script accordingly.

<!--翻译-->
> ### 4. 询问自定义需求
> 询问用户是否需要添加或移除阻止列表中的某些模式。根据需求编辑已复制的脚本。

<!--原文-->
> ### 5. Verify
> Run a quick test:
> ```bash
> echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>
> ```
> Should exit with code 2 and print a BLOCKED message to stderr.

<!--翻译-->
> ### 5. 验证
> 运行快速测试：
> ```bash
> echo '{"tool_input":{"command":"git push origin main"}}' | <路径-到-脚本>
> ```
> 应以退出码 2 退出，并向 stderr 输出 BLOCKED 消息。
