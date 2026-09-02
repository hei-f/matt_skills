# setup-matt-pocock-skills -- 详细分析报告

---

## 第一部分：中文详细总结

### 一、技能的定位与目的

`setup-matt-pocock-skills` 是一个**前置设置技能**，在每个代码仓库中只需运行一次。它的作用是为该仓库配置工程技能（engineering skills）所依赖的运行时环境，具体包括：

1. **Issue 追踪器（Issue Tracker）**：明确本仓库的议题/需求存放在哪里，是 GitHub Issues、GitLab Issues、本地 markdown 文件（`.scratch/`），还是其他外部系统（Jira、Linear 等）。
2. **分类标签（Triage Labels）**：定义五个标准分类角色（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`）在本仓库追踪器中的实际标签字符串。
3. **领域文档布局（Domain Docs Layout）**：确定 `CONTEXT.md` 和 ADR（架构决策记录）的存放位置和消费规则，支持单上下文（single-context）和多上下文（multi-context）两种布局。

这个技能的输出是纯 markdown 文件，全部提交到仓库中。没有全局或用户级配置模式：每个仓库各自维护自己的 `docs/agents/` 目录。它是**提示驱动的技能（prompt-driven skill）**，而非确定性脚本：它会探索仓库状态、呈现发现的内容、与用户确认后再写入。

---

### 二、技能所配置的内容

#### 2.1 Issue 追踪器（`docs/agents/issue-tracker.md`）

追踪器选项及默认姿态：

| 选项 | 议题存放位置 | 前置条件 |
|------|-------------|----------|
| **GitHub**（默认） | 仓库的 GitHub Issues | 需要 `gh` CLI |
| **GitLab** | 仓库的 GitLab Issues | 需要 `glab` CLI |
| **本地 markdown** | `.scratch/<feature>/` 目录下的文件 | 无需远程仓库 |
| **其他**（Jira、Linear 等） | 用户指定 | 用户用一段文字描述工作流 |

**默认姿态**：该技能设计时优先考虑 GitHub。如果 `git remote` 指向 GitHub，就推荐 GitHub；如果指向 GitLab（`gitlab.com` 或自托管），就推荐 GitLab；其他情况或用户偏好不同时，提供以上四种选项。

**"PR 作为请求入口"标志**：GitHub 和 GitLab 模板中包含一个 `PRs as a request surface` 标志，**默认为关闭（no）**。该标志控制外部 PR/MR 是否进入分类队列。该技能不会主动提出此选项，留待用户后续手动修改文件时启用。

#### 2.2 分类标签（`docs/agents/triage-labels.md`）

五个标准分类角色及其含义：

| 标准角色名称 | 含义 |
|-------------|------|
| `needs-triage` | 维护者需要评估此议题 |
| `needs-info` | 等待报告者提供更多信息 |
| `ready-for-agent` | 已完全明确规格，可由 AFK agent 处理 |
| `ready-for-human` | 需要人类开发者实现 |
| `wontfix` | 不会采取行动 |

默认情况下，标签字符串与角色名称相同。用户可以选择保留默认值（推荐），也可以覆盖为追踪器中已有的标签名（例如将 `needs-triage` 映射为 `bug:triage`）。

**重要**：该技能**不会创建标签**。`triage-labels.md` 只是一个映射文件，告诉下游技能（如 `/triage`）本仓库追踪器中对应五个角色的实际标签字符串是什么。如果追踪器尚未创建这些标签，需要用户手动创建。

#### 2.3 领域文档布局（`docs/agents/domain.md`）

**单上下文布局（默认）**：适用于绝大多数仓库。

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

**多上下文布局**：仅在检测到 monorepo 信号时才会提供。

```
/
├── CONTEXT-MAP.md
├── docs/adr/                          ← 系统级决策
└── src/
    ├── ordering/
    │   ├── CONTEXT.md
    │   └── docs/adr/                  ← 上下文特定决策
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

领域文档的消费规则：
- 探索前先读取 `CONTEXT.md`（或 `CONTEXT-MAP.md`）和相关的 ADR
- 使用术语表中的词汇，不要使用同义词
- 如果输出与现有 ADR 冲突，明确标注而非静默覆盖
- 如果上述文件不存在，**静默跳过**，不标记缺失，也不建议立即创建（由 `/domain-modeling` 在需要时懒加载创建）

---

### 三、执行流程

#### 步骤 1：探索仓库状态

检查以下内容以了解仓库的起始状态：

