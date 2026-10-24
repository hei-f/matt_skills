# wizard -- 交互式 Bash 向导生成器

## 第一部分：详细总结

### 概述

`wizard` 是一个**模型调用（model-invoked）**的 Skill，属于 `engineering/` 桶。它生成交互式 Bash 脚本，引导**人类**逐步完成只有他们能执行的步骤——配置第三方服务、设置凭据/CI 密钥、浏览不熟悉的仪表板、运行一次性迁移或切换等。

关键设计原则：**AI Agent 只编写脚本，从不运行它**。脚本会打开浏览器并阻塞等待人工输入，因此只有人类能够运行。这在根本上保证了敏感信息（如 API 密钥）不会进入模型上下文——脚本在用户本机运行时通过隐藏终端输入捕获密钥并直接写入 `.env` 或 `gh secret`，模型与脚本运行时没有连接。

该 Skill 在 v1.2 版本从 `in-progress/` 毕业，正式纳入插件发布集。

### 核心定位与适用场景

`wizard` 位于自动化与手动操作的交界处：当自动化的尽头需要人类介入时，它生成一个**可执行程序**来驱动这个过程，而非在聊天中丢出一串编号步骤（那些步骤会滚出屏幕并腐化）。它的设计哲学是：与其让 README 中的初始化文档因无人维护而腐化，不如写一个可执行版本，它不会安静地过时。

| 场景 | Wizard 的作用 |
|---|---|
| 新开发者需要配置六个服务才能启动应用 | 依次打开每个仪表板，捕获密钥，写入 `.env` 和 CI |
| 一次性迁移需要按特定顺序切换开关 | 通过确认门禁串联不可逆步骤 |
| 项目需从状态 A 转移到状态 B | 引导过渡，报告无法完成的部分 |
| 正打算将这些步骤写入 README | 改为编写可执行版本，它不会安静地腐化 |

**不适合的场景**：用它来"决定"构建什么——那是 `grill-with-docs` 和 `to-spec` 的职责。

### 模板驱动架构

wizard 的核心是一个固定的**模板库** `template.sh`。该文件分为两部分：

**标记之上的库（Library above STAGES marker）**：固定代码，在所有 wizard 脚本中完全一致，禁止手动编辑。它提供了完整、精致的 UX：

- **分阶段进度**：`stage "name"` 清屏，宣告当前阶段及 "Stage X/Y" 进度；`TOTAL_STAGES` 变量驱动显示
- **确认关卡**：`pause` 等待人类完成浏览器操作后按回车；`confirm "question"` 提供 y/N 门禁，用于不可逆操作前
- **跨平台 URL 打开**：`open_url URL` 自动检测 macOS（`open`）、Linux（`xdg-open`）、Windows（`explorer.exe`）、WSL（`wslview`）；打开失败或无可用打开器时打印警告并继续运行，而非崩溃
- **隐藏密钥输入**：`ask_secret KEY "prompt"` 使用 `read -rs`，敏感内容永不会回显到终端滚动历史中
- **幂等 `.env` 写入**：`write_env KEY VALUE` 执行 upsert——不存在则以 0600 权限创建 `.env`；值以单引号包裹写入（`KEY='VALUE'`，内嵌单引号转义），空格、`#`、`$` 等字符可原样保留；已存在的同名键会被替换（保留无关节）；对已存在的 `.env` 穿过符号链接写入并保留其权限模式；`_existing KEY` 读取现有值（还原单引号包裹），供重跑时作为默认值
- **GitHub secrets/variables 写入**：`set_secret NAME VALUE` 封装 `gh secret set`；`set_var NAME VALUE` 封装 `gh variable set`。若 `gh` 不可用或未认证，则优雅降级为警告，并在 `SKIPPED` 数组中记录待手动完成的事项
- **结尾摘要**：`finish` 清屏后列出已写入的值、已设置的密钥，并单独列出因环境限制跳过的操作

**标记之下的阶段（Stages below marker）**：由 Agent 编写，是整个脚本中唯一可变的部分。每个 `stage` 是一个聚焦的任务单元，按依赖顺序排列。

