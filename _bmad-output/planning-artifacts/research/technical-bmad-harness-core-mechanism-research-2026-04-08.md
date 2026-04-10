---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments: []
workflowType: 'research'
lastStep: 1
research_type: 'technical'
research_topic: 'BMad 内部工作流机制 + Harness Sense→Decision 连接机制'
research_goals: '深入理解 BMad 角色/工作流/状态链/文档类型/质量校验，明确 Harness 如何感知 BMad 状态、读取 GitHub Board、拾取 Ready Story、编码 DoR/DoD 边界，为 MVP 连接方案提供设计依据'
user_name: 'Sue'
date: '2026-04-08'
web_research_enabled: true
source_verification: true
---

# Research Report: technical

**Date:** 2026-04-08
**Author:** Sue
**Research Type:** technical

---

## Research Overview

[Research overview and methodology will be appended here]

---

## Technical Research Scope Confirmation

**Research Topic:** BMad-method 内部工作流机制 + Harness Sense→Decision 连接机制
**Research Goals:** 深入理解 BMad-method 角色/工作流/状态链/文档类型/质量校验，明确 Harness 如何感知 BMad-method 状态、读取 GitHub Board、拾取 Ready Story、编码 DoR/DoD 边界，为 MVP 连接方案提供设计依据

**Technical Research Scope:**

- Architecture Analysis - BMad-method 角色体系、Skill 调度机制、状态链流转设计
- Implementation Approaches - create-prd 11 步工作流、dev-story 执行流程、Story 生命周期
- Technology Stack - Claude Code 运行机制、Skill/Workflow 文件结构、config.yaml 驱动方式
- Integration Patterns - BMad-method 状态对外暴露、Harness 感知 Done 信号、GitHub Board 读取与 Ready Story 拾取
- Performance Considerations - DoR/DoD 边界划分、BMad-method Session 生命周期、Harness 介入点

**Research Methodology:**

- Current web data with rigorous source verification
- Multi-source validation for critical technical claims
- Confidence level framework for uncertain information
- Comprehensive technical coverage with architecture-specific insights
- 结合项目内已安装的 BMad-method 文件做实证分析

**Scope Confirmed:** 2026-04-08

---

## 技术栈分析

### BMad-method 运行时架构

BMad-method（全称 BMad Method Agile-AI Driven Development）是一个开源的 AI 驱动敏捷开发框架，当前版本 v6.2.2，GitHub 仓库约 43,900 stars。其核心定位是"将人类经验编码为结构化约束"，让 AI Agent 在约束中作为领域专家引导人类完成开发全流程。

_运行时宿主：_ Claude Code（主要）、Cursor、Windsurf、Codex 等 IDE
_安装方式：_ `npx bmad-method install`（Node.js v20+、Python 3.10+、uv 包管理器）
_核心语言：_ Markdown（Skill/Workflow 定义）、YAML（配置与状态）、Python（bmad_init.py 配置管理脚本）
_Source:_ https://github.com/bmad-code-org/BMAD-METHOD

**关键发现：** BMad-method 本质上是一个"纯 Prompt 工程框架"——所有逻辑都编码在 Markdown/YAML 文件中，由 LLM 在运行时解释执行。没有传统意义的编程语言运行时，唯一的可执行代码是 `bmad_init.py` 配置管理脚本。

### 模块体系与 Skill 系统

BMad-method 采用模块化架构，本项目安装了 4 个模块：

| 模块 | 版本 | 用途 |
|------|------|------|
| `core` | 6.2.2 | 通用工具（brainstorming、review、distillator 等） |
| `bmm`（BMad Method） | 6.2.2 | 完整 SDLC 工作流（4 阶段 34+ 技能） |
| `bmb`（BMad Builder） | 1.1.0 | 元工具：构建自定义 Agent/Workflow/Module |
| `cis`（Creative Intelligence Suite） | 0.1.9 | 创意智能：头脑风暴、设计思维、创新策略等 |