- `git remote -v` 和 `.git/config`：判断是否是 GitHub 仓库、哪个仓库
- `AGENTS.md` 和 `CLAUDE.md`：是否已存在，是否已有 `## Agent skills` 区块
- `CONTEXT.md` 和 `CONTEXT-MAP.md`：是否已存在
- `docs/adr/` 及 `src/*/docs/adr/`：ADR 目录
- `docs/agents/`：本技能之前的输出是否已存在
- `.scratch/`：本地 markdown 议题追踪器惯例是否已在使用的标志
- `triage` 技能是否已安装：决定是否执行 Section B
- Monorepo 信号：`pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段、或包含独立 `src/` 的 `packages/*` 目录

#### 步骤 2：呈现发现并询问

汇总已存在和缺失的内容，按以下三个部分依次处理。每个部分先给出推荐答案，用户用一个词即可确认。

**Section A：Issue 追踪器**（总是询问）

> 说明："Issue 追踪器"是本仓库议题的存放位置。`to-tickets`、`triage`、`to-spec` 等技能会从中读取和写入。

默认推荐匹配 `git remote` 的平台。如果 remote 指向 GitHub，推荐 GitHub；指向 GitLab，推荐 GitLab。否则，提供四种选项。

**Section B：分类标签**（仅当 `triage` 技能已安装时询问）

> 是否保留默认分类标签？（推荐：**是**）

默认五个标签名称与角色名称完全相同。只有在用户明确表示追踪器已使用其他命名惯例时，才收集覆盖映射。

**注意**：如果探索阶段发现 `triage` 技能未安装，则**完全跳过 Section B**，不写入 `triage-labels.md`，也不在 `## Agent skills` 区块中包含 `### Triage labels` 子区块。

**Section C：领域文档**（默认单上下文，仅在检测到 monorepo 信号时询问）

默认使用**单上下文**布局。仅在探索发现 monorepo 信号（`pnpm-workspace.yaml`、`workspaces` 字段、或已填充的 `packages/*`）时，才提供**多上下文**选项并让用户确认。

#### 步骤 3：确认并编辑草稿

向用户展示以下草稿内容：
- 将要添加到 `CLAUDE.md` / `AGENTS.md` 的 `## Agent skills` 区块
- `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md`（仅当 `triage` 已安装）的内容

让用户在写入前进行编辑。

#### 步骤 4：写入文件

**文件选择规则**（用于 `## Agent skills` 区块）：

1. 如果 `CLAUDE.md` 已存在，编辑它
2. 否则如果 `AGENTS.md` 已存在，编辑它
3. 如果两者都不存在，**询问用户**要创建哪一个，不代为决定
4. **绝不**在 `CLAUDE.md` 已存在时创建 `AGENTS.md`（反之亦然）；始终编辑已存在的那个

如果选择的文件中已有 `## Agent skills` 区块，则原地更新其内容，而不是追加重复区块。不要覆盖用户对周围章节的编辑。

`## Agent skills` 区块的结构：

```markdown
## Agent skills

### Issue tracker

[议题追踪位置的简要说明]。详见 `docs/agents/issue-tracker.md`。

### Triage labels

[标签词汇的简要说明]。详见 `docs/agents/triage-labels.md`。

### Domain docs

[布局简要说明："single-context" 或 "multi-context"]。详见 `docs/agents/domain.md`。
```

`### Triage labels` 子区块仅在 `triage` 已安装且 Section B 已执行时才包含。

然后写入文档文件，使用技能文件夹中的种子模板：
- `issue-tracker-github.md`：GitHub 议题追踪器
- `issue-tracker-gitlab.md`：GitLab 议题追踪器
- `issue-tracker-local.md`：本地 markdown 议题追踪器
- `triage-labels.md`：标签映射（仅当 `triage` 已安装）
- `domain.md`：领域文档消费规则和布局

对于"其他"追踪器，根据用户描述从头编写 `docs/agents/issue-tracker.md`。

#### 步骤 5：完成

告知用户设置已完成，哪些工程技能现在会读取这些文件。提醒用户可以随时直接编辑 `docs/agents/*.md`；只有在需要切换追踪器或从头开始时才需要重新运行此技能。

---

### 四、三个输出文件

| 文件 | 路径 | 说明 |
|------|------|------|
| Issue 追踪器配置 | `docs/agents/issue-tracker.md` | 定义议题存放位置、CRUD 操作、PR 作为请求入口标志、`/wayfinder` 的路标操作 |
| 分类标签映射 | `docs/agents/triage-labels.md` | 将五个标准角色映射到追踪器中的实际标签字符串；仅在 `triage` 技能安装时创建 |
| 领域文档规则 | `docs/agents/domain.md` | 定义如何消费 `CONTEXT.md` 和 ADR，以及单/多上下文布局 |

外加在 `CLAUDE.md` 或 `AGENTS.md` 中追加的 `## Agent skills` 区块，作为指向这三个文件的索引。

---

### 五、关键规则和边界条件

1. **文件选择**：优先 `CLAUDE.md`，其次 `AGENTS.md`，都不存在则询问用户。绝不两者都创建。
2. **Section B 跳过条件**：当 `triage` 技能未安装时，完全跳过 Section B，不写 `triage-labels.md`，不在 `## Agent skills` 块中包含 `### Triage labels`。
3. **多上下文领域文档的前提**：仅在检测到 monorepo 信号（`pnpm-workspace.yaml`、`package.json` 的 `workspaces` 字段、或已填充的 `packages/*` 目录）时才提供多上下文选项。这些信号仅在真正的大型多包仓库中存在；其缺失意味着单上下文，适用于几乎所有的仓库。
4. **"PR 作为请求入口"标志默认为关闭**：不主动向用户提出此选项，用户可后续手动修改。
5. **不创建实际标签**：`triage-labels.md` 是映射文件，不执行 `gh label create` 等命令。
6. **不创建 `CONTEXT.md`**：领域文档规则记录了文档的消费方式，但实际创建由 `/domain-modeling` 在需要时执行。

---

## 第二部分：原始英文文本及中文翻译

### SKILL.md

---

<!--原文-->
> # Setup Matt Pocock's Skills
<!--翻译-->
> # 设置 Matt Pocock 技能

---

<!--原文-->
> Scaffold the per-repo configuration that the engineering skills assume:
>
> - **Issue tracker**: where issues live (GitHub by default; local markdown is also supported out of the box)
> - **Triage labels**: the strings used for the five canonical triage roles
> - **Domain docs**: where `CONTEXT.md` and ADRs live, and the consumer rules for reading them
<!--翻译-->
> 搭建工程技能所依赖的每个仓库的配置：
>
> - **Issue 追踪器**：议题的存放位置（默认 GitHub；也支持本地 markdown）
> - **分类标签**：五个标准分类角色所使用的标签字符串
> - **领域文档**：`CONTEXT.md` 和 ADR 的存放位置，以及读取它们的消费规则

---

<!--原文-->
> This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.
<!--翻译-->
> 这是一个提示驱动的技能，而非确定性脚本。先探索、呈现发现的内容、与用户确认，然后写入。

---

<!--原文-->
> ## Process
<!--翻译-->
> ## 流程

---

<!--原文-->
> ### 1. Explore
>
> Look at the current repo to understand its starting state. Read whatever exists; don't assume:
>
> - `git remote -v` and `.git/config`: is this a GitHub repo? Which one?
> - `AGENTS.md` and `CLAUDE.md` at the repo root: does either exist? Is there already an `## Agent skills` section in either?
> - `CONTEXT.md` and `CONTEXT-MAP.md` at the repo root
> - `docs/adr/` and any `src/*/docs/adr/` directories
> - `docs/agents/`: does this skill's prior output already exist?
> - `.scratch/`: a sign that a local-markdown issue tracker convention is already in use
> - Is the `triage` skill installed? (a `triage` skill folder alongside this one, or `triage` in your available skills.) This decides whether Section B runs at all.
> - Monorepo signals: a `pnpm-workspace.yaml`, a `workspaces` field in `package.json`, or a populated `packages/*` with its own `src/`. These are present only in a genuinely large multi-package repo; their absence means single-context, which is almost every repo.
<!--翻译-->
> ### 1. 探索
>
> 查看当前仓库以了解其起始状态。读取已存在的内容；不要假设：
>
> - `git remote -v` 和 `.git/config`：这是 GitHub 仓库吗？是哪一个？
> - 仓库根目录的 `AGENTS.md` 和 `CLAUDE.md`：是否存在？是否已有 `## Agent skills` 部分？
> - 仓库根目录的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
> - `docs/adr/` 及任何 `src/*/docs/adr/` 目录
> - `docs/agents/`：本技能之前的输出是否已存在？
> - `.scratch/`：本地 markdown 议题追踪器惯例是否已在使用的标志
> - `triage` 技能是否已安装？（与本技能并列的 `triage` 技能文件夹，或可用技能列表中的 `triage`）这决定 Section B 是否执行。
> - Monorepo 信号：`pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段、或包含独立 `src/` 的已填充 `packages/*` 目录。这些仅在真正的大型多包仓库中存在；其缺失意味着单上下文，这适用于几乎所有的仓库。