Agent 的工作仅限**界定流程并编写 STAGES 标记之下的阶段内容**——库之上的一致性就是设计目标，是 wizard 体验稳定性的保证。

### 值落点矩阵

范围界定时，对每个捕获的值需确定其落点：

| 落点 | 适用场景 |
|---|---|
| 仅 `.env` (`write_env`) | 本地开发需要，CI 不需要 |
| GitHub secret (`set_secret`) | CI 需要读取，且该值是敏感信息 |
| GitHub variable (`set_var`) | CI 需要读取，且该值可公开 |
| `.env` 和 secret 两者 | 本地开发与 CI 都需要 |
| 无处（纯动作阶段） | 该阶段只是一个动作：切换开关、升级方案、点击确认按钮——没有值被捕获 |

### 四步生成流程（详细）

#### 步骤 1：界定流程范围（Scope the procedure）

在编写任何代码之前，Agent 必须先**主动阅读仓库**而非冷冰冰地向用户提问。扫描目标包括：

- **初始化场景**：`.env`、`.env.example`、`.env.*`、`README`、`docker-compose*`、框架配置（如 `next.config.js`、`vite.config.ts`）以及 `.github/workflows/*` 中的**每一个** `secrets.*` / `vars.*` 引用——每个引用都是 wizard 必须产出的一个值
- **迁移/过渡场景**：理解当前状态、目标状态以及两者之间的**不可逆操作**

然后向用户展示**有序的阶段列表**和每个阶段产出的值，并请求确认——用户可以添加、删除或调整顺序。

**完成标准**：每个阶段已按顺序命名；对于捕获的每个值，明确：
- (a) 人类从哪里获取它（仪表板 URL、命令行输出、配置文件）
- (b) 写入目标（`.env`、GitHub secret/variable、两者皆有，或无处）
- (c) 是密钥（`ask_secret` 隐藏输入）还是公开值（`ask` 明文输入）

#### 步骤 2：绘制每个阶段的旅程（Map each stage's journey）

为每个阶段编写人类遵循的**精确路径**。每个阶段应包含：
- 打开哪个 URL
- 在仪表板上点击/导航什么
- 值在哪个位置显示
- 填入哪个变量名

格式示例："Dashboard → Developers → API keys → Reveal test key → copy → paste as STRIPE_SECRET_KEY"。

**关键规则**：如果不确定当前 UI 或确切命令，必须**向用户确认或查阅文档**——绝不编造可能不存在的步骤。"Native" UI 的方法论参考：先 `open_url`，再描述操作；人类先看到页面，再被引导点击。

**完成标准**：每个阶段都追踪到具体的、陌生人可执行的指令——即使从未见过该仪表板的人也能按步骤完成。

#### 步骤 3：编写 Wizard 脚本（Author the wizard）

将 `template.sh` 复制到目标路径，用一个 `stage` 替换示例阶段，严格按依赖顺序排列（某阶段产出的值被后续阶段使用时，顺序至关重要）。

**可用辅助函数清单**：

| 函数 | 签名 / 说明 |
|---|---|
| `banner "title"` | 开场横幅，显示总阶段数和使用说明 |
| `stage "name"` | 清屏并宣告新阶段，显示 "Stage X/Y" 进度 |
| `say "..."` | 打印一条纯指令 |
| `step "..."` | 打印一条操作项（蓝色 `•` 前缀，表示浏览器操作） |
| `note "..."` | 打印一条灰色注释 |
| `warn "..."` | 打印一条黄色警告 |
| `open_url URL` | 在用户的浏览器中打开 URL |
| `ask KEY "prompt"` | 读取公开值到变量 `$KEY`；重跑时以已有 `.env` 值作为默认 |
| `ask_secret KEY "prompt"` | 读取密钥值（输入隐藏）；重跑时以已有 `.env` 值作为默认 |
| `write_env KEY VALUE` | 幂等 upsert KEY='VALUE' 到 `.env` 文件 |
| `set_secret NAME VALUE` | 通过 `gh` 写入 GitHub Actions secret |
| `set_var NAME VALUE` | 通过 `gh` 写入 GitHub Actions variable |
| `pause "msg"` | 等待人类按回车确认已完成手动部分 |
| `confirm "question"` | y/N 门禁，返回成功表示人类输入了 y/yes |
| `finish` | 结尾摘要：列出已写入的值、已设置的密钥、需手动完成的事项 |

