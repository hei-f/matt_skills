# Wayfinder Skill 详细报告

---

## 第一部分：中文详细总结

### 一、技能目的

Wayfinder 是一个用于规划**远超单次 agent 会话容量**的大型工作的技能。当一个模糊的想法出现，目标太大以至于一次会话装不下，且通往目的地的路径被"战争迷雾"笼罩时，Wayfinder 将其绘制为一张**共享地图（shared map）**，以**决策工单（decision tickets）**的形式存放在仓库的 Issue Tracker 上，然后逐个解决这些工单，直到路径清晰。

与普通开发工单不同，Wayfinder 的工单是**决策工单**——每个工单承载的是一个需要做出**决策**的问题，而不是要执行的一段构建任务。地图完成后，剩下的是一系列已做出的决策，而非可交付的代码。

---

### 二、核心概念

#### 2.1 地图（The Map）

地图是 Issue Tracker 上的**单一工单**，打有 `wayfinder:map` 标签，是整个流程的权威产物。其子工单构成地图的工单集。

地图是一个**索引（index），而非存储（store）**。它列出已做出的决策并指向承载细节的工单——每个决策只存在于一个地方（其工单中），地图仅做摘要和链接，从不重复陈述。

地图正文包含以下五个部分：

- **Destination（目的地）**：一两句话描述到达地图终点意味着什么——这份努力所要通向的规格说明、决策或变更。每个会话在选取工单之前都以此为方向。
- **Notes（备注）**：领域信息、每会话应咨询的技能、此努力的长期偏好等。
- **Decisions so far（已做决策）**：每条已关闭工单一行的索引，包含工单标题链接和一句话答案摘要。
- **Not yet specified（尚未明确）**：战争迷雾——属于范围内但尚无法精确表述为工单的待定问题。
- **Out of scope（范围之外）**：被明确排除在此次努力之外的工作。

地图的定位很关键：它是低分辨率视图，会话只加载地图本体而非每个工单的正文，按需放大查看。这保证了地图可以不断增长，而不会让每个会话为整段历史付出代价。

#### 2.2 工单（Tickets）

每个工单是地图的**子工单（child issue）**，其正文仅包含一个部分：

```
## Question
<该工单所要解决的决策或调查>
```

每个工单打有 `wayfinder:<type>` 标签，类型为 `research`、`prototype`、`grilling`、`task` 之一。

**认领（Claiming）**：会话在开始任何工作之前，先将工单分配给自己（assignee 即认领），其他并发会话会跳过已认领的工单。

**引用方式（Refer by name）**：在所有人类可读的叙述中（包括地图的 Decisions-so-far），始终以**工单标题（名称）**引用工单，绝不使用裸 ID、编号或 slug。一堆 `#42, #43, #44` 无法阅读，而名称可一目了然。ID 和 URL 嵌在名称链接内部，但从不代替名称。

#### 2.3 战争迷雾（Fog of War）

地图是**刻意不完整**的。在活跃工单之外存在"战争迷雾"——你能感知到前方有待做决策和调查，但尚无法精确锁定，因为它们依赖于仍在进行中的问题。

迷雾存放在地图的 **Not yet specified** 部分：疑似的问题、后续需回访的领域。这些是通向目的地的未探索前沿——范围内但不够清晰以形成工单。

**迷雾还是工单的判定标准**：能否**此刻**精确陈述问题，而非能否此刻回答它。

- **形成工单**：问题已足够清晰，即使它被阻塞。
- **留在迷雾**：尚无法精确表述。不要预先把迷雾切成工单大小的碎片——一片迷雾可能在未来孵化出多个工单，也可能一个都没有。

解决一个工单会清除其前方的迷雾，将新变得可明确的事物孵化（graduate）为新鲜工单，一次一个，直到通向目的地的路径清晰且不再有未解决工单。

**Not yet specified** 排除已决策内容（Decisions so far）、已存在的活跃工单和范围外内容（Out of scope）。

#### 2.4 前沿（Frontier）

前沿是**开放、未被阻塞、未被认领**的子工单集合——即已知领域的边缘。阻塞使用 Issue Tracker 的**原生依赖关系**来表达，这很关键，因为它让前沿可以在 Tracker 自身的 UI 中**可视化渲染**，人类无需打开地图就能看到哪些工单可被领取。仅在 Tracker 缺乏原生阻塞机制时才退而使用正文约定。

#### 2.5 范围之外（Out of Scope）

