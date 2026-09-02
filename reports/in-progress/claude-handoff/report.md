# claude-handoff — 将对话交接至后台 Agent

## 第一部分：详细总结

### 概述

`claude-handoff` 是一个用户调用的 Skill（`disable-model-invocation: true`），用于将当前对话转交给一个新的后台 Agent 继续工作。与 `handoff` Skill（将摘要保存到系统临时目录）不同，`claude-handoff` 直接启动一个后台 Agent，用摘要作为提示词植入，立即开始工作。这是 in-progress 目录下的测试版 Skill。

### 与 handoff 的核心区别

`handoff` 将交接文档保存到操作系统临时目录，供用户稍后手动传递给新 Agent。`claude-handoff` 则直接通过 `claude --bg` 命令启动后台 Agent，将摘要作为其提示词，新 Agent 在当前工作目录中开始并立即返回控制权。用户通过 `claude agents` 管理后台 Agent。

### 执行方式

核心命令：
```bash
claude --bg --name "<描述性名称>" "<交接摘要>"
```

- 始终传递 `-n` / `--name` 并附带描述性名称（如 `--name "Fix login bug"`）
- 名称用于设置任务列表中显示的标识、会话选择器和终端标题
- Agent 在当前工作目录中启动并立即返回

### 摘要内容要求

交接摘要必须包含：

- 当前工作的上下文和进度状态
- **"建议的 Skill"（suggested skills）部分**：明确指出下一个 Agent 应为哪些 Skill 调用 Skill 工具
- 参考已有产出物（Spec、计划、ADR、Issue、提交、diff）的路径或 URL，不要重复其内容
- 对敏感信息（API 密钥、密码、个人身份信息）进行脱敏处理，因为摘要会成为 Agent 的提示词

### 参数处理

如果用户传递了参数（通过 `argument-hint` 设置："What will the next session be used for?"），将这些参数视为下一个会话聚焦内容的描述，并相应定制摘要。

### 技能元数据

- 类型：用户调用（user-invoked），模型不会自动触发
- 所属目录：`in-progress/`（测试版，公开征集反馈，未在插件中发布）
- 参数提示："What will the next session be used for?"

---

## 第二部分：原文与翻译

<!--原文-->
> # claude-handoff
> Hand the current conversation off to a fresh background agent that picks up the work immediately.

<!--翻译-->
> # claude-handoff
> 将当前对话转交给一个新的后台 Agent，立即接手继续工作。

<!--原文-->
> Write a handoff summary of the current conversation so a fresh agent can continue the work. Instead of saving it, launch a background agent seeded with the summary as its prompt: `claude --bg --name "<descriptive name>" "<handoff summary>"`. It starts in the current working directory and returns immediately; the user manages it with `claude agents`.

<!--翻译-->
> 撰写当前对话的交接摘要，使新 Agent 能够继续工作。不是保存摘要，而是启动一个后台 Agent，用摘要作为其提示词植入：`claude --bg --name "<描述性名称>" "<交接摘要>"`。它在当前工作目录中启动并立即返回；用户通过 `claude agents` 管理它。

<!--原文-->
> Always pass `-n`/`--name` with a descriptive name (e.g. `--name "Fix login bug"`); it sets the display name shown in the job list, session picker, and terminal title.

<!--翻译-->
> 始终传递 `-n` / `--name` 并附带描述性名称（如 `--name "Fix login bug"`）；它设置任务列表、会话选择器和终端标题中显示的标识名称。

<!--原文-->
> Include a "suggested skills" section in the summary, naming which skills the next agent should call the Skill tool for.

<!--翻译-->
> 在摘要中包含一个"建议的 Skill"部分，明确指出下一个 Agent 应为哪些 Skill 调用 Skill 工具。

<!--原文-->
> Do not duplicate content already captured in other artifacts (specs, plans, ADRs, issues, commits, diffs). Reference them by path or URL instead.

<!--翻译-->
> 不要重复其他产出物（Spec、计划、ADR、Issue、提交、diff）中已捕获的内容。通过路径或 URL 引用它们。

<!--原文-->
> Redact any sensitive information, such as API keys, passwords, or personally identifiable information, since the summary becomes the agent's prompt.

<!--翻译-->
> 对任何敏感信息（如 API 密钥、密码或个人身份信息）进行脱敏处理，因为摘要会成为 Agent 的提示词。

<!--原文-->
> If the user passed arguments, treat them as a description of what the next session will focus on and tailor the summary accordingly.

<!--翻译-->
> 如果用户传递了参数，将其视为下一个会话聚焦内容的描述，并相应定制摘要。