**质量标准**：
1. 先 `open_url` 再 `ask` 值——绝不索要尚未发送去获取的内容
2. 对密钥始终使用 `ask_secret`
3. `write_env` 每个持久化值
4. `set_secret` 仅对 CI 真正需要的值（参考步骤 1 中的 `secrets.*` 引用）
5. 任何不可逆操作前使用 `confirm`
6. 设置 `TOTAL_STAGES` 等于你编写的阶段数
7. 每个 `stage` 聚焦一个任务——注意清屏意味着溢出屏幕的内容会丢失
8. 不要改动 `STAGES` 标记之上的库代码

#### 步骤 4：验证与交付（Verify and hand off）

- **语法检查**：运行 `bash -n <script>` 检查语法错误；若 `shellcheck` 可用则运行静态分析
- **权限设置**：`chmod +x <script>` 确保可执行
- **静态追踪**（绝不端到端运行，因为脚本会打开浏览器并阻塞在人工输入上）：确认步骤 1 中列出的每个值都在脚本中被捕获，并写入步骤 1 中指定的目标；确认每个 `set_secret NAME` 中的 `NAME` 与 `.github/workflows/` 中引用的 `${{ secrets.NAME }}` 完全匹配
- **交付说明**：告知用户脚本路径和运行方式。如果是可重复的初始化路径，提交到仓库并在 `README.md` 中通过链接引用——让下一个人运行脚本而非重新询问 AI

### 生命周期与状态管理

#### 默认一次性

| 脚本类型 | 处理方式 |
|---|---|
| 一次性迁移、个人配置、不可重复的过渡 | 保存到临时目录或 `scripts/` 路径，运行后删除 |
| 仓库中下一个人也需要运行的初始化路径 | 提交仓库，在 README 中链接——让他们运行脚本而非重新询问 AI |

#### 重跑与幂等性

wizard 通过 `.env` 文件实现幂等重跑：

1. **Ctrl-C 中断后重新运行**：每个 `ask`/`ask_secret` 调用后台的 `_existing KEY` 函数，查询 `.env` 中该键的当前值
2. **如果值已存在**：prompt 显示 `[Enter keeps current]`，按回车保留原值
3. **如果值不存在**：正常询问
4. **效果**：只需在对的阶段按回车跳过，仅重新输入错误的值

这是对"无回退按钮"限制的设计回应——阶段只向前运行，没有后退机制，但重跑廉价到可以接受。

#### 运行环境依赖

- **必需**：Bash（任何主流 Unix/macOS/WSL 环境）
- **可选**：`gh` CLI（仅在需要写入 GitHub secrets/variables 时需要；缺失时优雅降级为警告并记录待办）
- **自动适配**：彩色输出依赖 `tput`（终端支持），不支持时自动退化为纯文本

### 已知限制

1. **无回退按钮**：阶段只向前运行，错误回答在 stage 3 意味着 Ctrl-C 整个重跑。设计上是刻意选择——重跑廉价，默认值机制使重新走到出错阶段的代价很低（一路回车即可）
2. **`ask_secret` 箭头键不工作**：`ask` 使用 Readline（`read -e -r`），方向键可正常移动光标；但 `ask_secret` 的隐藏输入使用 `read -rs`（未启用 Readline），方向键插入转义序列（`^[[D` / `^[[C`）而非移动光标。退格键正常。解决方法：退格删除到错误位置，而非移动光标进入错误位置
3. **不检查第三方服务状态**：范围界定只读取本地 `.env` 文件判断值是否存在。如果密钥已存在于 `.env`，wizard 将其作为默认值提供；但如果第三方账户已创建但密钥从未保存到本地，wizard 仍会引导前往仪表板——它无法得知服务端的账户状态
4. **单阶段单屏幕**：由于每个 `stage` 都会清屏，超出终端高度的步骤会丢失滚动部分。需要将大型任务拆分为更小的阶段

