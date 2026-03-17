# OpenCode 领导型协作套件

这是一套“**领导型 Agent + 多角色子代理**”的完整模板。目标是：
- 主代理只负责指挥与协调，不直接修改代码
- PM / 后端 / 前端 / QA 子代理各司其职
- 通过官方 `agent / command / permission / subtask` 机制实现高质量协作

> **重要限制**：当前 OpenCode 以“主代理居中调度”为主，子代理之间没有稳定的直接互聊通道。
>
> **重要说明**：OpenCode 官方并不内建 `Prometheus`。如果你已经安装了 **oh my opencode**，则应优先把本套件视为**兼容复用该插件中的 `Prometheus` 角色契约**；本仓库内的 `prometheus-local.md` 作为本地后备实现，用来保证在没有插件时仍可运行。

---

## ✅ 快速安装

1. 将本目录中的 `opencode.json` 与 `.opencode/` 整体复制到你的项目根目录
2. 如你的项目已有 `opencode.json`，请把本模板中的 `agent` 和 `command` 段合并进去
3. 替换各 `model` 为你在 `/models` 中可用的真实 ID

---

## ✅ 使用方式（推荐流程）

在 OpenCode 中输入：

```
/kickoff 我需要新增订单支付功能，包含前后端联调和测试策略
```

然后系统会自动进入插件优先的“委派流程”：
0. Sisyphus → 默认总协调
1. Prometheus → 产品规划 / 阶段拆解 / 路线图（插件优先）
2. PM → 需求 & 验收标准
3. Backend → API & 数据设计
4. Frontend → UI & 联调方案
5. QA → 测试矩阵 & 回归

最后使用：

```
/sync 汇总成果并给出统一结论
```

---

## ✅ 当前结构应该怎么用（实战版）

当前默认是 **插件优先架构**：
- `default_agent = sisyphus`（依赖 oh my opencode 插件可用）
- 规划优先走插件 `prometheus`
- 执行优先走 `/start-work` → `atlas`
- 本地 `leader` 用于**结构化收敛**

> 关键点：`/start-work` **不会自动回到本地 leader**。如果你走插件执行流，执行完成后需要**显式调用** `/leader` 或 `/sync` 做结构化收敛。
>
> 额外关键点：`/dispatch` 是 **FE/BE-only 强制分发**。如果这一路径里没有先完成 Prometheus / PM / QA，就不要直接把它当成完整 `/sync` 输入；此时应优先用 `/leader` 做 FE/BE-only 收敛，或先 `/kickoff` 补完整链路。

### 用法 1：最推荐的标准流程

适合：复杂需求、跨前后端、希望充分利用 oh my opencode。

步骤：

1. 先规划

```text
@plan 我需要新增订单支付功能，包含前后端联调、异常处理、测试策略
```

或（更明确，直接指定插件 Prometheus）：

```text
/prometheus 我需要新增订单支付功能，包含前后端联调、异常处理、测试策略
```

2. 再执行

```text
/start-work
```

3. 如果你要严格触发前后端不同代理，请在规划里显式写清 owner 与标签

```text
[BE] 实现支付创建接口，owner=backend
[FE] 接入支付确认页和状态流转，owner=frontend
[QA] 补充支付成功/失败/重试测试，owner=qa
```

原因是 `/start-work` 会进入插件 `atlas`，是否将任务交给 `backend` 或 `frontend` 取决于计划中是否有明确标记。

4. 最后做结构化收敛

```text
/leader 请按结构化格式汇总当前结果，检查 VERIFY 表、BLOCK 表、HANDOFF 表，并给出最终方案
```

或：

```text
/sync
```

说明：
- `/start-work` 负责把计划交给 Atlas 执行
- `/leader` / `/sync` 才负责结构化收敛
- 这三步是**手动串联**，不是自动回跳

### 用法 1.5：对话里强制分发前后端代理

适合：你希望当前对话直接把前端和后端交给不同模型，不依赖 Atlas 自行路由。

```text
/dispatch 新增订单支付功能：
- 后端实现支付创建与状态查询接口
- 前端实现支付页、轮询状态与失败重试
- 最后提醒我用 /leader 做 FE/BE-only 汇总
```

这个命令会让插件 `sisyphus` 显式把 FE 任务交给 `frontend`，把 BE 任务交给 `backend`，从而保证你要的模型被正确触发。

注意：
- `/dispatch` 只保证 FE / BE 强制分发
- 如果你还需要产品规划、验收标准、QA 测试矩阵，请改走 `/kickoff`
- `/sync` 默认用于完整链路收敛；FE/BE-only 更适合先走 `/leader`

### 用法 2：你想强制走“本地结构化协议”

适合：你特别在意表格输出、编号、交接包、阻塞表、验证表。

```text
/kickoff 我需要新增订单支付功能，包含前后端联调和测试策略
```