---

<!--原文-->
> ### 2. Present findings and ask
>
> Summarise what's present and what's missing. Then take the sections in order. One section, one answer, then the next.
>
> Lead each section with the recommended answer so the user can accept it in a word. Give a one-line explainer only when the choice genuinely branches; skip the section entirely when exploration already settled it (Section B when `triage` isn't installed, Section C when there's no monorepo).
<!--翻译-->
> ### 2. 呈现发现并询问
>
> 汇总已存在和缺失的内容。然后按顺序处理各部分。一个部分，一个回答，再下一个。
>
> 每个部分以推荐答案开头，让用户用一个词即可确认。仅当选择确实有分支时才给出一行说明；当探索已经明确了答案时（`triage` 未安装时跳过 Section B，无 monorepo 时跳过 Section C），完全跳过该部分。

---

<!--原文-->
> **Section A: Issue tracker.**
>
> > Explainer: The "issue tracker" is where issues live for this repo. Skills like `to-tickets`, `triage`, and `to-spec` read from and write to it. They need to know whether to call `gh issue create`, write a markdown file under `.scratch/`, or follow some other workflow you describe. Pick the place you actually track work for this repo.
>
> Default posture: these skills were designed for GitHub. If a `git remote` points at GitHub, propose that. If a `git remote` points at GitLab (`gitlab.com` or a self-hosted host), propose GitLab. Otherwise (or if the user prefers), offer:
>
> - **GitHub**: issues live in the repo's GitHub Issues (uses the `gh` CLI)
> - **GitLab**: issues live in the repo's GitLab Issues (uses the `glab` CLI)
> - **Local markdown**: issues live as files under `.scratch/<feature>/` in this repo (good for solo projects or repos without a remote)
> - **Other** (Jira, Linear, etc.): ask the user to describe the workflow in one paragraph; the skill will record it as freeform prose
>
> Record the choice in `docs/agents/issue-tracker.md`. The GitHub and GitLab templates carry a "PRs as a request surface" flag, defaulted **off**. Leave it off and don't raise it: a user who wants external PRs in the triage queue can flip the flag in the file later.
<!--翻译-->
> **Section A：Issue 追踪器。**
>
> > 说明："Issue 追踪器"是本仓库议题的存放位置。`to-tickets`、`triage` 和 `to-spec` 等技能会从中读取和写入。它们需要知道是调用 `gh issue create`、在 `.scratch/` 下写 markdown 文件，还是遵循你描述的某种其他工作流。选择你实际跟踪本仓库工作的地方。
>
> 默认姿态：这些技能是为 GitHub 设计的。如果 `git remote` 指向 GitHub，推荐它。如果 `git remote` 指向 GitLab（`gitlab.com` 或自托管主机），推荐 GitLab。否则（或用户偏好），提供：
>
> - **GitHub**：议题存放在仓库的 GitHub Issues 中（使用 `gh` CLI）
> - **GitLab**：议题存放在仓库的 GitLab Issues 中（使用 `glab` CLI）
> - **本地 markdown**：议题作为 `.scratch/<feature>/` 下的文件存放在本仓库中（适合个人项目或无远程的仓库）
> - **其他**（Jira、Linear 等）：请用户用一段文字描述工作流；技能将记录为自由文本
>
> 将选择记录在 `docs/agents/issue-tracker.md` 中。GitHub 和 GitLab 模板带有一个 "PRs as a request surface" 标志，默认为**关闭**。不主动提出此标志：希望外部 PR 进入分类队列的用户可以稍后在文件中手动启用。

---

<!--原文-->
> **Section B: Triage label vocabulary.** Skip this section entirely if the `triage` skill isn't installed (exploration told you), since an uninstalled skill needs no labels.
>
> If it is installed, ask exactly one question:
>
> > Do you want to keep the default triage labels? (recommended: **yes**)
>
> The defaults are the five canonical roles, each label string equal to its name: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. On **yes**, write them as-is. Only if the user says no, usually because their tracker already uses other names (e.g. `bug:triage` for `needs-triage`), collect the overrides so `triage` applies existing labels instead of creating duplicates.
<!--翻译-->
> **Section B：分类标签词汇。** 如果 `triage` 技能未安装（探索阶段已告知），则完全跳过此部分，因为未安装的技能不需要标签。
>
> 如果已安装，只问一个问题：
>
> > 是否保留默认分类标签？（推荐：**是**）
>
> 默认值为五个标准角色，每个标签字符串等于其名称：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。如果回答**是**，按原样写入。仅当用户说否时（通常是因为其追踪器已使用其他名称，例如用 `bug:triage` 代替 `needs-triage`），收集覆盖映射，以便 `triage` 使用已有标签而不是创建重复项。