### 安全性设计

- **密钥不在模型上下文中**：Agent 只生成脚本，不运行它。脚本运行时在用户本机通过 `read -rs` 捕获密钥（不回显），直接写入 `.env` 或 `gh secret`。模型与运行时完全隔离
- **一个例外**：如果在界定流程范围的聊天阶段，用户在聊天中粘贴了密钥——它就像任何其他粘贴文本一样进入了上下文。范围界定应使用密钥名称（如 `STRIPE_SECRET_KEY`）而非实际密钥值
- **`set -euo pipefail`**：所有生成脚本以此开头，任何命令失败、使用未定义变量、管道中任意环节失败都会导致脚本退出

### 与其他 Skills 的关系

- **最近邻**：`setup-matt-pocock-skills`——两者目标一致（让仓库进入可用状态），但分工明确：那个专门配置此 Skill 集，wizard 为所有其他事物生成初始化路径
- **搭档**：`implement`——当一个构建产出需要凭据或手动切换才能上线时，wizard 承担必须由人类完成的那一半
- **路由**：不确定当前场景适合哪个 Skill → 使用 `ask-matt` 获取路由建议
- **在 grilling 之后**：`/grill-with-docs → /to-spec → /wizard` 序列是常见模式（先对齐、再定 spec、再配环境），但 wizard 是独立工具而非链条环节——它可以在任何时间点触发

### 触发条件

此 Skill 为**模型调用（model-invoked）**，意味着：
- 用户可以输入 `/wizard` 手动调用
- Agent 在遇到必须由人类完成的步骤时（如无法生成的密钥、无法点击的仪表板）会**主动**触达它
- 从用户调用转为模型调用的改动（v1.2）解决了一个故障模式：Agent 在构建中遇到凭据障碍后，在聊天中倾倒六条编号步骤让用户手动跟随——现在它生成 wizard 脚本来代替

---

## 第二部分：原文与翻译

<!--原文-->
> # Wizard
> A **wizard** is a bash script that walks a human, step by step, through a manual procedure that's tedious to do by hand and tedious to re-explain to an AI every time. It opens each URL, says exactly what to click and copy, captures the values, writes them where they belong (`.env`, GitHub secrets), confirms at every stage, and shows how many stages are left. It might configure third-party services, run a one-off migration, or move the project from one state to another.

<!--翻译-->
> # 向导
> **向导**是一个 Bash 脚本，逐步引导人类完成手动流程——手动操作繁琐，每次向 AI 重新解释同样繁琐。它打开每个 URL，准确指出点击和复制的位置，捕获返回值，将值写入其归属位置（`.env`、GitHub secrets），在每个阶段确认，并显示剩余阶段数。它可能配置第三方服务、运行一次性迁移，或将项目从一个状态转移到另一个状态。

<!--原文-->
> The delightful UX is already solved by [template.sh](template.sh): stage-by-stage progress, confirmation gates, cross-platform URL opening (including WSL), hidden secret entry, idempotent `.env` upserts, `gh secret`/`gh variable` writes, and a closing summary. **Your job is only to scope the procedure and author its stages.** The library above the `STAGES` marker is identical in every wizard; that consistency is the point: never hand-edit it.

<!--翻译-->
> 出色的 UX 已由 [template.sh](template.sh) 解决：逐阶段进度、确认门禁、跨平台 URL 打开（含 WSL）、隐藏密钥输入、幂等 `.env` upsert、`gh secret`/`gh variable` 写入，以及结尾摘要。**你的工作仅是界定流程并编写阶段内容。** `STAGES` 标记之上的库在每个向导中完全一致；一致性就是设计目标：绝不手动编辑它。

<!--原文-->
> A wizard is ephemeral by default: built for one run, saved to a scratch or `scripts/` path, deleted when the job's done. Commit it only when the user wants a repeatable setup path that should live in the repo.