_Skill 部署模式：_ 每个 Skill 是一个目录，包含 `SKILL.md`（触发定义）+ `workflow.md`（执行逻辑）。安装器将 `_bmad/` 源文件部署到 IDE 特定目录（`.claude/skills/`、`.cursor/skills/` 等）。

_Agent 角色体系：_ 9 个核心 BMM Agent + 6 个 CIS Agent，每个 Agent 有独立人格名和领域专长：

| Agent | 人格名 | 阶段 | 核心能力 |
|-------|--------|------|----------|
| bmad-agent-analyst | Mary | 分析 | 业务分析、市场/领域研究、产品简报 |
| bmad-agent-pm | John | 规划 | PRD 创建/验证/编辑、Epic 分解 |
| bmad-agent-ux-designer | Sally | 规划 | UX 设计规范 |
| bmad-agent-architect | Winston | 方案 | 架构设计、实施就绪检查 |
| bmad-agent-dev | Amelia | 实施 | Story 开发（TDD）、代码审查 |
| bmad-agent-qa | Quinn | 实施 | E2E/API 测试生成 |
| bmad-agent-sm | Bob | 实施 | Sprint 规划、Story 创建、回顾 |
| bmad-agent-quick-flow-solo-dev | Barry | 实施 | 快速开发（低仪式感） |
| bmad-agent-tech-writer | Paige | 分析 | 技术文档、Mermaid 图表 |

### 状态管理与数据持久化

BMad-method 的状态管理完全基于文件系统，无数据库依赖：

**1. 文档 Frontmatter（Step-File 工作流）**
```yaml
---
stepsCompleted: [1, 2, 3]    # 已完成步骤数组
inputDocuments: []             # 已加载的输入文档
workflowType: 'prd'           # 工作流标识
---
```
用于工作流续接——若用户中断后恢复，step-01b 读取 `stepsCompleted` 路由到正确步骤。

**2. Sprint Status 文件（`sprint-status.yaml`）**
实施阶段的核心状态追踪器，由 `bmad-sprint-planning` 生成，被 `bmad-create-story`、`bmad-dev-story`、`bmad-code-review` 原地更新。

```
Story 状态机：backlog → ready-for-dev → in-progress → review → done
Epic 状态机：backlog → in-progress → done
规则：永不降级已推进的状态
```

**3. Story 文件（独立 .md 文件）**
每个 Story 是独立的上下文容器，包含：验收标准、任务清单、Dev Agent Record、File List、Change Log、Status 字段。

**4. `project-context.md`（项目宪法）**
由 `bmad-generate-project-context` 生成，被 7 个工作流在激活时自动加载。携带项目级编码标准、模式和约束。

**关键发现：** 所有状态都是文件系统上的 Markdown/YAML 文件。这意味着 Harness 可以通过读取文件系统来感知 BMad-method 的状态——这是 R4（Sense→Decision 机制）的直接设计依据。

### 开发工具与工作流引擎

BMad-method 内部存在两种工作流引擎模式：

**模式 A：Step-File 架构（用户交互式）**
- 适用：`create-prd`（11步）、`create-architecture`（8步）、`create-epics-and-stories`（4步）、`validate-prd`（13步）
- 特征：每步是独立文件，JIT 加载；用户必须选择 `[C] Continue` 推进；输出文档追加式构建
- 状态追踪：文档 frontmatter 中的 `stepsCompleted` 数组

**模式 B：XML Workflow 架构（自主执行式）**
- 适用：`dev-story`（10步）、`sprint-planning`（5步）、`create-story`（6步）、`code-review`（4步）
- 特征：步骤定义为 `<step n="N">` XML 块；`<action>`、`<check>`、`<output>`、`<ask>`、`<critical>`、`<goto>` 指令
- 执行方式：非停顿连续执行，仅在显式 HALT 条件时暂停（外部依赖、连续3次失败、歧义）