迷雾只会向**目的地**方向聚集。目的地确定了范围，所以超出目的地的工作是"范围之外"——它不是迷雾，不属于 Not yet specified。它在 **Out of scope** 部分拥有自己的位置：被有意识地排除在当前努力之外的工作。

范围之外的工作永远不会孵化（前沿在目的地处停止）。它仅在目的地被重新绘制时才会回来，且此时作为全新努力而非延续。

将其判为范围之外是一种**范围界定行为**，而非路线上的一个步骤。当一个已存在的工单被发现位于目的地之外（制图时范围判断失误，或在解决过程中暴露），应**关闭**它（关闭的工单明确不在前沿上），并在 Out of scope 中留下一行：摘要加为什么出界，链接到该已关闭工单。它不会进入 Decisions so far，后者记录的是实际走过的路线。

---

### 三、工单类型

所有工单分为 **HITL**（Human In The Loop，与人类协作）或 **AFK**（Agent Alone，agent 独立驱动）两种模式。HITL 工单仅通过与能为自己发言的人类的**现场交流**来解决；agent 绝不能代替人类发言（一个自问自答的 grilling agent 已违反此原则）。

| 类型 | 模式 | 触发场景 | 解决方式 |
|------|------|----------|----------|
| **Research（调研）** | AFK | 需要阅读文档、第三方 API 或知识库等当前工作目录之外的知识来获取某个决策所需的事实 | 由调用 research 技能的子 agent 解决，在 chart 阶段并行触发，结果记录在 `research/<name>` 临时分支 |
| **Prototype（原型）** | HITL | "它应该长什么样"或"它应该如何表现"——仅靠对话无法确定的问题。制作廉价、粗糙的具体产物以供反馈（大纲、草稿、存根或 UI/逻辑代码） | 调用 prototype 技能，原型作为资产链接到工单 |
| **Grilling（对话）** | HITL | 默认情况。可以通过对话梳理清楚的问题 | 同时调用 grilling 和 domain-modeling 技能 |
| **Task（任务）** | HITL 或 AFK | 无决策、原型或调研内容，但某项手动工作阻塞了决策的做出：注册服务以便评估其 API、开通权限、移动数据以便观察其形态等 | Agent 能独立完成的就独立驱动（AFK），否则给人类一份精确的 checklist（HITL）。解决方案记录已完成的工作及后续工单依赖的信息（凭证位置、新 URL、行数等） |

**Task 说明**：这是唯一"做"而非"决定"的类型，且它仅因**解除决策阻塞**而获得存在资格，绝不因交付目的地的一部分。这是实践中最容易出错的类型——agent 往往将其解读为实施步骤并开始在地图内编写产品代码。

**Research 是唯一例外**：Research 工单可以并行解决，不受"每会话一个工单"的限制。

---

### 四、Destination（目的地）概念

目的地因努力而异，**命名目的地是制图的第一步**——它在任何工单出现之前确定，因为它确定的范围是每个工单的衡量标准。

目的地可以是：
- 一份待交接和迭代的规格说明
- 在规划开始前锁定的一项决策
- 一份概念验证
- 一处就地完成的变更，如数据结构迁移

地图是领域无关的：工程工作、课程内容、任何符合形态的事物都可以。

---

### 五、Plan, Don't Do（规划，而非执行）

Wayfinder **默认只规划**：每个工单解决一个决策，当地图完成时路径已清晰，不再有任何需要决策的事项，此时应**交接**而非继续编码。想要直接动手的冲动通常是已到达地图边缘的信号。

此外，一个 effort 可以通过在地图的 **Notes** 中声明来覆盖此默认行为，将执行带入地图内部。但默认是产出决策，而非可交付物。

**关键提醒**：Notes 由 agent 编写，约束和豁免权存在于同一文件中，这是一个已知弱点。阅读任何非自己绘制的地图的 Notes，将实现保持在独立会话中，并将任何看起来像构建步骤的 `wayfinder:task` 视为类型错误。

---

### 六、两种模式

#### 6.1 模式一：绘制地图（Chart the Map）

用户在有一个模糊想法时调用。