<!--翻译-->
> 向导默认为一次性：为单次运行构建，保存到临时或 `scripts/` 路径，任务完成后删除。仅当用户希望将可重复的初始化路径留在仓库中时才提交。

<!--原文-->
> ### 1. Scope the procedure
> Work out every manual step the human must take and every value that gets captured along the way. Read the repo first, don't ask cold:
> - For setup: `.env`, `.env.example`, `.env.*`, `README`, `docker-compose*`, framework config, and `.github/workflows/*` (every `secrets.*` / `vars.*` reference is a value the wizard must produce).
> - For a migration or transition: the current state, the target state, and the irreversible actions between them.
> Then show the user the ordered list of stages and the values each produces, and confirm: they may add, drop, or reorder.

<!--翻译-->
> ### 1. 界定流程范围
> 梳理人类必须执行的每个手动步骤，以及沿途捕获的每个值。先阅读仓库，不要冷冰冰地提问：
> - 对初始化场景：`.env`、`.env.example`、`.env.*`、`README`、`docker-compose*`、框架配置，以及 `.github/workflows/*`（每个 `secrets.*` / `vars.*` 引用都是向导必须产出的值）。
> - 对迁移或过渡场景：当前状态、目标状态，以及两者之间的不可逆操作。
> 然后向用户展示有序的阶段列表和每个阶段产出的值，并确认：用户可以添加、删除或调整顺序。

<!--原文-->
> **Done when:** every stage is named in order, and for each captured value you know (a) where the human gets it, (b) where it's written (`.env`, a GitHub secret, both, or nowhere; some stages are pure actions), and (c) whether it's secret (hidden entry) or public.

<!--翻译-->
> **完成标准**：每个阶段已按顺序命名，且对于每个捕获的值你知道 (a) 人类从哪里获取它，(b) 它被写入哪里（`.env`、GitHub secret、两者皆有，或无处——某些阶段是纯动作），(c) 它是密钥（隐藏输入）还是公开值。

<!--原文-->
> ### 2. Map each stage's journey
> For each stage, write the precise path a human follows: which URL to open, what to do there, where a value is shown, which variable it fills: e.g. "Dashboard → Developers → API keys → Reveal test key → copy". Where you don't actually know the current UI or the exact command, say so and ask the user or check the docs: never invent steps that may not exist.

<!--翻译-->
> ### 2. 绘制每个阶段的旅程
> 为每个阶段写出人类遵循的精确路径：打开哪个 URL、在那里做什么、值在哪里显示、填入哪个变量。例如："Dashboard → Developers → API keys → Reveal test key → copy"。如果不确定当前 UI 或确切命令，说明并询问用户或查阅文档：绝不编造可能不存在的步骤。

<!--原文-->
> **Done when:** every stage traces to concrete instructions a stranger could follow.

<!--翻译-->
> **完成标准**：每个阶段都追踪到陌生人也可遵循的具体指令。

<!--原文-->
> ### 3. Author the wizard
> Copy `template.sh` to the target path. Replace the example stage with one `stage` per step, in dependency order. Use the library helpers: `stage`, `say`/`step`, `open_url`, `ask`/`ask_secret`, `write_env`, `set_secret`/`set_var`, `pause`/`confirm`. Set `TOTAL_STAGES` to the number of stages you wrote.

<!--翻译-->
> ### 3. 编写向导
> 将 `template.sh` 复制到目标路径。按依赖顺序，用一个 `stage` 替换示例阶段。使用库辅助函数：`stage`、`say`/`step`、`open_url`、`ask`/`ask_secret`、`write_env`、`set_secret`/`set_var`、`pause`/`confirm`。将 `TOTAL_STAGES` 设置为你编写的阶段数。

<!--原文-->
> Hold the bar the template sets: open the URL before asking for its value, use `ask_secret` for anything secret, `write_env` every persisted value, `set_secret` only the values CI actually needs, and `confirm` before any irreversible action. Each `stage` clears the screen so only the current step is visible: keep a stage to one focused task so nothing the human needs scrolls away. Don't touch the library above the marker.