---

<!--原文-->
> **Section C: Domain docs.** Default to **single-context** (one `CONTEXT.md` + `docs/adr/` at the repo root). This fits almost every repo; write it without asking.
>
> Offer **multi-context** (a root `CONTEXT-MAP.md` pointing to per-context `CONTEXT.md` files) only when exploration found monorepo signals. Then confirm which layout they want.
<!--翻译-->
> **Section C：领域文档。** 默认使用**单上下文**（一个 `CONTEXT.md` + 仓库根目录下的 `docs/adr/`）。这适用于几乎所有的仓库；无需询问即可写入。
>
> 仅在探索发现 monorepo 信号时，才提供**多上下文**（根目录的 `CONTEXT-MAP.md` 指向每个上下文的 `CONTEXT.md` 文件）。然后确认他们想要哪种布局。

---

<!--原文-->
> ### 3. Confirm and edit
>
> Show the user a draft of:
>
> - The `## Agent skills` block to add to whichever of `CLAUDE.md` / `AGENTS.md` is being edited (see step 4 for selection rules)
> - The contents of `docs/agents/issue-tracker.md`, `docs/agents/domain.md`, and `docs/agents/triage-labels.md` (the last only when `triage` is installed)
>
> Let them edit before writing.
<!--翻译-->
> ### 3. 确认并编辑
>
> 向用户展示以下草稿：
>
> - 将要添加到 `CLAUDE.md` / `AGENTS.md`（编辑对象的选择规则见步骤 4）中的 `## Agent skills` 区块
> - `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md`（后者仅当 `triage` 已安装时）的内容
>
> 让他们在写入前进行编辑。

---

<!--原文-->
> ### 4. Write
>
> **Pick the file to edit:**
>
> - If `CLAUDE.md` exists, edit it.
> - Else if `AGENTS.md` exists, edit it.
> - If neither exists, ask the user which one to create; don't pick for them.
>
> Never create `AGENTS.md` when `CLAUDE.md` already exists (or vice versa); always edit the one that's already there.
>
> If an `## Agent skills` block already exists in the chosen file, update its contents in-place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.
<!--翻译-->
> ### 4. 写入
>
> **选择要编辑的文件：**
>
> - 如果 `CLAUDE.md` 存在，编辑它。
> - 否则如果 `AGENTS.md` 存在，编辑它。
> - 如果两者都不存在，询问用户要创建哪一个；不要代为决定。
>
> 绝不在 `CLAUDE.md` 已存在时创建 `AGENTS.md`（反之亦然）；始终编辑已存在的那个。
>
> 如果所选文件中已有 `## Agent skills` 区块，则原地更新其内容，而不是追加重复项。不要覆盖用户对周围章节的编辑。

---

<!--原文-->
> The block:
>
> ```markdown
> ## Agent skills
>
> ### Issue tracker
>
> [one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.
>
> ### Triage labels
>
> [one-line summary of the label vocabulary]. See `docs/agents/triage-labels.md`.
>
> ### Domain docs
>
> [one-line summary of layout: "single-context" or "multi-context"]. See `docs/agents/domain.md`.
> ```
>
> Include the `### Triage labels` sub-block, and write `docs/agents/triage-labels.md`, only when `triage` is installed and Section B ran. When it isn't, both are omitted.
<!--翻译-->
> 该区块：
>
> ```markdown
> ## Agent skills
>
> ### Issue tracker
>
> [议题追踪位置的简要说明]。详见 `docs/agents/issue-tracker.md`。
>
> ### Triage labels
>
> [标签词汇的简要说明]。详见 `docs/agents/triage-labels.md`。
>
> ### Domain docs
>
> [布局简要说明："single-context" 或 "multi-context"]。详见 `docs/agents/domain.md`。
> ```
>
> 仅在 `triage` 已安装且 Section B 已执行时，才包含 `### Triage labels` 子区块并写入 `docs/agents/triage-labels.md`。否则两者都省略。

---

<!--原文-->
> Then write the docs files using the seed templates in this skill folder as a starting point:
>
> - [issue-tracker-github.md](./issue-tracker-github.md): GitHub issue tracker
> - [issue-tracker-gitlab.md](./issue-tracker-gitlab.md): GitLab issue tracker
> - [issue-tracker-local.md](./issue-tracker-local.md): local-markdown issue tracker
> - [triage-labels.md](./triage-labels.md): label mapping (only if `triage` is installed)
> - [domain.md](./domain.md): domain doc consumer rules + layout
>
> For "other" issue trackers, write `docs/agents/issue-tracker.md` from scratch using the user's description.
<!--翻译-->
> 然后使用本技能文件夹中的种子模板作为起点写入文档文件：
>
> - [issue-tracker-github.md](./issue-tracker-github.md)：GitHub 议题追踪器
> - [issue-tracker-gitlab.md](./issue-tracker-gitlab.md)：GitLab 议题追踪器
> - [issue-tracker-local.md](./issue-tracker-local.md)：本地 markdown 议题追踪器
> - [triage-labels.md](./triage-labels.md)：标签映射（仅当 `triage` 已安装时）
> - [domain.md](./domain.md)：领域文档消费规则和布局
>
> 对于"其他"追踪器，根据用户的描述从头编写 `docs/agents/issue-tracker.md`。

---

<!--原文-->
> ### 5. Done
>
> Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit `docs/agents/*.md` directly later; re-running this skill is only necessary if they want to switch issue trackers or restart from scratch.
<!--翻译-->
> ### 5. 完成
>
> 告知用户设置已完成，以及哪些工程技能现在将读取这些文件。提一下他们可以稍后直接编辑 `docs/agents/*.md`；只有在想切换议题追踪器或从头开始时才需要重新运行此技能。

---

### domain.md

---

<!--原文-->
> # Domain Docs
>
> How the engineering skills should consume this repo's domain documentation when exploring the codebase.
<!--翻译-->
> # 领域文档
>
> 工程技能在探索代码库时应如何消费本仓库的领域文档。

---