**关键发现：** 模式 B 的自主执行特性是 Harness 自动推进的基础——`dev-story` 本身就设计为"一旦启动，自主运行到完成"。Harness 只需触发启动和感知完成。

### 运行环境与部署

BMad-method 无独立服务端，完全运行在 IDE 的 AI Agent 运行时中：

| 平台 | 角色 | 能力 |
|------|------|------|
| Claude Code | 主运行时 | 执行 Skill/Workflow、文件读写、bash 命令 |
| HappyCapy | 交互/调度平台 | 实时对话、Automations 定时触发 |
| GitHub | CI/PM 平台 | Actions CI、PR Review、Projects Board、Issues 追踪 |

_部署拓扑：_ `_bmad/` 是源（authoritative），`.claude/skills/` 是 IDE 部署副本。安装器负责从源同步到 IDE 目录。

_Source:_ 本地文件系统实证分析 + https://github.com/bmad-code-org/BMAD-METHOD

### 技术演进趋势

- **社区活跃度高：** 134 位贡献者、1,779 次提交、多语言 README（EN/CN/VN），表明全球化采用
- **生态扩展中：** 第三方项目已出现（`bmad_automated` CLI 自动化、`antigravity-bmad-config` 工作流模板等），预示向自动化编排方向演进
- **模块化设计支持扩展：** `bmb` 模块提供元工具能力，允许社区构建自定义模块，Harness 可作为新模块接入
- **多 IDE 支持策略：** 同一 Skill 部署到多个 IDE 目录，框架本身与 IDE 解耦

_Source:_ https://github.com/bmad-code-org/BMAD-METHOD + 第三方 GitHub 项目搜索

---

## 集成模式分析

### BMad-method 完整状态机与跨工作流联动

通过本地实证分析（读取项目内已安装的 4 个 BMad workflow 源文件），确认了完整的状态流转链：

```
sprint-planning ─→ sprint-status.yaml 生成
                    epic-N: backlog
                    N-M-story-name: backlog
                    epic-N-retrospective: optional
        │
        ▼
create-story ─→ 自动发现第一个 backlog story
        │
        ├── 若为 epic 首个 story: epic-N: backlog → in-progress
        └── N-M-story: backlog → ready-for-dev
            + 生成 story 文件 (Status: ready-for-dev)
        │
        ▼
dev-story ─→ 自动发现第一个 ready-for-dev story
        │
        ├── Step 4: ready-for-dev → in-progress（双写 yaml + story file）
        ├── Steps 5-8: TDD 实现，标记 task checkboxes [x]
        └── Step 9: in-progress → review（DoD 校验通过后）
        │
        ▼
code-review ─→ review → done
```

_Source:_ 本地文件实证 — `.claude/skills/bmad-{sprint-planning,create-story,dev-story,sprint-status}/workflow.md`

### sprint-status.yaml 精确 Schema

```yaml
generated: <datetime>
last_updated: <datetime>
project: <project_name>
project_key: NOKEY
tracking_system: file-system
story_location: "<implementation_artifacts path>"

development_status:
  epic-1: backlog                    # Epic 状态
  1-1-user-authentication: done      # Story 状态
  1-2-account-management: ready-for-dev
  epic-1-retrospective: optional     # 回顾状态
  epic-2: backlog
  2-1-personality-system: backlog
  epic-2-retrospective: optional
```

**Key 分类规则：**
- Epic: 以 `epic-` 开头且不以 `-retrospective` 结尾
- Retrospective: 以 `-retrospective` 结尾
- Story: 其余一切（匹配 `number-number-name` 模式）

**状态值域：**

| 实体 | 有效状态 | 遗留映射 |
|------|----------|----------|
| Story | `backlog` → `ready-for-dev` → `in-progress` → `review` → `done` | `drafted` → `ready-for-dev` |
| Epic | `backlog` → `in-progress` → `done` | `contexted` → `in-progress` |
| Retro | `optional` ↔ `done` | -- |

