---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments: []
session_topic: '构建基于BMad/ECC等AI开发规范框架的Harness工程化项目，使AI Coding Agent能够按标准可靠的工作流程自动推进软件项目完整开发交付闭环，并与GitHub等项目管理工具同步跟踪，形成从1到10的持续演进能力'
session_goals: '一套可落地的优选方案'
selected_approach: 'ai-recommended'
techniques_used: ['First Principles Thinking', 'SCAMPER Method', 'Solution Matrix']
ideas_generated: ['自主推进引擎+关键门控', '经验工程化容器(BMad+)', '弹性Agent涌现', '共创-门控确认频谱', '三角架构(Claude Code + HappyCapy + GitHub)', '可插拔约束基座']
context_file: ''
---

# Brainstorming Session Results

**Facilitator:** Sue
**Date:** 2026-04-07

## Session Overview

**Topic:** AI Agent 驱动的端到端软件交付自动化 Harness 工程体系
**Goals:** 一套可落地的优选方案

### Session Setup

本次会话聚焦于如何构建基于 BMad/ECC 等 AI 开发规范框架的 Harness 工程化项目，使 AI Coding Agent（如 Claude Code）能够按照标准可靠的工作流程，在一定范围内自动推进软件项目的完整开发交付闭环，并与 GitHub 等项目管理工具同步跟踪，形成从 1 到 10 的持续演进能力。

## Technique Selection

**Approach:** AI-Recommended Techniques

**Recommended Techniques:**

- **First Principles Thinking:** 打碎固有假设，从最基本的真相重建 Harness 核心构成
- **SCAMPER Method:** 在第一性原理基础上，系统化七维发散生成方案组件
- **Solution Matrix:** 按关键变量轴构建矩阵，收敛输出优选落地方案

---

## Phase 1: First Principles - 底层原理挖掘

### 核心约束与事实

1. **交付的本质 = 可验证的状态转换**：从想法→代码→测试通过→用户可用，每一步都是可验证的状态跳跃
2. **AI Agent 的特性**：全才但经验为零，可弹性伸缩无边际成本，缺乏责任感和直觉
3. **人类的特性**：有经验可举一反三，上下文处理力有限，有刚性成本，有责任感
4. **BMad 的定位**：已经在做"将人类经验编码为结构化约束"这件事，运行在 Claude Code 中

### 推导出的底层原理