1. **命名目的地**：调用 grilling 和 domain-modeling 技能，确定此地图要通向什么。
2. **绘制前沿（广度优先）**：再次 grilling，这次**广度优先**——在整个空间上扇形展开，而非在任一线上深入。浮现开放决策和当下可执行的第一步。**如果此时没有浮现任何迷雾**（通向目的地的路径已清晰，整个旅程足够小以在一次会话内完成），则不需要地图，停下来询问用户如何继续。
3. **创建地图**（打上 `wayfinder:map` 标签）：Destination 和 Notes 填写完毕，Decisions-so-far 为空，迷雾草稿写入 Not yet specified。
4. **创建工单并连接阻塞边**：先创建所有当前可明确的工单作为地图的子工单，然后**第二轮**连接阻塞关系（工单需要 ID 才能相互引用）。连接后工单被分类为前沿的和被阻塞的；尚无法明确的留在迷雾中。
5. **触发调研子 agent**：对每个刚创建的 `research` 工单，启动子 agent 并行解决，在 `research/<name>` 分支上捕获发现，并从工单链接上下文。
6. **停止**：绘制是一个会话的工作，它不解决任何工单（除了 research）。

#### 6.2 模式二：在地图中推进（Work through the Map）

用户以地图（URL 或编号）调用。工单可选：不指定时 agent 自行选择下一个决策。

1. **加载地图**：低分辨率视图，不加载每个工单正文。
2. **选择工单**：用户指定则用该工单，否则按顺序取第一个前沿工单。**认领**：在开始任何工作前将工单分配给自己。
3. **解决工单**：**按需放大**——按需获取相关工单或已关闭工单的完整正文；调用 `## Notes` 中列出的技能。如不确定，调用 grilling 和 domain-modeling。
4. **记录解决方案**：将答案作为**解决评论**发布，**关闭**工单，并在地图的 Decisions-so-far 中增加一条摘要链接。
5. **孵化新建**：创建新浮现的工单（先创建再连线）；将答案使变得可明确的迷雾孵化（graduated fog），从 Not yet specified 中清除已孵化的条目，使其仅以新工单的形式存在。如果答案揭示某个工单位于目的地之外，将其**判为范围之外**而非在路线上解决。如果决策使地图的其他部分无效，更新或删除这些工单。

**绝不每会话解决超过一个工单**（research 除外）。用户可能并行执行未被阻塞的工单，因此应预期其他会话在同时编辑 Tracker。

---

### 七、广度优先制图（Breadth-First Mapping）

在绘制阶段，Grilling 应采用**广度优先**策略：在整个空间上扇形展开，而非在任一线上深入。目标是浮现**整个问题空间的开放决策**以及当下可执行的第一步，而不是把一条线索穷尽到底。

这与避免"瀑布陷阱"密切相关：如果在地图早期就深入某条线索制定大量工单，后续工单可能基于会被早期工单推翻的假设。广度优先加**积极原型化**是保持路线有效的关键。

---

### 八、已知问题与陷阱

1. **Agent 在地图内写生产代码**：最常见的失败模式。"Plan, don't do" 的默认可被 Notes 覆盖，且约束与豁免权同在一个 agent 可控的文件中。
2. **工单过多导致后面失效**：27 个工单到第 13 个就不再合理。应对方法：将地图范围限定在有界的 epic 而非整个产品，并积极使用 prototype 提前消除不确定性。
3. **Grilling 冗长耗尽决策精力**：每个问题三段的冗长度造成决策疲劳，且问题失去"为什么"的链条。这是当前模型属性而非技能缺陷，尚无修复。缓解措施：使用更低的推理强度，在全局 CLAUDE.md 中加简洁指令。
4. **并行工单的上下文断裂**：两个 grilling 工单并行时，一个会话问的问题另一个刚回答过。一逐一解决是更安全的默认。并行 prototype 工单时 agent 可能自行选择方案并关闭工单——选择应由人类做出。

---

## 第二部分：SKILL.md 原文与翻译

<!--原文-->
> A loose idea has arrived, too big for one agent session, and wrapped in fog: the way from here to the **destination** isn't visible yet. Wayfinding is about finding that way, not charging at the destination. This skill charts the way as a **shared map** on the repo's issue tracker, then works its **decision tickets** (questions whose resolution is a decision, not slices of a build to execute) one at a time until the route is clear.
<!--翻译-->
> 一个模糊的想法出现了，太大以至于一次 agent 会话装不下，且被迷雾笼罩：从这里通向**目的地**的路径尚不可见。Wayfinding 是关于找到那条路，而非冲向目的地。此技能将路径绘制为仓库 Issue Tracker 上的**共享地图**，然后逐个解决其**决策工单**（其解决方案是一个决策的问题，而非要执行的一段构建），直到路线清晰。