**关键设计特性：** 永不降级已推进的状态。`sprint-planning` 在重新生成时会保留已有的更高状态。

### Story 文件精确结构

```markdown
# Story {epic}.{story}: {title}

Status: ready-for-dev            ← 纯文本行，非 YAML frontmatter

## Story
As a {role}, I want {action}, so that {benefit}.

## Acceptance Criteria
1. [验收标准]

## Tasks / Subtasks
- [ ] Task 1 (AC: #)              ← Markdown checkbox 追踪
  - [ ] Subtask 1.1

## Dev Notes
- 架构约束、源码路径、测试标准

## Dev Agent Record
### Agent Model Used
### Debug Log References
### Completion Notes List
### File List
```

**Harness 感知接口：** Status 字段是纯文本行 `Status: {value}`，位于文件第 3 行。可用简单正则 `/^Status:\s*(.+)$/m` 读取。

### dev-story 自主执行与 HALT 机制

**自动发现逻辑（Step 1）：**
- 有 sprint-status.yaml → 从上到下扫描 `development_status`，找第一个 `ready-for-dev` 的 story key
- 无 sprint-status.yaml → 扫描 `{implementation_artifacts}/*.md`，逐文件检查 Status 行
- 无可用 story → HALT，提供菜单（运行 create-story / 指定路径 / 显示状态）

**HALT 条件（仅这些情况暂停）：**
1. Story 文件不可访问
2. 任务/子任务需求歧义
3. 需要超出 spec 的新依赖（需用户审批）
4. 连续 3 次实现失败
5. 缺少必要配置文件
6. Step 9 最终门控失败（未完成任务、回归失败、缺少文件清单、DoD 不通过）

**关键发现：** dev-story 设计为"无停顿连续执行"——除了上述 HALT 条件外，不会为里程碑、进度或会话边界停下。这意味着 Harness 触发 dev-story 后，可以预期它自主运行到 `review` 状态或 HALT。

### sprint-status 数据模式（机器可读接口）

`bmad-sprint-status` 提供三种执行模式：

| 模式 | 用途 | 输出 |
|------|------|------|
| `interactive` | 人类交互 | 可视化报告 + 交互菜单 |
| `data` | 工作流间调用 | 结构化 key=value（next_workflow_id, next_story_id, counts, risks） |
| `validate` | 文件完整性检查 | `is_valid = true/false` + error/suggestion |

**Data 模式输出字段：**
- `next_workflow_id` — 推荐的下一个工作流
- `next_story_id` — 推荐的下一个 story key
- `count_backlog`, `count_ready`, `count_in_progress`, `count_review`, `count_done` — 各状态计数
- `epic_backlog`, `epic_in_progress`, `epic_done` — Epic 状态计数
- `risks` — 风险检测结果

**关键发现：** Data 模式是 Harness 的天然感知接口——它已经提供了"下一步该做什么"的决策数据。Harness 无需自己解析 sprint-status.yaml，直接调用 `sprint-status data` 即可获取结构化状态。

### GitHub Projects v2 API 集成能力

_Source:_ https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-api-to-manage-projects

**API 体系：** Projects v2 使用 GraphQL API（无 REST 端点用于 item 操作）

**读取项目状态：**
```graphql
query {
  node(id: "PROJECT_ID") {
    ... on ProjectV2 {
      items(first: 50) {
        nodes {
          id
          fieldValues(first: 8) {
            nodes {
              ... on ProjectV2ItemFieldSingleSelectValue {
                name   # e.g. "Ready for Dev"
                field { ... on ProjectV2FieldCommon { name } }
              }
            }
          }
          content { ... on Issue { number title url } }
        }
      }
    }
  }
}
```

**gh CLI 操作：**
- `gh project item-list NUMBER --owner ORG --format json` — 列出所有 items（JSON）
- `gh project item-edit --id ITEM_ID --field-id FIELD_ID --single-select-option-id OPTION_ID` — 更新状态

**关键约束：**