<!--翻译-->
> 守住模板设定的标准：先打开 URL 再请求其值，对密钥使用 `ask_secret`，`write_env` 每个持久化值，`set_secret` 仅对 CI 真正需要的值，任意不可逆操作前使用 `confirm`。每个 `stage` 清屏只留当前步骤：一个阶段只聚焦一个任务，人类所需的内容不会滚出屏幕。不要改动标记之上的库。

<!--原文-->
> ### 4. Verify and hand off
> - `bash -n <script>`; run `shellcheck` if available.
> - `chmod +x <script>`.
> - Don't run it end-to-end yourself: it opens browsers and blocks on human input. Trace it statically instead: every value from step 1 is captured and lands where step 1 said, and every `set_secret` name exactly matches a `secrets.*` reference in CI.
> - Tell the user how to run it. If it's a repeatable setup path, commit it and link it from the README so the next person runs the script instead of asking an AI.

<!--翻译-->
> ### 4. 验证与交付
> - `bash -n <script>`；若有 `shellcheck` 则运行。
> - `chmod +x <script>`。
> - 不要自己端到端运行：它会打开浏览器并阻塞等待人工输入。改为静态追踪：步骤 1 中每个值都被捕获并落入步骤 1 指定的目标，每个 `set_secret` 名称与 CI 中的 `secrets.*` 引用完全匹配。
> - 告知用户如何运行。如果是可重复的初始化路径，提交并在 README 中链接，让下一个人运行脚本而非重新询问 AI。

<!--原文-->
> `wizard` generates an interactive bash script that walks a human, step by step, through a manual procedure: wiring up third-party services, running a one-off migration, moving a project from state A to state B. It opens each URL, says what to click and copy, captures what comes back, and writes it into `.env` files and GitHub Actions secrets.

<!--翻译-->
> `wizard` 生成一个交互式 Bash 脚本，逐步引导人类完成手动流程：接入第三方服务、运行一次性迁移、将项目从状态 A 转移到状态 B。它打开每个 URL、指出点击和复制的位置、捕获返回值，并将其写入 `.env` 文件和 GitHub Actions secrets。

