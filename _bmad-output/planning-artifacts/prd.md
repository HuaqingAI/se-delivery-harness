---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-02b-vision
  - step-02c-executive-summary
inputDocuments:
  - _bmad-output/planning-artifacts/research/technical-bmad-harness-core-mechanism-research-2026-04-08.md
  - _bmad-output/planning-artifacts/research/technical-github-happycapy-capability-research-2026-04-08.md
  - _bmad-output/planning-artifacts/research/technical-R5-R6-R7-research-2026-04-08.md
  - _bmad-output/brainstorming/brainstorming-session-2026-04-07-1000.md
workflowType: 'prd'
briefCount: 0
researchCount: 3
brainstormingCount: 1
projectDocsCount: 0
classification:
  projectType: 'orchestration-platform'
  projectTypeDetail: '平台无关的 AI Agent 编排套件（核心编排引擎 + 可插拔平台适配层）'
  domain: 'software-engineering-automation'
  domainDetail: 'AI Agent Orchestration for Software Delivery'
  complexity: 'medium-high'
  projectContext: 'greenfield'
  interactionMode: 'autonomous-with-human-checkpoints'
  runtimeTopology: 'distributed'
  partyModeInsights:
    - '项目类型从 Developer Tool 精化为 Orchestration Platform'
    - 'GitHub App 定位为身份+环境初始化+权限管理，非驱动引擎；驱动统一用 Actions'
    - '核心引擎通过 PlatformAdapter 接口解耦，每平台可独立或组合跑通完整流程'
    - 'MVP 路径：PAT + gh CLI + HappyCapy Automation；GitHub App 为 Phase 2 渐进升级'
    - '接口抽象需架构阶段用双平台场景走查验证'
---

# Product Requirements Document - se-delivery-harness

**Author:** Sue
**Date:** 2026-04-08

## Executive Summary

SE Delivery Harness 是一个平台无关的编排套件，使 AI Agent 能够自主驱动 Sprint 执行——从感知就绪的 Story 到交付经过验证的、生产级质量的 Pull Request。它将经过验证的软件工程实践（DoR/DoD、检查清单、质量门禁）编码为机器可执行的约束，然后编排"感知→决策→执行→监控→完成"循环，将 AI 从被动的代码补全工具转变为能够持续、独立交付的可靠初级工程师。

系统面向使用 BMad Method 进行规划的软件工程团队和独立开发者，旨在弥合"AI 能写代码"与"AI 能交付 Story"之间的鸿沟。主要运行环境为 Claude Code 会话（通过 HappyCapy Automation 或 GitHub Actions 驱动），并提供可插拔的 PlatformAdapter 接口，允许任意 PM 平台独立运行完整交付周期，或与其他平台协作分工。

### What Makes This Special

现有 AI 编码工具（Copilot、Cursor）在代码行/函数级别提供辅助。现有自动化工具（CI/CD、CodeRabbit）处理孤立的单一环节。SE Delivery Harness 是首个编排完整交付生命周期的系统——拾取就绪 Story、实现代码、运行测试、创建 PR、对照验收标准验证、更新项目状态——作为一个自主闭环运转。核心洞察：BMad v6.2.2 已将人类工程经验编码为结构化、可验证的约束；Claude Code 现已具备足够的工具使用能力来执行这些约束；持久化运行环境（HappyCapy、GitHub Actions）提供了跨会话的连续性。这三个前提条件首次汇聚，使自主 Sprint 执行在今天成为可能。

## Project Classification

- **项目类型：** 编排平台——核心引擎 + 可插拔平台适配层
- **领域：** 软件工程自动化 / AI Agent 编排
- **复杂度：** 中高——多平台集成、适配器架构、跨会话状态管理
- **项目上下文：** 绿地项目（Greenfield）
- **交互模式：** 自主运行 + 人类检查点（PR 审查、质量门禁升级）
- **MVP 路径：** PAT + gh CLI + HappyCapy Automation；GitHub App（仅身份/权限管理）推迟至 Phase 2
- **架构原则：** 核心引擎不直接导入任何平台 SDK；所有平台交互通过 PlatformAdapter 接口；每个适配器可独立运行完整的"感知→决策→执行→监控→完成"周期