| 约束 | 影响 |
|------|------|
| 无服务端字段过滤 | 必须获取全部 items 后客户端过滤 |
| `GITHUB_TOKEN` 无法访问 Projects | 必须使用 PAT 或 GitHub App token（`project` scope） |
| Webhook 仅限 org 级别 | 接收端需公网 HTTPS 端点 |
| Webhook `edited` payload 不含新值 | 需额外 GraphQL 查询获取当前状态 |
| Projects v2 webhook 仍在 public preview | API 可能变化 |

### Harness 感知模式对比

基于以上分析，Harness 有三条感知路径：

**路径 A：文件系统直读（最简单，MVP 推荐）**
```
Harness → 读取 sprint-status.yaml → 解析 development_status
        → 找到 ready-for-dev story → 触发 dev-story
```
- 优点：零外部依赖，直接读 YAML，BMad-method 已有完整状态
- 缺点：仅适用于 Harness 与 BMad-method 在同一文件系统（HappyCapy 环境满足）
- 实现复杂度：低

**路径 B：BMad sprint-status data 模式调用（推荐中期方案）**
```
Harness → 调用 sprint-status(mode=data) → 获取结构化状态
        → 根据 next_workflow_id 决策 → 触发对应工作流
```
- 优点：复用 BMad-method 内置决策逻辑，包含风险检测和推荐
- 缺点：需要在 Harness session 内触发 BMad skill 调用
- 实现复杂度：中

**路径 C：GitHub Projects Board 轮询（适用于跨系统协作）**
```
Harness → gh project item-list → 过滤 "Ready for Dev" → 匹配 story key
        → 触发 dev-story → 完成后更新 Board 状态
```
- 优点：支持人工在 Board 上拖拽触发、跨系统可见性
- 缺点：需要 PAT/App token、GraphQL 查询、Board↔sprint-status 双向同步
- 实现复杂度：高

### DoR/DoD 边界划分

基于 BMad-method 内部校验机制的分析：

**Definition of Ready（DoR）— Harness 拾取前置条件：**
1. sprint-status.yaml 中 story 状态为 `ready-for-dev`
2. 对应 story 文件存在于 `{implementation_artifacts}/`
3. Story 文件 Status 行为 `ready-for-dev`
4. Story 文件包含完整的 Acceptance Criteria 和 Tasks 节
5. project-context.md 存在（dev-story Step 2 需要加载）

**Definition of Done（DoD）— dev-story Step 9 内置校验：**
1. 所有 Tasks/Subtasks checkboxes 标记为 `[x]`
2. 完整测试套件通过（单元 + 集成 + E2E）
3. Linting 通过
4. Dev Agent Record 已更新（File List、Completion Notes）
5. sprint-status.yaml 中状态更新为 `review`
6. Story 文件 Status 行更新为 `review`

**Harness 介入点设计：**
- **Sense 点：** sprint-status.yaml 中出现 `ready-for-dev` story
- **Decision 点：** 验证 DoR 条件是否全部满足
- **Action 点：** 触发 `bmad-dev-story` skill
- **Monitor 点：** 等待 story 状态变为 `review` 或检测 HALT
- **Complete 点：** 触发 `bmad-code-review` → story 变为 `done`

---

## 架构评估与性能考量

### 架构方案评估矩阵

基于前述技术栈和集成模式分析，评估 Harness Sense→Decision→Action 的三种架构方案：