<!--原文-->
> The [agent](https://www.aihero.dev/ai-coding-dictionary/agent) writes the script; it never runs it. You do, on your own machine. A wizard is a program, not a list of instructions. The script runs each step in order and keeps the values you enter. Your part is to click, paste, and press Enter.

<!--翻译-->
> [Agent](https://www.aihero.dev/ai-coding-dictionary/agent) 编写脚本；它从不运行。你在自己的机器上运行。向导是一个程序，而不是一份指令列表。脚本按顺序运行每一步，并保留你输入的值。你的角色是点击、粘贴并按回车。

<!--原文-->
> A **stage** is one focused task on one screen. The script clears the terminal between stages, so a stage that overflows the screen loses the part that scrolled away. You author stages in dependency order and set `TOTAL_STAGES`, which drives the progress display.

<!--翻译-->
> 一个**阶段**是一个屏幕上的一个聚焦任务。脚本在阶段之间清屏，所以溢出屏幕的阶段会丢失滚出的部分。你按依赖顺序编写阶段，并设置驱动进度显示的 `TOTAL_STAGES`。

<!--原文-->
> The skill scopes the procedure before it writes any script. It reads the repo instead of asking cold: `.env*`, `docker-compose*`, framework config, and every `secrets.*` / `vars.*` reference in `.github/workflows/`. Each of those references is a value the wizard has to produce. It then shows you the ordered stage list to confirm. Only after you confirm does it map each stage to the exact path a human follows ("Dashboard → Developers → API keys → Reveal test key → copy"). Where it doesn't know the current UI, it asks you or checks the docs rather than inventing clicks.

<!--翻译-->
> 该 Skill 在编写任何脚本之前先界定流程。它先阅读仓库而非冷冰冰地提问：`.env*`、`docker-compose*`、框架配置以及 `.github/workflows/` 中的每个 `secrets.*` / `vars.*` 引用。每个引用都是向导必须产出的值。然后向你展示有序的阶段列表以供确认。只有在你确认之后，它才将每个阶段映射到人类遵循的精确路径（"Dashboard → Developers → API keys → Reveal test key → copy"）。如果不了解当前 UI，它会询问你或查阅文档，而非凭空编造点击路径。

<!--原文-->
> For each captured value, scoping decides where it goes:
> | Destination | When |
> | --- | --- |
> | `.env` only | Local dev needs it, CI doesn't |
> | GitHub secret | CI reads it, and it's sensitive |
> | GitHub variable | CI reads it, and it's public |
> | Both `.env` and a secret | Local dev and CI both need it |
> | Nowhere | The stage is a pure action: a switch flipped, a plan upgraded |

<!--翻译-->
> 对于每个捕获的值，范围界定决定其落点：
> | 落点 | 适用场景 |
> | --- | --- |
> | 仅 `.env` | 本地开发需要，CI 不需要 |
> | GitHub secret | CI 需要读取，且是敏感信息 |
> | GitHub variable | CI 需要读取，且是公开信息 |
> | `.env` 和 secret 两者 | 本地开发与 CI 都需要 |
> | 无处 | 该阶段是一个纯动作：切换开关、升级方案 |

<!--原文-->
> The template ships the whole experience: progress with time remaining, confirmation gates, cross-platform URL opening including WSL, hidden entry for secrets, idempotent `.env` upserts, `gh secret` / `gh variable` writes, and a closing summary of everything it had to skip. Everything above the `STAGES` marker is a fixed library, identical in every wizard and never hand-edited. Because the library never changes, every wizard behaves the same way. The only work per wizard is to scope the procedure and author its stages.

<!--翻译-->
> 模板提供了完整的体验：进度显示与剩余时间、确认门禁、跨平台 URL 打开（含 WSL）、密钥隐藏输入、幂等 `.env` upsert、`gh secret` / `gh variable` 写入，以及跳过的所有事项的结尾摘要。`STAGES` 标记之上的所有内容是一个固定库，在每个向导中完全相同且永不手动编辑。由于这个库从不变化，每个向导的行为方式都相同。每个向导唯一要做的工作就是界定流程并编写它的阶段内容。

<!--原文-->
> The agent that writes a wizard never runs it end to end, because the script opens browsers and waits for human input. The agent checks it statically instead: `bash -n`, `shellcheck` where available, and a trace that every value lands where scoping said it would, with every `set_secret` name matching a real `secrets.*` reference in CI. So the first run is yours, and that run is the test.

<!--翻译-->
> 编写向导的 Agent 从不端到端运行它，因为脚本会打开浏览器并等待人工输入。Agent 改为静态检查：`bash -n`、`shellcheck`（可用时），以及追踪每个值落入范围界定所指定的目标，每个 `set_secret` 名称匹配 CI 中的真实 `secrets.*` 引用。所以首次运行由你来完成，而那次运行就是测试。

<!--原文-->
> **Do my API keys end up in the model's context?**
> No. The agent writes a script; it doesn't run it. You run the script yourself, and it captures the key with hidden terminal entry and writes it straight to `.env` or `gh secret`. The wizard is a CLI, and the model is not connected to it. This holds only for values the wizard captures at runtime. If you paste a key into the chat while scoping the procedure, it's in the context like any other pasted text.

<!--翻译-->
> **我的 API 密钥会进入模型的上下文吗？**
> 不会。Agent 编写脚本，它不运行。你自己运行脚本，脚本通过隐藏的终端输入捕获密钥并直接写入 `.env` 或 `gh secret`。向导是一个 CLI，模型与之没有连接。这只适用于向导在运行时捕获的值。如果你在界定流程时在聊天中粘贴了密钥，它会像任何其他粘贴文本一样进入上下文。

<!--原文-->
> **Can I go back and fix a value I mistyped?**
> Not mid-run. There is no back button. The stages run forward, and a wrong answer on stage 3 means Ctrl-C and re-run. Re-running is cheap, because any value already written to `.env` is offered back as a default, so you press Enter through the stages you got right and retype only the wrong one. Users asked for this in launch week, and it is still open: "loved it! One thing though, is there a way to go back and correct what you've entered?"

<!--翻译-->
> **我可以回退并修正输错的值吗？**
> 运行中不行。没有回退按钮。阶段只向前运行，stage 3 的错误回答意味着 Ctrl-C 并重新运行。重跑成本低，因为任何已写入 `.env` 的值会作为默认值提供，因此你只需通过对的阶段按回车，仅重新输入错误的那一个。用户在发布周提出了这一点，而它至今仍然开放："太棒了！不过还有一点，有没有办法回去修正已输入的内容？"

<!--原文-->
> **Does it know what I've already set up?**
> Partly, and less than the launch reactions assumed. It reads the repo before it asks (your `.env` files, `docker-compose`, framework config, the `secrets.*` references in CI), so it scopes to values that are missing rather than starting from zero the way a README does. What it doesn't do is check the third-party service. If a key exists in your `.env` the wizard offers it back and Enter keeps it; if you already created the Stripe account but never saved the key, the wizard still sends you to the dashboard for it.

<!--翻译-->
> **它知道我已完成哪些配置吗？**
> 部分知道，且不如发布时的反应所假设的那么多。它在提问前阅读仓库（你的 `.env` 文件、`docker-compose`、框架配置、CI 中的 `secrets.*` 引用），因此它的范围界定针对缺失的值，而非像 README 那样从零开始。它不做的是检查第三方服务。如果密钥存在于你的 `.env` 中，向导会将其作为默认值提供，回车保留它；但如果你已创建 Stripe 账户却从未保存密钥，向导仍会将你引导到仪表板获取。

<!--原文-->
> ## It's working if
> - You're shown an ordered list of stages, and the values each one produces, and asked to confirm, before any script exists.
> - Every URL is opened before the value from that page is asked for. You're never asked to paste something you haven't been sent to fetch.
> - Secrets are typed blind. Nothing sensitive echoes into your scrollback.
> - Each stage fits one screen. Nothing you still need has scrolled away.
> - Ctrl-C and re-run picks up where you left off, offering the values already saved as defaults.
> - The final screen lists what it wrote, and separately lists what it couldn't do and you have to finish by hand.

<!--翻译-->
> ## 运行正常的标志
> - 在任何脚本存在之前，你已看到有序阶段列表和每个产出的值，并被要求确认。
> - 每个 URL 在值被索要之前已被打开。你永远不会被要求粘贴尚未被引导去获取的内容。
> - 密钥以盲输入方式键入。敏感内容从不回显到终端滚动历史。
> - 每个阶段适配一个屏幕。你仍需的任何内容都没有滚出视野。
> - Ctrl-C 后重新运行从断点继续，已保存的值作为默认值提供。
> - 最终画面列出已写入的内容，并单独列出无法完成、需你手动补完的事项。

<!--原文-->
> ## Where it fits
> `wizard` is a reach-for-it-anytime standalone, sitting at the line where automation stops and a human has to click. Its nearest neighbour is [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills), because both exist to get a repo into a working state. That one configures this skill set, and `wizard` generates a setup path for everything else. It also pairs with [implement](https://aihero.dev/skills-implement): when a build lands a feature that needs credentials or a manual cutover, a wizard covers the steps only a human can do. When you're unsure which skill fits the moment, [ask-matt](https://aihero.dev/skills-ask-matt) routes you.

<!--翻译-->
> ## 定位
> `wizard` 是一个随时可用的独立工具，位于自动化停止、人类必须点击的边界线上。其最近邻是 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills)，因为两者都旨在让仓库进入可用状态。那个配置此 Skill 集，而 `wizard` 为所有其他事物生成初始化路径。它也与 [implement](https://aihero.dev/skills-implement) 搭档：当一个构建产出需要凭据或手动切换才能上线时，wizard 覆盖只有人类才能完成的步骤。当你不确定哪个 Skill 适合当前时刻，[ask-matt](https://aihero.dev/skills-ask-matt) 为你路由。