<!--原文-->
> The destination varies per effort, and naming it is the first act of charting: it shapes every ticket. It might be a spec to hand off and iterate on, a decision to lock before planning starts, or a change made in place like a data-structure migration. The map is domain-agnostic: engineering work, course content, whatever fits the shape.
<!--翻译-->
> 目的地因努力而异，命名目的地是制图的第一步：它塑造每一个工单。它可能是一份待交接和迭代的规格说明、一项在规划开始前锁定的决策，或一处就地完成的变更，如数据结构迁移。地图是领域无关的：工程工作、课程内容、任何符合此形态的事物都可以。

<!--原文-->
> ## Plan, don't do
<!--翻译-->
> ## 规划，而非执行

<!--原文-->
> Wayfinder is **planning** by default: each ticket resolves a decision, and the map is done when the way is clear, with nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal you've reached the edge of the map and it's time to hand off. An effort can override this in its **Notes**, carrying execution into the map itself, but absent that, produce decisions, not deliverables.
<!--翻译-->
> Wayfinder 默认只做**规划**：每个工单解决一个决策，当地图完成时路径已清晰，在任何人在去做那件事之前已没有需要决策的事项。想要直接动手的冲动通常是你已到达地图边缘的信号，是时候交接了。一次努力可以通过其 **Notes** 覆盖此默认行为，将执行带入地图内部，但除此之外，产出决策，而非可交付物。

<!--原文-->
> ## Refer by name
<!--翻译-->
> ## 以名称引用