<!--原文-->
> ## Before exploring, read these
>
> - **`CONTEXT.md`** at the repo root, or
> - **`CONTEXT-MAP.md`** at the repo root if it exists: it points at one `CONTEXT.md` per context. Read each one relevant to the topic.
> - **`docs/adr/`**: read ADRs that touch the area you're about to work in. In multi-context repos, also check `src/<context>/docs/adr/` for context-scoped decisions.
>
> If any of these files don't exist, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The `/domain-modeling` skill (reached via `/grill-with-docs` and `/improve-codebase-architecture`) creates them lazily when terms or decisions actually get resolved.
<!--翻译-->
> ## 探索前，先读取这些
>
> - 仓库根目录的 **`CONTEXT.md`**，或者
> - 如果存在仓库根目录的 **`CONTEXT-MAP.md`**：它指向每个上下文的一个 `CONTEXT.md`。读取与主题相关的每个文件。
> - **`docs/adr/`**：读取涉及你即将处理的领域的 ADR。在多上下文仓库中，还要检查 `src/<context>/docs/adr/` 以获取上下文范围内的决策。
>
> 如果上述任何文件不存在，**静默继续**。不要标记它们的缺失；不要建议事先创建。`/domain-modeling` 技能（通过 `/grill-with-docs` 和 `/improve-codebase-architecture` 访问）会在术语或决策实际解决时懒加载创建它们。

---

<!--原文-->
> ## File structure
>
> Single-context repo (most repos):
>
> ```
> /
> ├── CONTEXT.md
> ├── docs/adr/
> │   ├── 0001-event-sourced-orders.md
> │   └── 0002-postgres-for-write-model.md
> └── src/
> ```
>
> Multi-context repo (presence of `CONTEXT-MAP.md` at the root):
>
> ```
> /
> ├── CONTEXT-MAP.md
> ├── docs/adr/                          ← system-wide decisions
> └── src/
>     ├── ordering/
>     │   ├── CONTEXT.md
>     │   └── docs/adr/                  ← context-specific decisions
>     └── billing/
>         ├── CONTEXT.md
>         └── docs/adr/
> ```
<!--翻译-->
> ## 文件结构
>
> 单上下文仓库（大多数仓库）：
>
> ```
> /
> ├── CONTEXT.md
> ├── docs/adr/
> │   ├── 0001-event-sourced-orders.md
> │   └── 0002-postgres-for-write-model.md
> └── src/
> ```
>
> 多上下文仓库（根目录存在 `CONTEXT-MAP.md` 时）：
>
> ```
> /
> ├── CONTEXT-MAP.md
> ├── docs/adr/                          ← 系统级决策
> └── src/
>     ├── ordering/
>     │   ├── CONTEXT.md
>     │   └── docs/adr/                  ← 上下文特定决策
>     └── billing/
>         ├── CONTEXT.md
>         └── docs/adr/
> ```

---

<!--原文-->
> ## Use the glossary's vocabulary
>
> When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in `CONTEXT.md`. Don't drift to synonyms the glossary explicitly avoids.
>
> If the concept you need isn't in the glossary yet, that's a signal: either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/domain-modeling`).
<!--翻译-->
> ## 使用术语表的词汇
>
> 当你的输出中命名一个领域概念时（在议题标题、重构提案、假设、测试名称中），使用 `CONTEXT.md` 中定义的术语。不要偏离到术语表明确避免的同义词。
>
> 如果你需要的概念尚不在术语表中，这是一个信号：要么你在发明项目不使用的语言（请重新考虑），要么确实存在空白（为 `/domain-modeling` 记录下来）。

---

<!--原文-->
> ## Flag ADR conflicts
>
> If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:
>
> > _Contradicts ADR-0007 (event-sourced orders), but worth reopening because..._
<!--翻译-->
> ## 标记 ADR 冲突
>
> 如果你的输出与现有 ADR 矛盾，明确标注而不是静默覆盖：
>
> > _与 ADR-0007（事件溯源订单）矛盾，但值得重新讨论，因为……_

---

### triage-labels.md

---

<!--原文-->
> # Triage Labels
>
> The skills speak in terms of five canonical triage roles. This file maps those roles to the actual label strings used in this repo's issue tracker.
<!--翻译-->
> # 分类标签
>
> 技能使用五个标准分类角色来表达。本文件将这些角色映射到本仓库议题追踪器中使用的实际标签字符串。

---

<!--原文-->
> | Label in mattpocock/skills | Label in our tracker | Meaning                                  |
> | -------------------------- | -------------------- | ---------------------------------------- |
> | `needs-triage`             | `needs-triage`       | Maintainer needs to evaluate this issue  |
> | `needs-info`               | `needs-info`         | Waiting on reporter for more information |
> | `ready-for-agent`          | `ready-for-agent`    | Fully specified, ready for an AFK agent  |
> | `ready-for-human`          | `ready-for-human`    | Requires human implementation            |
> | `wontfix`                  | `wontfix`            | Will not be actioned                     |
<!--翻译-->
> | mattpocock/skills 中的标签 | 我们追踪器中的标签 | 含义                               |
> | -------------------------- | ------------------ | ---------------------------------- |
> | `needs-triage`             | `needs-triage`     | 维护者需要评估此议题               |
> | `needs-info`               | `needs-info`       | 等待报告者提供更多信息             |
> | `ready-for-agent`          | `ready-for-agent`  | 已完全明确规格，可由 AFK agent 处理 |
> | `ready-for-human`          | `ready-for-human`  | 需要人类开发者实现                 |
> | `wontfix`                  | `wontfix`          | 不会采取行动                       |

---

<!--原文-->
> When a skill mentions a role (e.g. "apply the AFK-ready triage label"), use the corresponding label string from this table.
>
> Edit the right-hand column to match whatever vocabulary you actually use.
<!--翻译-->
> 当某个技能提及某个角色时（例如"应用 AFK-ready 分类标签"），请使用此表中对应的标签字符串。
>
> 编辑右列以匹配你实际使用的任何词汇。

---

### issue-tracker-github.md

---

<!--原文-->
> # Issue tracker: GitHub
>
> Issues and specs for this repo live as GitHub issues. Use the `gh` CLI for all operations.
<!--翻译-->
> # Issue 追踪器：GitHub
>
> 本仓库的议题和规格以 GitHub issues 的形式存在。所有操作使用 `gh` CLI。

---

<!--原文-->
> ## Conventions
>
> - **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
> - **Read an issue**: `gh issue view <number> --comments`, filtering comments by `jq` and also fetching labels.
> - **List issues**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with appropriate `--label` and `--state` filters.
> - **Comment on an issue**: `gh issue comment <number> --body "..."`
> - **Apply / remove labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
> - **Close**: `gh issue close <number> --comment "..."`
>
> Infer the repo from `git remote -v`; `gh` does this automatically when run inside a clone.
<!--翻译-->
> ## 约定
>
> - **创建议题**：`gh issue create --title "..." --body "..."`。多行正文使用 heredoc。
> - **读取议题**：`gh issue view <number> --comments`，通过 `jq` 过滤评论并同时获取标签。
> - **列出议题**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，配合适当的 `--label` 和 `--state` 过滤。
> - **评论议题**：`gh issue comment <number> --body "..."`
> - **添加/移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
> - **关闭议题**：`gh issue close <number> --comment "..."`
>
> 从 `git remote -v` 推断仓库；`gh` 在克隆内运行时自动处理。

---

<!--原文-->
> ## Pull requests as a triage surface
>
> **PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_
>
> When set to `yes`, PRs run through the same labels and states as issues, using the `gh pr` equivalents:
>
> - **Read a PR**: `gh pr view <number> --comments` and `gh pr diff <number>` for the diff.
> - **List external PRs for triage**: `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments` then keep only `authorAssociation` of `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, or `NONE` (drop `OWNER`/`MEMBER`/`COLLABORATOR`).
> - **Comment / label / close**: `gh pr comment`, `gh pr edit --add-label`/`--remove-label`, `gh pr close`.
>
> GitHub shares one number space across issues and PRs, so a bare `#42` may be either: resolve with `gh pr view 42` and fall back to `gh issue view 42`.
<!--翻译-->
> ## Pull Request 作为分类入口
>
> **PR 作为请求入口：否。** _（如果本仓库将外部 PR 视为功能请求，则设置为 `yes`；`/triage` 会读取此标志。）_
>
> 当设置为 `yes` 时，PR 经过与议题相同的标签和状态，使用 `gh pr` 等价命令：
>
> - **读取 PR**：`gh pr view <number> --comments` 以及 `gh pr diff <number>` 获取差异。
> - **列出外部 PR 进行分类**：`gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`，然后仅保留 `authorAssociation` 为 `CONTRIBUTOR`、`FIRST_TIME_CONTRIBUTOR` 或 `NONE` 的（丢弃 `OWNER`/`MEMBER`/`COLLABORATOR`）。
> - **评论/标签/关闭**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。
>
> GitHub 共享议题和 PR 的编号空间，因此一个裸露的 `#42` 可能是其中之一：先用 `gh pr view 42` 解析，失败则回退到 `gh issue view 42`。