| 评估维度 | A. 文件直读 + Skill 触发 | B. sprint-status data + 编排 | C. GitHub Board 驱动 |
|----------|--------------------------|------------------------------|---------------------|
| **实现复杂度** | 低（YAML 解析 + 正则匹配） | 中（Skill 调用协议 + 状态解析） | 高（GraphQL + 双向同步） |
| **感知延迟** | 即时（文件系统读取） | 秒级（Skill 执行开销） | 分钟级（轮询间隔）或秒级（webhook） |
| **决策质量** | 基础（自建逻辑） | 高（复用 BMad 风险检测 + 推荐） | 基础（仅状态匹配） |
| **BMad 侵入性** | 零（只读文件） | 低（调用已有接口） | 零（独立系统） |
| **外部依赖** | 无 | 无 | PAT/App token + GitHub API |
| **可观测性** | 需自建 | BMad 内置风险报告 | GitHub Board 天然可视 |
| **多人协作** | 弱（文件锁冲突风险） | 弱（同上） | 强（Board 是共享视图） |
| **扩展到 CI** | 需额外集成 | 需额外集成 | 天然（Actions 同生态） |

### MVP 推荐架构：A+B 混合

```
┌─────────────────────────────────────────────────┐
│                  Harness Session                 │
│                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │  Sense   │───→│ Decision │───→│  Action  │   │
│  │          │    │          │    │          │   │
│  │ 读取     │    │ DoR 校验 │    │ 触发     │   │
│  │ sprint-  │    │ + risk   │    │ bmad-    │   │
│  │ status   │    │ 检测     │    │ dev-     │   │
│  │ .yaml    │    │          │    │ story    │   │
│  └──────────┘    └──────────┘    └──────────┘   │
│       │                               │          │
│       │         ┌──────────┐          │          │
│       └────────→│ Monitor  │←─────────┘          │
│                 │          │                      │
│                 │ 轮询     │                      │
│                 │ story    │                      │
│                 │ Status   │                      │
│                 │ 变化     │                      │
│                 └────┬─────┘                      │
│                      │                            │
│                 ┌────▼─────┐                      │
│                 │ Complete │                      │
│                 │          │                      │
│                 │ 触发     │                      │
│                 │ code-    │                      │
│                 │ review   │                      │
│                 └──────────┘                      │
└─────────────────────────────────────────────────┘
```

**推荐理由：**
- Sense 用路径 A（文件直读）：最简单，零依赖，HappyCapy 环境下文件系统共享
- Decision 用路径 B 的思路（但自建轻量版）：DoR 校验逻辑足够简单，无需调用完整 sprint-status skill
- Action 直接触发 BMad skill：`dev-story` 自主执行，Harness 只需启动
- Monitor 用文件轮询：定期读 sprint-status.yaml 和 story 文件 Status 行

### 性能与边界考量

**1. Session 生命周期约束**
- HappyCapy session 有上下文窗口限制（自动压缩旧消息）
- BMad dev-story 单次执行可能消耗大量 context（TDD 循环、测试输出、代码生成）
- **建议：** 每个 story 使用独立 session 或 agent，避免 context 累积
- **风险：** 若 dev-story HALT，需要人工介入或新 session 恢复

**2. 并发安全**
- sprint-status.yaml 是单文件，无锁机制
- 若多个 Harness 实例同时操作（理论场景），存在写冲突
- **MVP 约束：** 单一 Harness 实例，串行执行 story，避免并发问题
- **中期方案：** 引入文件锁或使用 GitHub Issue 作为分布式锁

**3. HALT 恢复策略**
- dev-story HALT 时，story 状态停在 `in-progress`
- Harness 需要区分"正在执行"和"已 HALT 需恢复"
- **检测方法：** 设定超时阈值（如 30 分钟无状态变化）→ 判定为 HALT
- **恢复策略：** 通知人工 / 新 session 重新触发 dev-story（它会检测 `in-progress` 并续接）

**4. BMad-method 版本兼容**
- 当前分析基于 v6.2.2，BMad-method 活跃开发中
- sprint-status.yaml schema 已有遗留映射机制（`drafted` → `ready-for-dev`）
- **建议：** Harness 解析 sprint-status.yaml 时使用 BMad 相同的遗留映射规则
- **风险：** 未来版本可能新增状态值或改变 schema