**[FP #1] 自主推进引擎 + 关键门控**

不是被动的状态机，而是能自主往前走的引擎。AI Agent 拿到任务后自主推进开发，只在人类定义的关键节点暂停等待人类决策。自动化的不是"流程"，而是"推进力"。

**[FP #2] BMad+ 层：主动性 + 沟通**

BMad 解决了"经验编码为约束"。Harness 要构建的是在此之上的两个能力：
- (a) AI 的主动推进能力——不等指令，自己看 backlog 自己领任务
- (b) GitHub 原生的沟通能力——Issue、PR、Comment 成为 AI 与人类干系人的标准沟通协议

**[FP #3] 弹性 Agent 涌现**

多 Agent 不需要主动编排。推进引擎发现多个 Ready Story 时自然 fork 出多个执行流。伸缩是涌现的、被动的、动态的，而非主动调控机制。

> **修订说明（Party Mode 后）：** 涌现的单位是"阶段级 Agent 组合"，而非同一 Story 内的并行执行。规划阶段由多角色协同（PM、Architect、QA 等）；执行阶段每个 Dev Agent 专注单一 Story，不做内部并行。多个 Story 同时 Ready 时才会 fork 出多个独立执行流。

---

## Phase 2: SCAMPER - 系统化发散

### 关键发现

**人机协作不是两种模式，是一个确认频谱**

| 方向 | 特征 | 阶段 |
|------|------|------|
| 高频确认（共创） | 每步确认、实时对话、人AI共创 | 规划阶段（头脑风暴→PRD→Architecture→Story拆分） |
| 低频确认（门控） | 关键节点1次确认 | 执行阶段（PR Review、Sprint验收） |

本质都是人确认，区别只是频率。不需要两套引擎，需要的是确认频率的可配置。

**三角架构**

```
         Claude Code (BMad 运行时)
            /              \
           /                \
     HappyCapy            GitHub
     (交互/定时)        (CI/Review/PM)
```

BMad 运行在 Claude Code 中。HappyCapy 和 GitHub 都是通过 Claude Code + BMad 来完成工作的。两个平台各有所长，可以组合配置。

**平台能力分布**

| 能力 | HappyCapy | GitHub |
|------|-----------|--------|
| 共创对话 | 强（实时交互） | 弱 |
| 定时任务 | Automations | Actions |
| CI/测试 | 弱 | 强（Actions） |
| Code Review | 可以 | 原生 PR Review |
| 项目看板 | -- | Projects |

**约束基座需要可插拔**

BMad 是起点不是终点。Harness 的 L1 是一个可插拔的规范框架槽位，BMad 是默认实现。复杂业务场景需要扩展文档类型（流程图、状态流转等）。

**Agent 角色 vs Agent 编队**

BMad 里定义了角色（PM、Architect、Dev、QA）。编队伸缩只发生在 Dev 阶段——多个 Dev Agent 并行处理多个 Story。

**驱动方式：混合驱动**

| 触发方式 | 适用场景 |
|----------|----------|
| 事件驱动 | Story 开发完成→自动推进下一个（GitHub 原生支持） |
| 定时巡检 | 平台限制时的替代方案（HappyCapy Automations） |
| 人类触发 | 用户反馈处理、Sprint 规划、方向性决策 |

---

## Phase 3: Solution Matrix - 优选方案收敛

### 架构设计

```
┌──────────────────────────────────────────────────┐
│             Harness 核心架构                       │
│                                                   │
│  推进引擎 (运行在 Claude Code 中)                  │
│  ├─ BMad 规范流程（可替换为 ECC 等）                │
│  ├─ 推进循环：感知→决策→执行→验证→同步             │
│  ├─ 确认机制：高频(共创) ←→ 低频(门控) 可配置       │
│  └─ 并行涌现：多 Ready Story 自然 fork 多 Dev      │
│                                                   │
│  平台适配层 (可配置组合)                            │
│  ├─ HappyCapy：交互/定时触发/共创对话               │
│  ├─ GitHub：PM/CI/Review/状态同步                  │
│  └─ 其他：预留适配器接口                            │
│                                                   │
│  项目配置 (per-project)                            │
│  ├─ 哪些阶段用哪个平台                             │
│  ├─ 每个步骤的确认频率                              │
│  ├─ 触发方式（事件/定时/手动）                       │
│  └─ 扩展文档类型（流程图等）                        │
└──────────────────────────────────────────────────┘
```

### 项目生命周期

```
Phase 1: 规划 [HappyCapy 上共创]
  人 ←→ Claude Code(BMad) 实时对话
  产出：PRD → Architecture → Epic/Story
  同步到 GitHub：Issues + Project Board + Milestone

Phase 2: 执行 [HappyCapy + GitHub 组合]
  HappyCapy Automation 定时触发：
    → Claude Code(BMad) 检查 GitHub Board
    → 领取 Ready Story
    → 自主开发 → 提交 PR
  GitHub Actions 触发：
    → CI 测试 → Lint 检查
    → 通知人类 Review
  人在 GitHub 上 Review PR → 合并
  → 事件/定时 → 推进下一个 Story

Phase 3: 收敛 [人类触发]
  Sprint 验收 → 复盘 → 下一 Sprint 规划
  → 回到 Phase 1 或 继续 Phase 2
```

### GitHub 同步策略

| 对象 | 用途 |
|------|------|
| Project Board | 项目全景状态看板 |
| Milestones | Sprint/阶段跟踪 |
| Issues | Story/Task/Bug 跟踪 |
| PRs | 代码交付 + Review 门控 |
| Comments | AI 与人类的沟通记录 |
| Actions/Automations | 自动推进触发器 |

### 项目配置示例（概念）

```yaml
# harness.config.yml
project:
  name: "my-project"
  methodology: bmad          # bmad | ecc | custom
  pm_platform: github        # github | ...

phases:
  planning:
    platform: happycapy
    confirmation: per-step   # 每步确认（共创）

  execution:
    trigger: scheduled        # scheduled | event | manual
    interval: 30m
    dev_platform: happycapy
    ci_platform: github
    review: github-pr
    confirmation: per-story  # Story 级确认（门控）

  review:
    trigger: manual
    platform: happycapy
```

---

## Phase 4: 待验证猜想与 Research 清单

方案中有大量细节基于猜想，需要专项 Research 明确：

### 必须先做（MVP 基础）

**R1: BMad 工作流细节**
- BMad 有哪些角色、每个角色具体做什么
- create-prd 的 11 步具体内容及确认点
- BMad 的 Story 生命周期（从创建到完成）
- BMad 支持哪些文档类型，缺什么（如流程图）
- BMad 的质量验证机制（checklist？自动检查？）

**R2: GitHub 能力边界**
- GitHub Projects 的状态流转能力（能否支撑推进引擎的状态感知）
- GitHub Actions 的事件触发粒度（PR 合并后自动推进下一 Story 是否可行）
- Issue/PR 的自动化操作边界（自动创建、状态变更、Label 管理）
- GitHub API 对 Claude Code / Agent 的可用性

**R3: HappyCapy 能力边界**
- Automations 的定时触发能力（最小间隔？能传什么上下文？）
- Session 间能否共享项目状态（多 Agent 并行时）
- HappyCapy 与 GitHub 之间的联动可行性
- Claude Code 在 HappyCapy 中的 BMad skill 安装和运行方式

**R4: 感知→决策机制** ⚠️ 优先级已提升，直接影响 Harness 衔接方式
- BMad 状态对外暴露机制（文件系统？Session 输出？）
- AI Agent 如何读取 GitHub Board 状态并决定下一步的具体实现方式
- "领取 Ready Story"的机制——DoR 由谁编码？AI 自动判断还是人工标记 Ready？
- DoR 归 Harness 管理（推进引擎判断），DoD 归 BMad 内部管理（黑盒输出"Done"信号）

**R5: 执行阶段的实际闭环**
- Claude Code + BMad 执行一个 Story 的实际端到端流程
- dev-story 完成 → 单元测试通过 → PR 创建 → bmad-code-review 触发 → 人工合并的完整链路验证
- AI 写的代码质量保障：BMad 的规范约束是否足够

### 可后做（扩展阶段）

**R6: 多 Agent 并行的可行性**
- 多个 HappyCapy Session 同时操作同一 Repo 是否会冲突
- Git 分支策略：每个 Agent 一个分支？如何避免冲突？
- 共享上下文的具体实现方式

**R7: 配置化的范围**
- harness.config.yml 概念是否可行——谁读它？怎么生效？
- 哪些东西真的需要配置化，哪些首期可以硬编码

---

## Phase 5: Party Mode 团队讨论 - 关键补充与修正

### 参与者

John (PM), Winston (Architect), Amelia (Dev), Bob (Scrum Master)

### 关键结论

**1. Harness 与 BMad 的职责分界（核心共识）**

BMad 管内部流程和角色，Harness 管"BMad 之间"和"BMad 之后"的衔接与推进。Harness 不侵入 BMad 的内部逻辑。

| 职责 | 负责方 | 备注 |
|------|--------|------|
| Story 内部执行流程 | BMad | create-prd, dev-story 等 |
| 角色分配和切换 | BMad | 由 skill 决定角色 |
| Story 内部状态管理 | BMad | Review → Done |
| Done → PR 创建 | Harness | BMad Done 后衔接 |
| 代码 Review | bmad-code-review | PR 触发，自动执行后标记已 Review |
| PR Review 门控 | 人类 (GitHub) | 查看 bmad-code-review 结果后决定合并 |
| PR 合并 → Board 更新 | Harness | 状态同步 |
| 下一个 Story 的调度 | Harness 推进引擎 | Board 感知 + 触发 |
| Sprint 规划 / 复盘 | 人类触发 + BMad 共创 | 高频确认阶段 |

**2. 状态链扩展**

BMad 的状态链末端需要接上 GitHub 的交付链路：

```
BMad 状态链:  ... → Review → Done
                              ↓
Harness 扩展:            创建 PR → bmad-code-review → 人类查看结果合并 → 交付完成
```

**3. 推进引擎简化（Amelia 提出）**

执行阶段保持"愚蠢和可靠"。严格按 Sprint Plan 里的 Story 顺序执行，不做智能调度：

```
Sprint Plan (ordered list)
  → Story 1: Done
  → Story 2: Done
  → Story 3: In Progress...
  → Story 4: Ready (next)
```

引擎只需：读 Sprint Plan → 找到第一个 Ready → 启动。复杂度留给 Sprint Planning。

**4. Harness 本质定义（Winston 总结）**

Harness 本质上是一个 **"BMad Session 生命周期管理器"**：
- 从 GitHub Board 感知哪些 Story 可以开始
- 触发 Claude Code Session 启动 BMad 的 dev-story
- 监听 BMad 的 Done 状态 → 触发 PR 流程
- PR 合并后 → 更新 Board → 感知下一个可执行 Story

**5. 补充的 Research 需求**

- **R4 优先级提升**：BMad 状态对外暴露机制（文件系统？Session 输出？）需要尽早 Research，直接影响衔接方式——已移入"必须先做"清单
- **确认频率：步骤级可配置**（John 提出，已决策）：不只是阶段级，PRD 11 步中有些可合并确认，Story 开发中可能需要中途 check-in；首期硬编码合理默认值，配置化在 1→10 阶段实现
- **DoR 归 Harness，DoD 归 BMad**（Bob 提出，已澄清）：DoD 由 BMad 内部 checklist 管理，Harness 只感知"Done"信号（黑盒）；DoR 由 Harness 编码，推进引擎据此判断 Story 是否可以启动
- **首期硬编码**（Winston 建议，已决策）：先跑通完整闭环，配置化是 1→10 的事

---

## MVP Scope 定义

**目标：** 一个 Story 的完整自动推进闭环，硬编码，单项目单 Sprint。

**前提条件（人工完成）：** Sprint Planning 已完成，GitHub Board 上至少有一个 Story 标记为 Ready。

**自动化流程：**
```
HappyCapy Automation 定时触发
  → Claude Code 读取 GitHub Board，识别第一个 Ready Story
  → 启动 BMad dev-story 执行
  → BMad Done 信号 → Harness 自动创建 PR（未 Review 状态）
  → PR 触发 bmad-code-review → 执行代码审查 → 标记为已 Review
  → 人类查看 Review 结果 → 决定合并
  → 合并后 Harness 更新 Board 状态 → 识别下一个 Ready Story
```

**人类只做两件事：** Sprint Planning + 查看 bmad-code-review 结果并决定合并。

**不在 MVP 内：** 多 Story 并行、配置化、自动 Sprint Planning、复盘、错误恢复机制。

**成功标准：** 一个 Story 从 Board "Ready" 到 PR 合并，人工干预仅限以上两点。

---

## 下一步行动

1. **R4 Research**（最高优先级）：明确 BMad 状态对外暴露机制，以及 DoR 的具体判断实现方式
2. **R1/R2/R3 Research**：并行启动，覆盖 BMad 工作流、GitHub 能力边界、HappyCapy 能力边界
3. **R5 Research**：验证 dev-story → 单元测试 → PR → bmad-code-review → 合并的完整链路可行性
4. 基于 Research 结果修正方案细节，输出可实施的技术设计
5. 按 MVP Scope 实施验证，硬编码跑通第一个端到端闭环