---

<!--原文-->
> ## When a skill says "publish to the issue tracker"
>
> Create a GitHub issue.
<!--翻译-->
> ## 当技能说"发布到议题追踪器"
>
> 创建一个 GitHub issue。

---

<!--原文-->
> ## When a skill says "fetch the relevant ticket"
>
> Run `gh issue view <number> --comments`.
<!--翻译-->
> ## 当技能说"获取相关票据"
>
> 运行 `gh issue view <number> --comments`。

---

<!--原文-->
> ## Wayfinding operations
>
> Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.
>
> - **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `gh issue create --label wayfinder:map`.
> - **Child ticket**: an issue linked to the map as a GitHub sub-issue (`gh api` on the sub-issues endpoint). Where sub-issues aren't enabled, add the child to a task list in the map body and put `Part of #<map>` at the top of the child body. Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
> - **Blocking**: GitHub's **native issue dependencies**, the canonical, UI-visible representation. Add an edge with `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`, where `<blocker-db-id>` is the blocker's numeric **database id** (`gh api repos/<owner>/<repo>/issues/<n> --jq .id`, _not_ the `#number` or `node_id`). GitHub reports `issue_dependencies_summary.blocked_by` (open blockers only, the live gate). Where dependencies aren't available, fall back to a `Blocked by: #<n>, #<n>` line at the top of the child body. A ticket is unblocked when every blocker is closed.
> - **Frontier query**: list the map's open children (`gh issue list --state open`, scoped to the map's sub-issues / task list), drop any with an open blocker (`issue_dependencies_summary.blocked_by > 0`, or an open issue in the `Blocked by` line) or an assignee; first in map order wins.
> - **Claim**: `gh issue edit <n> --add-assignee @me`, the session's first write.
> - **Resolve**: `gh issue comment <n> --body "<answer>"`, then `gh issue close <n>`, then append a context pointer (gist + link) to the map's Decisions-so-far.
<!--翻译-->
> ## 路标操作（Wayfinding）
>
> 由 `/wayfinder` 使用。**地图**是一个单独的议题，带有作为票据的**子**议题。
>
> - **地图**：一个标记为 `wayfinder:map` 的单独议题，包含 Notes / Decisions-so-far / Fog 正文。`gh issue create --label wayfinder:map`。
> - **子票据**：作为 GitHub 子议题链接到地图的议题（在子议题端点上使用 `gh api`）。如果子议题未启用，将子议题添加到地图正文的任务列表中，并在子议题正文顶部放置 `Part of #<map>`。标签：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。一旦被认领，该票据分配给驱动开发者。
> - **阻塞**：GitHub 的**原生议题依赖**，规范的、UI 可见的表示形式。通过 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 添加依赖边，其中 `<blocker-db-id>` 是阻塞者的数字**数据库 ID**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，而不是 `#number` 或 `node_id`）。GitHub 报告 `issue_dependencies_summary.blocked_by`（仅开放阻塞者，即时门控）。如果依赖不可用，回退到子正文顶部的 `Blocked by: #<n>, #<n>` 行。当每个阻塞者都已关闭时，票据解除阻塞。
> - **前沿查询**：列出地图的开放子议题（`gh issue list --state open`，限定在地图的子议题/任务列表），丢弃任何有开放阻塞者（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行中的开放议题）或已有分配者的；按地图顺序的第一个获胜。
> - **认领**：`gh issue edit <n> --add-assignee @me`，会话的首次写入。
> - **解决**：`gh issue comment <n> --body "<answer>"`，然后 `gh issue close <n>`，然后将上下文指针（gist + 链接）追加到地图的 Decisions-so-far。