这会强制要求：
- Prometheus 先规划
- PM 引用 PLAN-ID / STAGE-ID
- Backend / Frontend 提交 VERIFY-xxx
- Leader 最后统一收敛

这里的 `/kickoff` 现在会**显式切到 `leader` agent** 执行，不再只是普通模板文本。

### 用法 3：插件 Prometheus 不可用时

适合：插件失效、环境不完整、或你想测试本地后备逻辑。

```text
/prometheus-local 我需要一个分阶段的产品规划与路线图
```

### 用法 4：只想用官方内建 Plan

适合：你想完全绕开插件规划器，只用 OpenCode 官方内建能力。

```text
/plan 请做产品规划，并用 PLAN-ID / STAGE-ID 输出
```

注意：
- 即便使用 `/plan`，后续 PM 仍必须引用 `PLAN-ID / STAGE-ID`。
- `/plan` 属于**替代规划路径**，不要与 `/kickoff` 混用（/kickoff 强制要求 Prometheus）。

### 用法 5：按专项专家增强某一步

适合：某个环节需要专项强化，而不是全流程重跑。

示例：

```text
@oracle 评估当前订单支付架构是否有明显风险
@metis 检查这个规划是否有缺口
@momus 审查这个执行计划是否清晰可落地
使用 `visual-engineering` 类别 + `frontend-design` / `web-design-guidelines` 强化支付页 UI/UX
使用 `writing` 类别 + `doc-coauthoring` / `internal-comms` 整理文档
@hephaestus 处理复杂实现或多文件重构
```

### 用法 6：日常最省心的方式

如果你不想记太多命令，可以直接：

```text
先 @plan
再 /start-work
最后 /leader（或 /sync）
```

你可以把它理解成：

| 阶段 | 推荐入口 |
|---|---|
| 规划 | `@plan` 或 `/prometheus` |
| 执行 | `/start-work` |
| 结构化收敛 | `/leader` |

---

## ✅ 命令入口

- `/leader` → 切换到本地 leader 做结构化收敛
- `/sync` → 推荐的标准收尾命令（进入本地 leader 做结构化收敛）
- `/pm`  → 需求澄清与验收标准
- `/plan` → 官方内建 Plan 规划（兼容模式）
- `/prometheus` → 产品规划 / 阶段拆解 / 路线图
- `/prometheus-local` → Prometheus 本地后备（无插件时使用）
- `/be`  → 后端设计 / 实现
- `/fe`  → 前端设计 / 实现
- `/qa`  → 测试计划 / 回归
- `/dispatch` → 当前对话强制分发前端 / 后端
- `/kickoff` → 自动启动“领导型委派流程”

---

## ✅ 为什么这个模板有效

- 使用 OpenCode 原生 `primary + subagent` 模型
- `Prometheus` 作为**自定义规划代理**承接产品规划、路线图、阶段拆解
- 若已安装 **oh my opencode**，优先复用其 `Prometheus` 角色；本套件保持同名与同结构输出，便于无缝接入
- 用 `permission.task` 明确 leader 只能调哪些子代理
- 用 `permission.edit: deny` 保证 leader 不改代码
- `subtask: true` 让子代理在 child session 独立运行
- 每个角色强制使用 Skill tool（如有可用技能）
- 全局 `permission.* = allow`，确保所有代理可使用通用工具、MCP、Skill、项目分析能力
- PM 具备只读分析型 bash + 本地项目结合能力，可做更深入的规划与头脑风暴
- 所有交付物强制编号 + 表格结构（便于跨角色引用）
- 强制交接包（HANDOFF-xxx）让信息传递不丢失
- 强制阻塞上报（BLOCK-xxx）避免关键前提被猜测
- PM 必须引用 Prometheus 的规划编号（PLAN-ID / STAGE-ID）
- 后端 / 前端如有修改，必须提交 VERIFY-xxx 验证结果表

---

## ✅ 与 oh my opencode 的当前兼容级别

当前这套 agent **可以兼容** oh my opencode，而且兼容方式分两层：

### 第一层：共存兼容
- 本地项目角色：`leader / pm / backend / frontend / qa / prometheus-local`
- 插件角色：`sisyphus / prometheus / atlas / hephaestus / librarian / oracle / metis / momus ...`

两者可以同时存在，因为：
- 本地角色负责**结构化交付契约**
- 插件角色负责**专项能力复用**

### 第二层：复用兼容
Leader 已允许委派给插件角色，并推荐如下映射：

| 场景 | 优先角色 |
|---|---|
| 默认主协调 | `sisyphus` |
| 产品规划 / 路线图 / 阶段拆解 | `prometheus` |
| 规划执行 | `/start-work` → `atlas` |
| UI/UX 设计强化 | `visual-engineering` + 前端设计 skill |
| 文档整稿 / 文案打磨 | `writing` + 文档 skill |
| 深度执行 / 复杂实现 | `hephaestus` |
| 架构判断 | `oracle` |
| 规划缺口分析 | `metis` |
| 规划评审 | `momus` |
| 研究检索 | `librarian` / `explore` |