**5. GitHub Board 双向同步（远期）**
- 若需要 Board 可视化，需解决状态映射：
  - BMad `backlog` ↔ Board "Backlog" column
  - BMad `ready-for-dev` ↔ Board "Ready for Dev" column
  - BMad `in-progress` ↔ Board "In Progress" column
  - BMad `review` ↔ Board "Review" column
  - BMad `done` ↔ Board "Done" column
- 同步方向：BMad → Board（单向推送）为 MVP，双向同步为远期
- **约束：** GitHub Projects v2 无服务端过滤，每次同步需拉取全量 items

---

## 结论与建议

### 核心研究结论

**R1 — BMad-method 内部工作流机制已完整解析：**
- 4 阶段 34+ 技能的完整 SDLC 框架，纯 Prompt 工程实现
- 双工作流引擎：Step-File（交互式）+ XML Workflow（自主执行式）
- 状态完全基于文件系统（sprint-status.yaml + story files + frontmatter）
- 6 个 Agent 角色覆盖分析→规划→方案→实施全流程

**R2 — Harness Sense→Decision 连接机制设计依据已明确：**
- **Sense 信号源：** sprint-status.yaml 的 `development_status` 字段
- **Decision 依据：** DoR 5 项前置条件 + story key 的 `ready-for-dev` 状态
- **Action 触发：** 直接调用 `bmad-dev-story` skill（自主执行到 review 或 HALT）
- **Monitor 机制：** 文件轮询 sprint-status.yaml + story file Status 行
- **Complete 触发：** story 到达 `review` 后触发 `bmad-code-review`

**R3 — MVP 架构方案已确定：**
- A+B 混合：文件直读感知 + 轻量 DoR 校验 + Skill 触发 + 文件轮询监控
- 零外部依赖，完全在 HappyCapy 环境内运行
- 串行执行，每 story 独立 session

### MVP 实施路线建议

**Phase 1: Sense + Decision（最小可行感知）**
1. 实现 sprint-status.yaml 解析器（读取 development_status，识别 ready-for-dev stories）
2. 实现 DoR 校验（5 项前置条件检查）
3. 实现 story 文件 Status 行读取
4. 输出：可感知 BMad 状态的 Harness 核心模块

**Phase 2: Action + Monitor（最小可行执行）**
1. 实现 dev-story Skill 触发机制
2. 实现文件轮询 Monitor（检测 status 变化 + HALT 超时）
3. 实现 code-review 触发（story 到达 review 后）
4. 输出：可自动推进单个 story 的 Harness

**Phase 3: Loop + Recovery（连续执行）**
1. 实现"完成一个 story → 感知下一个 → 继续执行"循环
2. 实现 HALT 恢复策略（超时检测 + 通知 + 新 session 续接）
3. 实现 Sprint 级别进度汇报
4. 输出：可连续推进整个 Sprint 的 Harness

**Phase 4: Visibility（远期，可选）**
1. GitHub Board 单向同步（BMad → Board）
2. 人工在 Board 上的操作反向感知
3. CI 状态集成

### 风险与缓解

| 风险 | 级别 | 缓解策略 |
|------|------|----------|
| dev-story HALT 无法自动恢复 | 高 | 超时检测 + 人工通知 + 新 session 续接 |
| Context 窗口溢出 | 中 | 每 story 独立 session |
| sprint-status.yaml 写冲突 | 低（MVP 串行） | 单实例约束 |
| BMad 版本升级 breaking change | 中 | 遗留映射 + schema 版本检测 |
| GitHub API rate limit | 低（远期才涉及） | 缓存 + 合理轮询间隔 |

### 待后续研究

- HappyCapy Automation 定时触发的精确 API 和能力边界
- Claude Code Agent SDK 的 session 管理和 skill 程序化调用接口
- BMad party-mode 多 agent 协作模式在 Harness 中的应用可能性
- bmad-correct-course 工作流在 HALT 恢复场景的适用性

---

**Research Completed:** 2026-04-08
**Confidence Level:** 高（核心结论基于本地文件实证 + 官方 GitHub 仓库 + GitHub API 文档交叉验证）