---

### issue-tracker-gitlab.md

---

<!--原文-->
> # Issue tracker: GitLab
>
> Issues and specs for this repo live as GitLab issues. Use the [`glab`](https://gitlab.com/gitlab-org/cli) CLI for all operations.
<!--翻译-->
> # Issue 追踪器：GitLab
>
> 本仓库的议题和规格以 GitLab issues 的形式存在。所有操作使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI。

---

<!--原文-->
> ## Conventions
>
> - **Create an issue**: `glab issue create --title "..." --description "..."`. Use a heredoc for multi-line descriptions. Pass `--description -` to open an editor.
> - **Read an issue**: `glab issue view <number> --comments`. Use `-F json` for machine-readable output.
> - **List issues**: `glab issue list -F json` with appropriate `--label` filters.
> - **Comment on an issue**: `glab issue note <number> --message "..."`. GitLab calls comments "notes".
> - **Apply / remove labels**: `glab issue update <number> --label "..."` / `--unlabel "..."`. Multiple labels can be comma-separated or by repeating the flag.
> - **Close**: `glab issue close <number>`. `glab issue close` does not accept a closing comment, so post the explanation first with `glab issue note <number> --message "..."`, then close.
> - **Merge requests**: GitLab calls PRs "merge requests". Use `glab mr create`, `glab mr view`, `glab mr note`, etc., the same shape as `gh pr ...` with `mr` in place of `pr` and `note`/`--message` in place of `comment`/`--body`.
>
> Infer the repo from `git remote -v`; `glab` does this automatically when run inside a clone.
<!--翻译-->
> ## 约定
>
> - **创建议题**：`glab issue create --title "..." --description "..."`。多行描述使用 heredoc。传递 `--description -` 打开编辑器。
> - **读取议题**：`glab issue view <number> --comments`。使用 `-F json` 获取机器可读输出。
> - **列出议题**：`glab issue list -F json`，配合适当的 `--label` 过滤。
> - **评论议题**：`glab issue note <number> --message "..."`。GitLab 将评论称为 "notes"。
> - **添加/移除标签**：`glab issue update <number> --label "..."` / `--unlabel "..."`。多个标签可以用逗号分隔或重复标志。
> - **关闭议题**：`glab issue close <number>`。`glab issue close` 不接受关闭时的评论，因此先用 `glab issue note <number> --message "..."` 发布解释，然后关闭。
> - **合并请求**：GitLab 将 PR 称为 "merge requests"。使用 `glab mr create`、`glab mr view`、`glab mr note` 等，与 `gh pr ...` 形式相同，以 `mr` 代替 `pr`，以 `note`/`--message` 代替 `comment`/`--body`。
>
> 从 `git remote -v` 推断仓库；`glab` 在克隆内运行时自动处理。

---

<!--原文-->
> ## Merge requests as a triage surface
>
> **MRs as a request surface: no.** _(Set to `yes` if this repo treats external merge requests as feature requests; `/triage` reads this flag.)_
>
> When set to `yes`, MRs run through the same labels and states as issues, using the `glab mr` equivalents:
>
> - **Read an MR**: `glab mr view <number> --comments` and `glab mr diff <number>` for the diff.
> - **List external MRs for triage**: `glab mr list -F json`, then keep only MRs whose author is not a project member/owner (a contributor's MR, not a maintainer's in-flight work).
> - **Comment / label / close**: `glab mr note`, `glab mr update --label`/`--unlabel`, `glab mr close`.
>
> Unlike GitHub, GitLab numbers issues and MRs separately, so `#42` is unambiguous once you know which surface the maintainer means.
<!--翻译-->
> ## 合并请求作为分类入口
>
> **MR 作为请求入口：否。** _（如果本仓库将外部合并请求视为功能请求，则设置为 `yes`；`/triage` 会读取此标志。）_
>
> 当设置为 `yes` 时，MR 经过与议题相同的标签和状态，使用 `glab mr` 等价命令：
>
> - **读取 MR**：`glab mr view <number> --comments` 以及 `glab mr diff <number>` 获取差异。
> - **列出外部 MR 进行分类**：`glab mr list -F json`，然后仅保留作者不是项目成员/所有者的 MR（贡献者的 MR，而非维护者进行中的工作）。
> - **评论/标签/关闭**：`glab mr note`、`glab mr update --label`/`--unlabel`、`glab mr close`。
>
> 与 GitHub 不同，GitLab 对议题和 MR 分开编号，因此一旦你知道维护者指代的是哪个入口，`#42` 就是明确的。

---

<!--原文-->
> ## When a skill says "publish to the issue tracker"
>
> Create a GitLab issue.
<!--翻译-->
> ## 当技能说"发布到议题追踪器"
>
> 创建一个 GitLab issue。

---

<!--原文-->
> ## When a skill says "fetch the relevant ticket"
>
> Run `glab issue view <number> --comments`.
<!--翻译-->
> ## 当技能说"获取相关票据"
>
> 运行 `glab issue view <number> --comments`。

---

<!--原文-->
> ## Wayfinding operations
>
> Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.
>
> - **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `glab issue create --label wayfinder:map`. (On GitLab tiers with native epics, an epic may hold the map instead; a labelled issue works everywhere.)
> - **Child ticket**: an issue carrying `Part of #<map>` at the top of its description and labels `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
> - **Blocking**: GitLab's **native blocking link**, the canonical, UI-visible representation. Add it with the `/blocked_by #<n>` quick action, posted as a note (`glab issue note <child> --message "/blocked_by #<blocker>"`). Native blocking links are a Premium/Ultimate feature; on the free tier (or where unavailable) fall back to a `Blocked by: #<n>, #<n>` line at the top of the description. A ticket is unblocked when every blocker is closed.
> - **Frontier query**: `glab issue list -F json` scoped to the map's children, drop any with an open blocker: a native `blocked_by` link to an open issue (`glab api projects/:id/issues/:iid/links`), or an open issue in the `Blocked by` line, or an assignee; first in map order wins.
> - **Claim**: `glab issue update <n> --assignee @me`, the session's first write.
> - **Resolve**: `glab issue note <n> --message "<answer>"`, then `glab issue close <n>`, then append a context pointer (gist + link) to the map's Decisions-so-far.
<!--翻译-->
> ## 路标操作（Wayfinding）
>
> 由 `/wayfinder` 使用。**地图**是一个单独的议题，带有作为票据的**子**议题。
>
> - **地图**：一个标记为 `wayfinder:map` 的单独议题，包含 Notes / Decisions-so-far / Fog 正文。`glab issue create --label wayfinder:map`。（在具有原生史诗功能的 GitLab 层级上，史诗也可以承载地图；带标签的议题在任何地方都有效。）
> - **子票据**：一个议题，在其描述顶部带有 `Part of #<map>`，标签为 `wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。一旦被认领，该票据分配给驱动开发者。
> - **阻塞**：GitLab 的**原生阻塞链接**，规范的、UI 可见的表示形式。通过 `/blocked_by #<n>` 快速操作添加，以 note 形式发布（`glab issue note <child> --message "/blocked_by #<blocker>"`）。原生阻塞链接是 Premium/Ultimate 功能；在免费层级（或不可用时）回退到描述顶部的 `Blocked by: #<n>, #<n>` 行。当每个阻塞者都已关闭时，票据解除阻塞。
> - **前沿查询**：`glab issue list -F json` 限定在地图的子议题，丢弃任何有开放阻塞者的：原生 `blocked_by` 链接到开放议题（`glab api projects/:id/issues/:iid/links`），或 `Blocked by` 行中的开放议题，或已有分配者的；按地图顺序的第一个获胜。
> - **认领**：`glab issue update <n> --assignee @me`，会话的首次写入。
> - **解决**：`glab issue note <n> --message "<answer>"`，然后 `glab issue close <n>`，然后将上下文指针（gist + 链接）追加到地图的 Decisions-so-far。

---

### issue-tracker-local.md

---

<!--原文-->
> # Issue tracker: Local Markdown
>
> Issues and specs for this repo live as markdown files in `.scratch/`.
<!--翻译-->
> # Issue 追踪器：本地 Markdown
>
> 本仓库的议题和规格以 markdown 文件的形式存放在 `.scratch/` 中。

---

<!--原文-->
> ## Conventions
>
> - One feature per directory: `.scratch/<feature-slug>/`
> - The spec is `.scratch/<feature-slug>/spec.md`
> - Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a single combined tickets file
> - Triage state is recorded as a `Status:` line near the top of each issue file (see `triage-labels.md` for the role strings)
> - Comments and conversation history append to the bottom of the file under a `## Comments` heading
<!--翻译-->
> ## 约定
>
> - 每个功能一个目录：`.scratch/<feature-slug>/`
> - 规格文件为 `.scratch/<feature-slug>/spec.md`
> - 实现议题是每个票据一个文件，位于 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，从 `01` 开始编号，绝不使用单个合并的票据文件
> - 分类状态记录在每个议题文件顶部附近的 `Status:` 行中（角色字符串见 `triage-labels.md`）
> - 评论和对话历史追加到文件底部 `## Comments` 标题下

---

<!--原文-->
> ## When a skill says "publish to the issue tracker"
>
> Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).
<!--翻译-->
> ## 当技能说"发布到议题追踪器"
>
> 在 `.scratch/<feature-slug>/` 下创建新文件（必要时创建目录）。