结论：
> 当前这套 agent **已经能兼容 oh my opencode**，并且已经进入“插件优先 + 本地结构化包装层”的状态，而不是只能并存。

---

## ✅ 插件优先架构（当前默认）

当前默认架构已调整为：

- `default_agent = sisyphus`
- 规划优先走插件 `prometheus`
- 规划执行优先走 `/start-work`（Atlas）
- 本地 `leader` 不再作为默认总控，而是作为**结构化收敛包装层**使用
- 本地 `prometheus-local` 仅在插件不可用时兜底

推荐理解方式：

| 层级 | 角色 |
|---|---|
| 插件主编排层 | `sisyphus / prometheus / atlas / hephaestus / oracle / metis / momus ...` |
| 本地结构化包装层 | `leader / pm / backend / frontend / qa / prometheus-local` |

也就是说：
> 插件负责“会工作”，本地角色负责“按你的格式交付”。

### 为什么之前没有正确触发前后端代理

之前的 `/be` 和 `/fe` 只是手动入口，但推荐流程 `@plan -> /start-work` 的执行者实际上是插件 `atlas`。如果规划结果里没有明确的 `[FE]` / `[BE]` / `owner=frontend|backend` 标记，Atlas 可能直接泛化执行，而不会稳定把任务切给本地 `frontend` / `backend`。

现在建议把三种入口分开理解：

| 场景 | 推荐入口 | 是否强制切 FE/BE 子代理 |
|---|---|---|
| 复杂规划 + 插件执行 | `@plan -> /start-work` | 取决于计划里是否写清 `[FE]/[BE]/owner` |
| 当前对话直接拆给前后端 | `/dispatch ...` | 是 |
| 手动指定某一类实现 | `/be ...` 或 `/fe ...` | 是 |

---

## ✅ 权限策略说明（重要）

本套件使用：

```json
"permission": { "*": "allow" }
```

然后再通过 **agent 级 permission** 做角色约束。

这符合 OpenCode 的权限机制：
- 全局 permission 负责提供默认能力
- agent permission 会与全局合并，并以 agent 规则为准
- `permission.task` 的匹配遵循 **last match wins**

因此，本套件虽然给所有代理开放了 MCP / 工具能力，但仍然通过 agent 限制了：
- Leader 不可 edit / bash
- PM 不可 edit
- QA 不可 edit

---

## ⚠️ 已知边界

- 子代理**不能稳定地再调子代理**（社区已有 feature request）
- 所有跨角色协作必须通过 leader 中转

---

## ✅ 交付格式强化说明（新增）

从本版本开始，所有角色输出必须满足以下硬性规则：

1. **所有核心交付物必须编号**
   - 例如：PRD-001 / API-001 / AC-001 / TC-001 / DEC-001 / BLOCK-001
   - 角色间引用必须用编号，不得用“上面内容/之前提到”这种模糊描述

2. **所有交付物必须表格化**
   - 角色输出不再接受散文式说明
   - 每个模块必须输出结构化表格

3. **必须附带交接包（HANDOFF-xxx）**
   - 每次输出必须给下游提供交接表
   - 交接必须包含“已确定结论 / 待确认项 / 下游动作”

4. **阻塞即上报（BLOCK-xxx）**
    - 缺关键前提时不得继续假设
    - 必须用阻塞表上报并标记能否推进

---

## ✅ Prometheus 约束调整（官方内建 Plan 兼容）

OpenCode 官方并不内建 `Prometheus`，如果你希望完全依赖官方内建代理，可使用 **Plan** 作为兼容替代。

如果你已经安装 **oh my opencode**：
1. 优先继续使用插件提供的 `Prometheus`
2. 本套件中的 `prometheus-local.md` 作为后备与兼容实现
3. 两者都必须遵守同一套输出契约：`PLAN-ID / STAGE-ID / HANDOFF / BLOCK`

兼容方式（推荐）：
1. 使用 `/plan` 命令调用官方内建 Plan
2. 要求 Plan 输出与 Prometheus 相同的 **PLAN-ID / STAGE-ID** 结构

强制要求：
- 即便使用 Plan，PM 仍必须引用 PLAN-ID / STAGE-ID
- Leader 仍按 Prometheus 流程管理规划输出
 - `/plan` 不与 `/kickoff` 混用；`/kickoff` 仍固定走 Prometheus（插件优先）

---

## ✅ 你可以直接开始

复制后即可使用。如果你希望我根据你的项目结构定制角色 prompt（例如后端语言、框架、数据库、测试工具），告诉我你的目录结构即可。