<!--原文-->
> Every map and ticket is an issue, so it has a **name**: its title. In everything the human reads (narration, the map's Decisions-so-far), refer to it by that name, never by a bare id, number, or slug. A wall of `#42, #43, #44` is illegible; names read at a glance. The id and URL don't vanish; a name wraps its link, but they ride _inside_ the name, never stand in for it.
<!--翻译-->
> 每张地图和每个工单都是一个 issue，因此它有一个**名称**：其标题。在人类阅读的所有内容中（叙述、地图的 Decisions-so-far），以该名称引用它，绝不使用裸 ID、编号或 slug。一堆 `#42, #43, #44` 是无法阅读的；名称可一目了然。ID 和 URL 并未消失；名称包裹其链接，但它们内嵌于名称之中，从不替代名称。

<!--原文-->
> ## The Map
<!--翻译-->
> ## 地图

<!--原文-->
> The map is a single issue on this repo's issue tracker, labelled `wayfinder:map`, the canonical artifact. Its tickets are child issues of the map.
<!--翻译-->
> 地图是此仓库 Issue Tracker 上的单一工单，打有 `wayfinder:map` 标签，是权威产物。其工单是地图的子工单。

<!--原文-->
> The map is an **index**, not a store. It lists the decisions made and points at the tickets that hold their detail; a decision lives in exactly one place, its ticket, so the map never restates it, only gists it and links.
<!--翻译-->
> 地图是一个**索引**，而非存储。它列出已做出的决策并指向承载其细节的工单；一个决策只存在于唯一一个地方，即其工单，因此地图从不重复陈述，仅做摘要和链接。

<!--原文-->
> **Where the map, its child tickets, blocking, and frontier queries physically live is tracker-specific.** The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`. Consult the tracker doc's "Wayfinding operations" section for how _this_ repo expresses them. If no tracker has been provided, default to the local-markdown tracker.
<!--翻译-->
> **地图、其子工单、阻塞关系和前沿查询的物理存放位置取决于具体的 Tracker。** Issue Tracker 应已被提供给你。如未提供，告知用户运行 `/setup-matt-pocock-skills`。查阅 Tracker 文档的 "Wayfinding operations" 部分以了解此仓库如何表达这些内容。如果没有配置任何 Tracker，默认使用本地 markdown tracker。

<!--原文-->
> ### The map body
<!--翻译-->
> ### 地图正文

<!--原文-->
> The whole map at low resolution, loaded once per session. Open tickets are **not** listed: they are open child issues, found by query.
<!--翻译-->
> 整个地图的低分辨率视图，每会话加载一次。开放的工单**不**列出：它们是开放的子工单，通过查询找到。

<!--原文-->
> ```markdown
> ## Destination
>
> <what reaching the end of this map looks like: the spec, decision, or change this effort is finding its way to. One or two lines; every session orients to it before choosing a ticket.>
>
> ## Notes
>
> <domain; skills every session should consult; standing preferences for this effort>
>
> ## Decisions so far
>
> <!-- the index: one line per closed ticket, enough to judge relevance, then zoom the link for the detail the ticket holds -->
>
> - [<closed ticket title>](link): <one-line gist of the answer>
>
> ## Not yet specified
>
> <!-- see "Fog of war": in-scope fog you can't ticket yet; graduates as the frontier advances -->
>
> ## Out of scope
>
> <!-- see "Out of scope": work ruled beyond the destination; closed, never graduates -->
> ```
<!--翻译-->
> ```markdown
> ## Destination
>
> <到达此地图终点意味着什么：此努力正在通向的规格说明、决策或变更。一两行；每个会话在选取工单之前以此为方向。>
>
> ## Notes
>
> <领域；每会话应咨询的技能；对此努力的长期偏好>
>
> ## Decisions so far
>
> <!-- 索引：每条已关闭工单一行的摘要，足以判断相关性，然后放大链接查看工单承载的细节 -->
>
> - [<已关闭工单标题>](link)：<答案的一句话摘要>
>
> ## Not yet specified
>
> <!-- 参见"战争迷雾"：范围内但尚无法形成工单的迷雾；随着前沿推进而孵化 -->
>
> ## Out of scope
>
> <!-- 参见"范围之外"：被判定超出目的地的工作；已关闭，永不孵化 -->
> ```

<!--原文-->
> ### Tickets
<!--翻译-->
> ### 工单

<!--原文-->
> Each ticket is a **child issue** of the map; the tracker's issue id is its identity. Its body is the question, sized to one 100K token agent session:
<!--翻译-->
> 每个工单是地图的**子工单**；Tracker 的 issue ID 是其身份标识。其正文是问题，大小适配一个 100K token 的 agent 会话：

<!--原文-->
> ```markdown
> ## Question
>
> <the decision or investigation this ticket resolves>
> ```
<!--翻译-->
> ```markdown
> ## Question
>
> <该工单所要解决的决策或调查>
> ```

<!--原文-->
> Each ticket carries a `wayfinder:<type>` label, one of `research`, `prototype`, `grilling`, `task` (see [Ticket Types](#ticket-types)).
<!--翻译-->
> 每个工单打有 `wayfinder:<type>` 标签，为 `research`、`prototype`、`grilling`、`task` 之一（参见[工单类型](#工单类型)）。

<!--原文-->
> A session **claims** a ticket by assigning it to the dev driving the map, **first**, before any work, so concurrent sessions skip it. That assignee _is_ the claim: an open, unassigned ticket is unclaimed.
<!--翻译-->
> 会话通过将工单分配给驱动地图的开发者来**认领**工单，**先**认领，再做任何工作，这样并发会话会跳过它。那个被分配人就是认领：一个开放、未分配的工单即未被认领。

<!--原文-->
> Blocking uses the tracker's **native** dependency relationship: essential because it renders the frontier _visually_ in the tracker's own UI, so the human sees what's takeable without opening the map. Only a tracker that lacks native blocking falls back to a body convention. A ticket is **unblocked** when every ticket blocking it is closed; the **frontier** is the open, unblocked, unclaimed children, the edge of the known.
<!--翻译-->
> 阻塞使用 Tracker 的**原生**依赖关系：这很关键，因为它让前沿可以在 Tracker 自身的 UI 中可视化渲染，因此人类无需打开地图就能看到哪些工单可被领取。仅在 Tracker 缺乏原生阻塞机制时才退而使用正文约定。当一个工单的所有阻塞工单都已关闭时，它是**未被阻塞的**；**前沿**是开放、未被阻塞、未被认领的子工单集合，即已知领域的边缘。

<!--原文-->
> The answer isn't part of the body; it's recorded on resolution (see [Work through the map](#work-through-the-map)). Assets created while resolving a ticket are linked from the issue, not pasted in.
<!--翻译-->
> 答案不是正文的一部分；它在解决时记录（参见[在地图中推进](#在地图中推进)）。解决工单时创建的资产从 issue 链接，而非粘贴进来。

<!--原文-->
> ## Ticket Types
<!--翻译-->
> ## 工单类型

<!--原文-->
> Every ticket is either **HITL** (human in the loop, worked _with_ a human who speaks for themselves) or **AFK**, driven by the agent alone. A HITL ticket only resolves through that live exchange; the agent never stands in for the human's side of it (a grilling agent that answers its own questions has broken this).
<!--翻译-->
> 每个工单要么是 **HITL**（human in the loop，与能为自己发言的人类协作），要么是 **AFK**（agent 独立驱动）。HITL 工单仅通过现场交流来解决；agent 绝不能代替人类发言（一个自问自答的 grilling agent 已违反此原则）。

<!--原文-->
> - **Research** (AFK): Reading documentation, third-party APIs, or local resources like knowledge bases to surface a fact a decision waits on. Resolved by a subagent that calls the Skill tool with "research". Use when knowledge outside the current working directory is required.
<!--翻译-->
> - **Research（调研）**（AFK）：阅读文档、第三方 API 或知识库等本地资源，以揭示某个决策所需的事实。由调用 research 技能的子 agent 解决。当需要当前工作目录之外的知识时使用。

<!--原文-->
> - **Prototype** (HITL): Raise the fidelity of the discussion by making a cheap, rough, concrete artifact to react to (an outline, a rough take, a stub, or UI/logic code) by calling the Skill tool with "prototype". Links the prototype as an asset. Use when "how should it look" or "how should it behave" is the key question.
<!--翻译-->
> - **Prototype（原型）**（HITL）：通过制作廉价、粗糙、具体的产物以供反馈（大纲、粗稿、存根或 UI/逻辑代码）来提高讨论的保真度，调用 prototype 技能。将原型作为资产链接。当"它应该长什么样"或"它应该如何表现"是关键问题时使用。

<!--原文-->
> - **Grilling** (HITL): Conversation. The default case. Always call the Skill tool twice, for "grilling" and "domain-modeling".
<!--翻译-->
> - **Grilling（对话）**（HITL）：对话。默认情况。始终调用 Skill 工具两次，分别使用 "grilling" 和 "domain-modeling"。

<!--原文-->
> - **Task** (HITL or AFK): Manual work that must happen before a _decision_ can be made: nothing to decide, prototype, or research, but the discussion is blocked until it's done. Signing up for a service so its API can be judged, provisioning access, moving data so its shape can be seen. This is the one type that _does_ rather than decides, and it earns its place by unblocking a decision, not by delivering the destination. The agent drives it alone where it can (AFK); otherwise it hands the human a precise checklist (HITL). Resolved when the work is done; the answer records what was done and any resulting facts (credentials location, new URLs, row counts) later tickets depend on.
<!--翻译-->
> - **Task（任务）**（HITL 或 AFK）：在做出**决策**之前必须发生的手动工作：没有需要决策、原型或调研的内容，但讨论被阻塞直到它完成。注册服务以便评估其 API、开通权限、移动数据以便观察其形态。这是唯一"做"而非"决定"的类型，且它通过解除决策阻塞而非交付目的地来获得其存在资格。Agent 在能独立完成时独立驱动（AFK）；否则交给人类一份精确的 checklist（HITL）。工作完成时即解决；答案记录已完成的工作及后续工单依赖的任何结果信息（凭证位置、新 URL、行数）。

<!--原文-->
> ## Fog of war
<!--翻译-->
> ## 战争迷雾

<!--原文-->
> The map is _deliberately_ incomplete: don't chart what you can't yet see. Beyond the live tickets lies the **fog of war**: the dim view of decisions and investigations you can tell are coming but can't yet pin down, because they hang on questions still open. Resolving a ticket clears the fog ahead of it, graduating whatever's now specifiable into fresh tickets, one at a time, until the way to the destination is clear and no tickets remain.
<!--翻译-->
> 地图是刻意不完整的：不要绘制你还看不到的东西。在活跃工单之外是**战争迷雾**：你能感知到前方的决策和调查，但尚无法精确锁定，因为它们依赖于仍在进行中的问题。解决一个工单会清除其前方的迷雾，将新变得可明确的事物孵化（graduate）为新鲜工单，一次一个，直到通向目的地的路径清晰且不再有工单剩余。

<!--原文-->
> The map's **Not yet specified** section is where that dim view is written down: the suspected question, the area to revisit later. It's the undiscovered frontier _toward_ the destination: everything here is in scope, just not sharp enough to ticket. Write as loosely or as fully as the view allows; it doubles as a signpost for collaborators reading where the effort is headed.
<!--翻译-->
> 地图的 **Not yet specified** 部分是写下那个模糊视野的地方：疑似的问题、稍后需回访的领域。它是通向目的地的未探索前沿：此处的一切都在范围内，只是不够清晰以形成工单。按视野允许的松散或完整程度书写；它同时作为协作者阅读努力走向的路标。

<!--原文-->
> **Fog or ticket?** The test is whether you can state the question precisely now, _not_ whether you can answer it now.
<!--翻译-->
> **迷雾还是工单？** 判定标准是你能否此刻精确陈述问题，而非你能否此刻回答它。

<!--原文-->
> - **Ticket when** the question is already sharp, even if it's blocked and you can't act on it yet.
<!--翻译-->
> - **形成工单**：问题已足够清晰，即使它被阻塞且你还无法对其操作。

<!--原文-->
> - **Not yet specified when** you can't yet phrase it that sharply. Don't pre-slice the fog into ticket-sized pieces: it's coarser than a ticket, and one patch may graduate into several tickets, or none, once the frontier reaches it.
<!--翻译-->
> - **留在 Not yet specified**：尚无法如此精确地表述。不要预先把迷雾切成工单大小的碎片：它比工单更粗糙，一片迷雾可能在前沿到达时孵化出多个工单，也可能一个都没有。

<!--原文-->
> **Not yet specified** excludes what's already decided (Decisions so far), what's already a live ticket, and what's out of scope (the next section).
<!--翻译-->
> **Not yet specified** 排除已决策内容（Decisions so far）、已成为活跃工单的内容以及范围之外的内容（下一节）。

<!--原文-->
> ## Out of scope
<!--翻译-->
> ## 范围之外

<!--原文-->
> Fog only ever gathers _toward_ the destination. The destination fixes the scope, so work beyond it is **out of scope**: it isn't fog, and it doesn't belong in **Not yet specified**. It gets its own **Out of scope** section on the map: work you've consciously ruled out of _this_ effort. Scope, not sharpness, lands it here.
<!--翻译-->
> 迷雾只会向目的地方向聚集。目的地确定了范围，因此超出目的地的工作是**范围之外**：它不是迷雾，也不属于 Not yet specified。它在地图上拥有自己的 **Out of scope** 部分：被你有意识地排除在此次努力之外的工作。是范围而非清晰度将其放在这里。

<!--原文-->
> Out-of-scope work never graduates (the frontier stops at the destination), so it returns only if the destination is redrawn, and then as a fresh effort, not a resumption.
<!--翻译-->
> 范围之外的工作永远不会孵化（前沿在目的地停止），因此它仅在目的地被重新绘制时才会回来，且此时作为全新努力而非延续。

<!--原文-->
> Ruling something out of scope is a scoping act, not a step on the route. When a ticket that already exists turns out to sit past the destination (mis-scoped in while charting, or exposed by a resolution), **close it** (a closed ticket is unambiguously off the frontier) and leave one line in the **Out of scope** section: the gist plus why it's out of scope, linking the closed ticket. It stays out of **Decisions so far**, which records the route actually walked; a scope boundary isn't a step on it.
<!--翻译-->
> 将某物判为范围之外是一种范围界定行为，而非路线上的一个步骤。当一个已存在的工单被发现位于目的地之外（制图时范围判断失误，或在解决过程中暴露），**关闭它**（关闭的工单明确不在前沿上）并在 **Out of scope** 部分留下一行：摘要加为什么出界，链接到该已关闭工单。它不会进入 Decisions so far，后者记录的是实际走过的路线；范围边界不是路线上的一个步骤。

<!--原文-->
> ## Invocation
<!--翻译-->
> ## 调用

<!--原文-->
> Two modes. Either way, **never resolve more than one ticket per session**, with the exception of research tickets.
<!--翻译-->
> 两种模式。无论哪种，**绝对不要每会话解决超过一个工单**，research 工单除外。

<!--原文-->
> ### Chart the map
<!--翻译-->
> ### 绘制地图

<!--原文-->
> User invokes with a loose idea.
<!--翻译-->
> 用户以模糊想法调用。

<!--原文-->
> 1. **Name the destination.** Call the Skill tool twice, for "grilling" and "domain-modeling", to pin down what this map is finding its way to: the spec, decision, or change. The destination fixes the scope, so it's settled first.
<!--翻译-->
> 1. **命名目的地。** 调用 Skill 工具两次，分别使用 "grilling" 和 "domain-modeling"，以确定此地图要通向什么：规格说明、决策或变更。目的地确定范围，因此它最先被确定。

<!--原文-->
> 2. **Map the frontier.** Grill again, **breadth-first** this time: fan out across the whole space rather than deep on any one thread, surfacing the open decisions and the first steps takeable now. **If this surfaces no fog** (the way to the destination is already clear, the whole journey small enough for one session), you don't need a map. Stop and ask the user how they'd like to proceed.
<!--翻译-->
> 2. **绘制前沿。** 再次 grilling，这次**广度优先**：在整个空间上扇形展开，而非在任一线上深入，浮现开放决策和当下可执行的第一步。**如果这没有浮现任何迷雾**（通向目的地的路径已清晰，整个旅程足够小以在一次会话内完成），你不需要地图。停下来询问用户希望如何继续。

<!--原文-->
> 3. **Create the map** (label `wayfinder:map`): Destination and Notes filled in, Decisions-so-far empty, the fog sketched into **Not yet specified**.
<!--翻译-->
> 3. **创建地图**（标签 `wayfinder:map`）：Destination 和 Notes 填写完毕，Decisions-so-far 为空，迷雾草稿写入 **Not yet specified**。

<!--原文-->
> 4. **Create the tickets you can specify now** as child issues of the map, then wire blocking edges in a **second pass** (issues need ids before they can reference each other). Wiring sorts them into the frontier and the blocked; everything you can't yet specify stays in the fog: the **Not yet specified** section.
<!--翻译-->
> 4. **创建当前可明确的工单**作为地图的子工单，然后**第二轮**连接阻塞边（工单需要 ID 才能相互引用）。连接将工单分类为前沿的和被阻塞的；尚无法明确的留在迷雾中：即 **Not yet specified** 部分。

<!--原文-->
> 5. **Fire the research subagents.** For each `research` ticket you just created, spin up a subagent that calls the Skill tool with "research" to resolve it in parallel, capturing its findings on a throwaway `research/<name>` branch with a context pointer from the ticket.
<!--翻译-->
> 5. **触发调研子 agent。** 对每个刚创建的 `research` 工单，启动一个调用 research 技能的子 agent 并行解决它，在一次性 `research/<name>` 分支上捕获其发现，并从工单链接上下文指针。

<!--原文-->
> 6. Stop: charting is one session's work; it hand-resolves nothing.
<!--翻译-->
> 6. 停止：绘制是一个会话的工作；它不手动解决任何工单。

<!--原文-->
> ### Work through the map
<!--翻译-->
> ### 在地图中推进

<!--原文-->
> User invokes with a map (URL or number). A ticket is **optional**: without one, you pick the next decision, not the user.
<!--翻译-->
> 用户以地图（URL 或编号）调用。工单是**可选的**：不指定时，由你选择下一个决策，而非用户。

<!--原文-->
> 1. Load the **map**: the low-res view, not every ticket body.
<!--翻译-->
> 1. 加载**地图**：低分辨率视图，而非每个工单的正文。

<!--原文-->
> 2. Choose the ticket. If the user named one, use it. Otherwise take the first frontier ticket in order. **Claim it**: assign it to yourself before any work.
<!--翻译-->
> 2. 选择工单。如果用户指定了，使用该工单。否则按顺序取第一个前沿工单。**认领它**：在开始任何工作前将其分配给自己。

<!--原文-->
> 3. Resolve it. **Zoom as needed**: fetch the full body of any related or closed ticket on demand; call the Skill tool for whichever skills the `## Notes` block names. If in doubt, call the Skill tool twice, for "grilling" and "domain-modeling".
<!--翻译-->
> 3. 解决它。**按需放大**：按需获取任何相关工单或已关闭工单的完整正文；调用 `## Notes` 块中列出的技能。如有疑问，调用 Skill 工具两次，分别使用 "grilling" 和 "domain-modeling"。

<!--原文-->
> 4. Record the resolution: post the answer as a **resolution comment**, **close** the issue, and **append a context pointer** to the map's Decisions-so-far.
<!--翻译-->
> 4. 记录解决方案：将答案作为**解决评论**发布，**关闭**该 issue，并在地图的 Decisions-so-far 中**增加一条上下文指针**。

<!--原文-->
> 5. Add newly-surfaced tickets (create-then-wire); graduate any fog the answer has made specifiable, clearing each graduated patch from **Not yet specified** so it lives only as its new ticket. If the answer reveals that a ticket (this one or another) sits beyond the destination, **rule it out of scope** rather than resolving it on the route. If the decision invalidates other parts of the map, update or delete those tickets.
<!--翻译-->
> 5. 添加新浮现的工单（先创建再连线）；将答案使变得可明确的迷雾孵化，从 Not yet specified 中清除每个已孵化的条目，使其仅以新工单的形式存在。如果答案揭示某个工单（此工单或其他工单）位于目的地之外，将其**判为范围之外**而非在路线上解决。如果决策使地图的其他部分无效，更新或删除这些工单。

<!--原文-->
> The user may run unblocked tickets in parallel, so expect other sessions to be editing the tracker concurrently.
<!--翻译-->
> 用户可能并行执行未被阻塞的工单，因此应预期其他会话在同时编辑 Tracker。