---

<!--原文-->
> ## When a skill says "fetch the relevant ticket"
>
> Read the file at the referenced path. The user will normally pass the path or the issue number directly.
<!--翻译-->
> ## 当技能说"获取相关票据"
>
> 读取引用路径上的文件。用户通常直接传递路径或议题编号。

---

<!--原文-->
> ## Wayfinding operations
>
> Used by `/wayfinder`. The **map** is a file with one **child** file per ticket.
>
> - **Map**: `.scratch/<effort>/map.md` (the Notes / Decisions-so-far / Fog body).
> - **Child ticket**: `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`, with the question in the body. A `Type:` line records the ticket type (`research`/`prototype`/`grilling`/`task`); a `Status:` line records `claimed`/`resolved`.
> - **Blocking**: a `Blocked by: NN, NN` line near the top. A ticket is unblocked when every file it lists is `resolved`.
> - **Frontier**: scan `.scratch/<effort>/issues/` for files that are open, unblocked, and unclaimed; first by number wins.
> - **Claim**: set `Status: claimed` and save before any work.
> - **Resolve**: append the answer under an `## Answer` heading, set `Status: resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
<!--翻译-->
> ## 路标操作（Wayfinding）
>
> 由 `/wayfinder` 使用。**地图**是一个文件，每个票据一个**子**文件。
>
> - **地图**：`.scratch/<effort>/map.md`（Notes / Decisions-so-far / Fog 正文）。
> - **子票据**：`.scratch/<effort>/issues/NN-<slug>.md`，从 `01` 开始编号，问题在正文中。`Type:` 行记录票据类型（`research`/`prototype`/`grilling`/`task`）；`Status:` 行记录 `claimed`/`resolved`。
> - **阻塞**：顶部附近的 `Blocked by: NN, NN` 行。当其所列出的每个文件都是 `resolved` 时，票据解除阻塞。
> - **前沿**：扫描 `.scratch/<effort>/issues/` 中开放、未阻塞且未认领的文件；按编号的第一个获胜。
> - **认领**：在任何工作前设置 `Status: claimed` 并保存。
> - **解决**：在 `## Answer` 标题下追加答案，设置 `Status: resolved`，然后将上下文指针（gist + 链接）追加到 `map.md` 中地图的 Decisions-so-far。
