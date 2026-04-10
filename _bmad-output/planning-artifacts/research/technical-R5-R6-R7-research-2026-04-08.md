---
stepsCompleted: [1, 2, 3, 4, 5, 6]
inputDocuments: []
workflowType: 'research'
lastStep: 1
research_type: 'technical'
research_topic: 'R5 R6 R7 - SE Delivery Harness 关键技术验证'
research_goals: '验证 Claude Code + BMad Story 完整交付链路可行性（R5）；评估多 Agent 并行操作同一 Repo 的可行性与冲突策略（R6）；明确配置化范围与硬编码边界（R7）'
user_name: 'Sue'
date: '2026-04-08'
web_research_enabled: true
source_verification: true
---

# SE Delivery Harness 关键技术验证：R5 R6 R7 综合技术研究报告

**Date:** 2026-04-08
**Author:** Sue
**Research Type:** Technical Research
**Status:** Complete

---

## Executive Summary

本报告对 SE Delivery Harness 的三项关键技术验证进行了全面调研：**R5（Claude Code + BMad Story 完整交付链路可行性）**、**R6（多 Agent 并行操作同一 Repo 的可行性与冲突策略）**、**R7（配置化范围与硬编码边界）**。基于对 Claude Code 官方文档、Anthropic 研究论文、GitHub 文档、BMad Method 文档及 30+ 来源的系统化分析，本报告确认三项技术验证的可行性，并提出具体实现路径。

**核心结论：三项技术验证均为"可行"，但需分阶段渐进实施。**

- **R5 可行性：高置信度**。Claude Code CLI + BMad Method 已具备完整的 Story 交付链路所需的所有原语（worktree 隔离、headless 执行、PR 自动关联、GitHub Actions 集成、对抗性代码审查）。关键约束：Claude 不会自动创建 PR——这是 Anthropic 强制的人工监督检查点。推荐首期使用 Prompt Chaining 模式。
- **R6 可行性：中高置信度**。Claude Code 是目前唯一具有文档化多 Agent 同 Repo 支持的 AI 编码工具（Agent Teams + Worktree + 子代理隔离 + 云 VM）。核心原则"按文件所有权分配任务"已被验证有效。但 Agent Teams 仍为实验性功能（v2.1.32+，默认禁用），存在已知限制（无会话恢复、不支持嵌套团队）。推荐首期使用 Worktree + 子代理，Agent Teams 作为扩展选项。
- **R7 可行性：高置信度**。行业已收敛到"Markdown + YAML frontmatter"配置模式（30+ 工具采用 Agent Skills 开放标准）。Claude Code 四层配置架构（Managed > User > Project > Local）+ 行为引导（CLAUDE.md）与技术强制（settings.json）的清晰分离为 Harness 配置设计提供了成熟范式。推荐首期 80%+ 场景硬编码，采用渐进式披露架构。

**Key Technical Findings:**
- Claude Code 日均成本 ~$6/开发者（Sonnet 4.6），模型分层可实现 5 倍成本差
- AI 辅助下任务完成速度提升 55%（GitHub Research, N=95, P=0.0017）
- 26 种 Hook 事件 + 4 种 Handler 类型覆盖完整会话生命周期
- 三阶段采纳路径：辅助 → 半自动 → 全自动，每阶段独立可验证

---

## Table of Contents

1. [Technical Research Scope Confirmation](#technical-research-scope-confirmation)
2. [Technology Stack Analysis](#technology-stack-analysis)
   - R5 核心技术栈：Story 完整交付链路
   - R6 核心技术栈：多 Agent 并行
   - R7 核心技术栈：配置化模式
3. [Integration Patterns Analysis](#integration-patterns-analysis)
   - R5 集成链路：Session → Git → PR → Review
   - R6 集成链路：多 Agent 协调协议
   - R7 集成链路：配置流与传播
4. [Architectural Patterns and Design](#architectural-patterns-and-design)
   - R5 架构决策：交付链路架构
   - R6 架构决策：多 Agent 协调架构
   - R7 架构决策：配置化架构
5. [Implementation Approaches and Technology Adoption](#implementation-approaches-and-technology-adoption)
   - 渐进式采纳策略
   - 测试策略、成本管理、安全风险评估
   - 实现路线图与技术栈推荐
6. [Research Synthesis and Conclusion](#research-synthesis-and-conclusion)
   - 可行性验证结论
   - 风险评估与缓解
   - 下一步行动建议

---

## Research Overview

本报告采用网络调研 + 多源交叉验证方法论，覆盖 Claude Code 官方文档（code.claude.com/docs）、Claude Agent SDK 文档（platform.claude.com/docs）、Anthropic 研究论文（anthropic.com/research）、GitHub 文档（docs.github.com）、BMad Method 文档（docs.bmad-method.org）、Agent Skills 开放标准（agentskills.io）以及 Git 官方文档（git-scm.com/docs）等权威来源。所有技术声明均附 URL 引用。调研分五个阶段系统展开：技术栈分析、集成模式分析、架构模式分析、实现研究，最终综合为本报告。详细执行摘要见上文。

## Technical Research Scope Confirmation

**Research Topic:** R5 R6 R7 - SE Delivery Harness 关键技术验证
**Research Goals:** 验证 Claude Code + BMad Story 完整交付链路可行性（R5）；评估多 Agent 并行操作同一 Repo 的可行性与冲突策略（R6）；明确配置化范围与硬编码边界（R7）

**Technical Research Scope:**

- Architecture Analysis - Claude Code 工具链、BMad skill 执行机制、HappyCapy Session 架构
- Implementation Approaches - Story 执行链路、PR 自动化、多 Agent 协作模式
- Technology Stack - Claude Code CLI、BMad、GitHub Actions/API、Git 分支策略
- Integration Patterns - Session → Git → GitHub PR → Review 的衔接方式
- Performance Considerations - 链路稳定性、并行冲突风险、配置化复杂度评估

**Research Methodology:**

- Current web data with rigorous source verification
- Multi-source validation for critical technical claims
- Confidence level framework for uncertain information
- Comprehensive technical coverage with architecture-specific insights

**Scope Confirmed:** 2026-04-08

## Technology Stack Analysis

### R5 核心技术栈：Story 完整交付链路

#### Claude Code CLI 能力

Claude Code 是 Anthropic 的 agentic coding CLI，支持终端、VS Code/JetBrains 扩展、桌面应用、Web 和移动端。核心能力：

_Git 与 PR 管理_
- 原生创建分支、暂存变更、编写提交信息、打开 PR
- `--worktree` (`-w`) 标志创建隔离的 git worktree 用于并行会话
- 会话通过 `gh pr create` 自动关联 PR，支持 `claude --from-pr <number>` 恢复

_测试执行与修复_
- 单条命令运行测试并修复失败：`claude "write tests for the auth module, run them, and fix any failures"`
- Headless 模式：`claude -p "Run the test suite and fix any failures" --allowedTools "Bash,Read,Edit"`

_Headless/程序化模式 (`-p` 标志)_
- `--bare` 跳过自动发现，适合 CI 脚本调用
- `--output-format text|json|stream-json` 控制输出格式
- `--allowedTools` 预批准工具避免权限提示
- `--permission-mode acceptEdits|dontAsk|bypassPermissions` 设定权限级别
- `--max-turns N` 限制代理轮次
- `--permission-prompt-tool` 委托权限决策给 MCP 工具，实现完全非交互运行

_GitHub Actions 集成_
- `anthropics/claude-code-action@v1` 支持 PR 评论中 `@claude` 触发
- 可配置自动 PR 创建、代码审查、定时自动化
- 支持 AWS Bedrock 和 Google Vertex AI 企业环境

_自动代码审查（研究预览）_
- 在 PR 上发布内联审查评论，按严重性分类：Important（红）、Nit（黄）、Pre-existing（灰）
- 使用并行专业代理群分析 diff
- 可通过 `CLAUDE.md` 和 `REVIEW.md` 自定义规则

_Source: https://code.claude.com/docs/en/overview, https://code.claude.com/docs/en/headless, https://code.claude.com/docs/en/github-actions, https://code.claude.com/docs/en/code-review_

#### Claude Agent SDK

CLI 同引擎的 Python (`claude-agent-sdk`) 和 TypeScript (`@anthropic-ai/claude-agent-sdk`) SDK：
- `query()` 异步生成器流式输出消息和工具结果
- 支持 Hooks（PreToolUse、PostToolUse、Stop、SessionStart 等）
- 子代理（Subagent）机制：独立上下文窗口 + 受限工具集
- 会话持久化：通过 `session_id` 恢复

_Source: https://platform.claude.com/docs/en/agent-sdk/overview_

#### BMad Method (v6.2.2)

BMad 是开源 AI 代理框架，安装到项目 `.claude/skills/` 目录，所有 skill 作为 `/command` 可用。

_核心 Skill 链路_
| Skill | 功能 | 阶段 |
|---|---|---|
| `bmad-sprint-planning` | 初始化 sprint-status.yaml | 规划 |
| `bmad-create-story` | 从 epic 生成 story 文件 | 准备 |
| `bmad-dev-story` | 实现 story（需新会话） | 执行 |
| `bmad-code-review` | 对抗性代码审查（需新会话） | 质量 |
| `bmad-retrospective` | Epic 完成后复盘 | 总结 |

_关键设计约束_
- 每个 story 必须在**独立新会话**中执行（避免上下文污染）
- `bmad-code-review` 采用对抗性方法——不允许 "looks good"
- `bmad-check-implementation-readiness` 门控检查产出 PASS/CONCERNS/FAIL
- `bmad-help` 自动分析项目状态并推荐下一步

_Source: https://github.com/bmadcode/bmad-method, https://docs.bmad-method.org_

### R6 核心技术栈：多 Agent 并行

#### Claude Code 并行工作机制

Claude Code 是目前唯一具有显式、文档化、生产级多 Agent 同 Repo 支持的 AI 编码工具。提供四种并行机制：

_A. Git Worktree 隔离 (`--worktree`)_
- 每次调用在 `<repo>/.claude/worktrees/<name>/` 创建隔离工作目录
- 独立分支 `worktree-<name>`，基于 `origin/HEAD`
- 共享 git 对象数据库，独立 HEAD、索引和工作文件
- `.worktreeinclude` 自动复制 gitignored 文件（如 `.env`）

_B. Agent Teams（实验性）_
- 多个 Claude Code 实例共享任务列表 + 代理间消息 + Team Lead
- 文件锁定防止竞争条件
- `TeammateIdle`、`TaskCreated`、`TaskCompleted` hooks 可执行质量门控
- 推荐团队规模 3-5 个 agent，每个 5-6 个任务
- **关键约束：避免两个 agent 编辑同一文件——按文件所有权分配任务**

_C. 子代理 Worktree 隔离_
- 子代理可设置 `isolation: worktree`，获得独立 worktree
- 无变更时自动清理

_D. 云端 VM 隔离 (`--remote`)_
- `claude --remote "task"` 在独立云 VM 启动
- 最强隔离——完全独立文件系统和 clone

_Source: https://code.claude.com/docs/en/agent-teams, https://code.claude.com/docs/en/common-workflows_

#### Git Worktree 技术细节

- 多个链接 worktree 共享同一 `.git/` 对象存储，独立 HEAD/索引
- **分支唯一性约束**：已在一个 worktree 签出的分支不能在另一个中签出
- 子模块支持不完整——多 worktree 含子模块**不推荐**
- worktree 轻量——共享对象数据库，无 clone 开销
- 但**每个 worktree 需独立安装依赖**（如 `npm install`）
- `origin/HEAD` 同步问题：需 `git remote set-head origin -a` 修复

_Source: https://git-scm.com/docs/git-worktree_

#### 其他 AI 工具的并行策略对比

| 工具 | 并行模式 | 冲突策略 |
|---|---|---|
| Claude Code | Worktree + Agent Teams + 云 VM | 文件所有权 + 任务锁定 |
| GitHub Copilot | 每任务一分支一 PR（串行） | 强制分支隔离 |
| Devin | Docker 容器隔离 | 独立环境 |
| 通用模式 | 分支-per-agent + PR 集成 | GitHub Merge Queue |

#### GitHub Merge Queue

多 Agent PR 场景的推荐基础设施：
- FIFO 排序 + 临时验证分支
- 每个 PR 基于最新 base + 所有前序队列 PR 测试
- 构建并发度可配置 1-100
- 失败 PR 自动移除，剩余独立重新验证
- 支持优先级插队

_Source: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue_

### R7 核心技术栈：配置化模式

#### AI 工具配置模式对比

| 工具 | 项目指令 | 个人设置 | 组织管理 | 技能 |
|---|---|---|---|---|
| Claude Code | CLAUDE.md | ~/.claude/ | managed-settings.json | .claude/skills/SKILL.md |
| Copilot | .github/copilot-instructions.md | 个人设置 | 组织策略 | .github/skills/ |
| Gemini CLI | GEMINI.md | ~/.gemini/ | - | SKILL.md |
| Cursor | .cursor/rules | - | - | SKILL.md |
| Kiro | .kiro/steering/*.md | ~/.kiro/ | - | Specs + Hooks |

#### Claude Code 四层配置架构

| 层级 | 位置 | 共享？ |
|---|---|---|
| Managed（组织） | 系统级 managed-settings.json | 是——IT/MDM 部署 |
| User（个人） | `~/.claude/` | 否——个人所有项目 |
| Project（项目） | `.claude/settings.json` | 是——提交到 git |
| Local（本地） | `.claude/settings.local.json` | 否——gitignored |

- CLAUDE.md 是行为引导（prose），settings.json 是技术强制（enforcement）
- CLAUDE.md 按目录树向上遍历，所有发现的文件拼接而非覆盖
- `.claude/rules/` 支持路径范围规则文件（YAML frontmatter 指定 `paths:`）
- CLAUDE.md 超过 200 行会降低遵循度和消耗过多上下文

_Source: https://code.claude.com/docs/en/settings, https://code.claude.com/docs/en/memory_

#### Agent Skills 开放标准

2025-2026 年最重要的发展——30+ AI 工具采用的跨工具配置格式（包括 Claude Code、Copilot、Gemini CLI、Cursor、OpenHands 等）。

_SKILL.md 结构_
```
skill-name/
  SKILL.md        # 必需：YAML frontmatter + Markdown 指令
  scripts/        # 可选：可执行代码
  references/     # 可选：参考文档
  assets/         # 可选：模板资源
```

_渐进式披露设计_
1. **发现层**（~100 tokens）：`name` + `description` 启动时加载
2. **指令层**（<5000 tokens）：激活时加载完整 SKILL.md
3. **资源层**（按需）：仅在需要时加载 scripts/references/assets

_Source: https://agentskills.io, https://agentskills.io/specification_

#### 约定优于配置原则

_何时硬编码（使用约定）_
- 行为在所有使用场景中普遍正确
- 配置会造成团队间兼容性碎片化
- 约定能级联启用其他约定（Rails 表命名）
- 80%+ 用例零配置即可工作

_何时配置化_
- 行为在不同项目间合理变化
- 团队有超出默认值的特定强制要求
- 安全策略需组织级强制（Claude Code managed settings）
- 需要路径级别的差异化

_何时渐进式披露_
- 面向不同复杂度需求的广泛受众
- 零配置覆盖 80%+，配置可发现但非必需
- SKILL.md 三层加载是 2025 年的标杆范例

_Source: https://rubyonrails.org/doctrine, https://biomejs.dev/guides/getting-started/, https://prettier.io/docs/configuration_

### Technology Adoption Trends

_行业趋势_
- **Markdown + YAML frontmatter** 已成为 AI 工具配置的主流格式（而非纯 YAML 或纯 JSON）
- **配置分层**是普遍模式：managed > user > project > local
- YAML 适合声明式自动化（GitHub Actions、Taskfile），但无循环/条件/测试能力
- "模板化 YAML" 被视为反模式——缩进错误导致静默 bug
- **12-Factor 原则**适用于 AI 代理：凭证和环境变量严格分离，行为指令放 Markdown 文件

_关键区分_
- **行为引导**（Markdown 文件，LLM 消费）vs **技术强制**（JSON settings，工具运行时消费）
- Claude Code 文档最明确阐述了这一点："Settings rules are enforced by the client. CLAUDE.md instructions shape behavior but are not a hard enforcement layer."

_Source: https://12factor.net/config, https://www.pulumi.com/blog/pulumi-yaml/_

## Integration Patterns Analysis

### R5 集成链路：Session → Git → PR → Review

#### A. Story 交付完整链路

```
[1] /bmad-create-story → 生成 story 文件
         ↓
[2] claude -w story-XXX → 创建 worktree 隔离环境
         ↓
[3] /bmad-dev-story → 在隔离 worktree 中实现 story
         ↓ (context: fork 子代理)
[4] git commit + gh pr create → 自动关联 PR
         ↓
[5] /bmad-code-review → 对抗性审查（新会话）
         ↓
[6] GitHub Merge Queue → CI 验证 + 自动合并
```

_关键集成点_
- 步骤 [2]→[3]：`--worktree` 创建 `.claude/worktrees/<name>/`，分支 `worktree-<name>` 基于 `origin/HEAD`
- 步骤 [3]→[4]：`gh pr create` 执行时自动关联会话到 PR，后续可用 `claude --from-pr <number>` 恢复
- 步骤 [4]→[5]：代码审查必须在**新会话**中运行（`context: fork`），确保审查者无实现者偏见
- `.worktreeinclude` 文件控制哪些 gitignored 文件复制到 worktree（如 `.env`）

_Source: https://code.claude.com/docs/en/common-workflows, https://code.claude.com/docs/en/skills_

#### B. Headless 模式 CI 集成

```bash
# 最小 CI 设定
export ANTHROPIC_API_KEY="sk-ant-..."
claude -p "implement story-42" \
  --bare \                          # 跳过所有自动发现
  --allowedTools "Read,Edit,Write,Bash,Glob,Grep" \
  --max-turns 50 \
  --output-format json
```

- `--bare` 模式跳过：Hooks、Skills、Plugins、MCP、CLAUDE.md 自动发现
- 认证必须通过 `ANTHROPIC_API_KEY`（跳过 OAuth/keychain）
- 显式加载配置：`--append-system-prompt-file`、`--settings`、`--mcp-config`
- `--bare` 将在未来版本成为 `-p` 的默认行为

_Source: https://code.claude.com/docs/en/headless_

#### C. GitHub Actions 自动化

```yaml
# PR 评论触发 Claude
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "/bmad-code-review"    # 可直接调用 BMad skill
    claude_args: "--max-turns 10"
    trigger_phrase: "@claude"       # PR 评论中 @claude 触发
```

- `prompt` 字段接受 skill 名称——repo 中 `.claude/skills/` 的任何 skill 可直接调用
- v1 破坏性变更：`mode: "tag"/"agent"` 移除（自动检测），`direct_prompt` → `prompt`
- 支持 AWS Bedrock (`use_bedrock`) 和 Google Vertex AI (`use_vertex`)

_Source: https://code.claude.com/docs/en/github-actions_

#### D. Claude Agent SDK 编排

```python
# Python SDK：Story 交付编排
async for message in query(
    prompt="/bmad-dev-story story-42",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Write", "Bash", "Glob", "Grep", "Agent"],
        hooks={
            "PreToolUse": [HookMatcher(matcher="Write|Edit", hooks=[protect_env_files])],
            "Stop": [HookMatcher(hooks=[verify_tests_pass])]
        },
        agents={
            "code-reviewer": AgentDefinition(
                description="Expert code reviewer",
                prompt="You are a code review specialist...",
                tools=["Read", "Grep", "Glob"],
                model="opus"
            )
        }
    ),
):
    if isinstance(message, ResultMessage):
        session_id = message.session_id  # 可持久化用于恢复
```

_SDK 关键能力_
- `resume=session_id`：恢复中断的会话（完整上下文）
- `fork_session=True`：从某一点分叉探索替代方案
- `AgentDefinition`：自定义子代理（受限工具集 + 独立模型）
- 26 种 Hook 事件：覆盖完整会话生命周期
- `ClaudeSDKClient` 多轮自动管理：同一实例内 `query()` 自动续接

_Source: https://platform.claude.com/docs/en/agent-sdk/overview, https://platform.claude.com/docs/en/agent-sdk/hooks_

#### E. BMad Skill 链接机制

- Skill **不直接链接**——由协调代理或主会话顺序调用
- `context: fork` 创建隔离子代理：只接收 SKILL.md 内容 + CLAUDE.md，不继承父对话历史
- `$ARGUMENTS` 替换：`/bmad-dev-story 42` → `$ARGUMENTS` 变为 `42`
- 动态上下文注入（`` !`command` `` 语法在发送给 Claude 前执行 shell 命令）：
  ```markdown
  ## Story context
  - Current sprint: !`cat sprint-status.yaml | yq .current_sprint`
  - Story file: !`cat stories/$ARGUMENTS.md`
  ```

_Source: https://code.claude.com/docs/en/skills_

### R6 集成链路：多 Agent 协调协议

#### A. Agent Teams 协调架构

| 组件 | 职责 |
|---|---|
| Team Lead | 创建团队、分配任务、审批计划、合成结果 |
| Teammates | 独立 Claude Code 实例，各有独立上下文窗口 |
| Task List | 共享任务列表（pending → in_progress → completed），支持依赖声明 |
| Mailbox | 代理间消息系统（定向 + 广播） |

_存储位置_
- 团队配置：`~/.claude/teams/{team-name}/config.json`
- 任务列表：`~/.claude/tasks/{team-name}/`

_协调规则_
- 任务认领使用**文件锁定**防止竞争条件
- 依赖任务在前置任务完成前不可被认领（自动解锁）
- Team Lead 是固定的（创建会话即为 Lead，不可转让）
- Teammate 完成轮次时自动通知 Lead（`TeammateIdle` 事件）
- 推荐 3-5 个 Teammate，每个 5-6 个任务
- Token 成本：约为标准会话的 7 倍

_Source: https://code.claude.com/docs/en/agent-teams_

#### B. Hook 质量门控

```json
{
  "hooks": {
    "TaskCompleted": [{
      "hooks": [{
        "type": "command",
        "command": "npm test -- --testPathPattern=\"$(echo $TASK_SUBJECT | tr ' ' '-')\"",
        "timeout": 60
      }]
    }],
    "TaskCreated": [{
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/validate-file-domain.sh"
      }]
    }],
    "TeammateIdle": [{
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/check-incomplete-work.sh"
      }]
    }]
  }
}
```

_Hook 退出码语义_
- `0`：成功，继续执行
- `2`：阻断——TaskCompleted 退 2 = 强制 agent 继续修复；TaskCreated 退 2 = 拒绝创建任务
- 其他：非阻断错误，继续执行

_Source: https://code.claude.com/docs/en/hooks-guide_

#### C. Git Merge Queue 多 Agent 工作流

```yaml
# GitHub Actions 必须包含 merge_group 触发器
on:
  pull_request:
  merge_group:  # Merge Queue 创建的临时验证分支
```

_多 Agent PR 合并流程_
1. 每个 Agent 在独立 worktree 中工作，完成后创建独立 PR
2. 所有 PR 进入 Merge Queue
3. Queue 创建临时分支：当前 base + 所有排队 PR 的合并态
4. CI 在合并态上运行（`merge_group` 事件）
5. 全部通过 → FIFO 顺序合并；失败 → 移除失败 PR，重建队列

_冲突处理回退_
- Agent PR 被 Queue 弹出时：在新 worktree（基于当前 `origin/HEAD`）重新实现冲突部分
- `git remote set-head origin -a` 同步默认分支引用

_Source: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue_

#### D. 文件所有权防冲突五层方案

1. **任务分解时划定文件域**：Lead 不分配重叠文件域给并行任务
2. **TaskCreated Hook 验证**：检查活跃任务注册表，拒绝重复文件域
3. **Worktree 隔离**：文件冲突在 PR Merge 时而非编辑时暴露
4. **子代理 `isolation: worktree`**：子代理级别的文件系统隔离
5. **目录级 CLAUDE.md**：在 `packages/auth/CLAUDE.md` 中声明所有者

_与其他框架对比_

| 维度 | Claude Code Agent Teams | CrewAI | LangGraph | AutoGen |
|---|---|---|---|---|
| 文件锁定 | 内建（任务认领） | 无 | 无 | 无 |
| Git 集成 | 原生（Worktree + 分支） | 无 | 无 | 无 |
| 质量 Hooks | TaskCreated/Completed/TeammateIdle | 无 | 无 | 无 |
| 计划审批 | 内建（Plan Mode 门控） | 无 | 无 | 无 |

_Source: https://code.claude.com/docs/en/agent-teams, https://docs.crewai.com/concepts/crews_

### R7 集成链路：配置流与传播

#### A. 配置优先级（从高到低）

| 优先级 | 层级 | 覆盖关系 |
|---|---|---|
| 1 | Managed Settings（组织策略） | 不可被任何层覆盖 |
| 2 | CLI 参数 | 覆盖所有文件配置 |
| 3 | Local Settings（`.claude/settings.local.json`） | 覆盖 Project/User |
| 4 | Project Settings（`.claude/settings.json`） | 覆盖 User |
| 5 | User Settings（`~/.claude/settings.json`） | 基础层 |

_合并规则_：数组拼接去重，标量后者覆盖前者，对象深度合并。`managed-settings.d/` 支持按字母序片段合并（`10-telemetry.json`, `20-security.json`）。

_Source: https://code.claude.com/docs/en/settings_

#### B. CLAUDE.md 发现与加载

- **目录树向上遍历**：从 CWD 向上，发现所有 `CLAUDE.md` 和 `CLAUDE.local.md`
- **拼接而非覆盖**：所有发现的文件拼接到上下文
- **子目录惰性加载**：子目录下的 CLAUDE.md 仅在 Claude 读取该目录文件时注入
- **导入语法**：`@path/to/import`，最大递归深度 5 层
- **HTML 注释**（`<!-- ... -->`）在注入前被移除
- **InstructionsLoaded Hook**：每次 CLAUDE.md 或 rules 文件加载时触发（可审计）

_Source: https://code.claude.com/docs/en/memory_

#### C. Hook 系统完整事件表（26 种）

| 事件 | 触发时机 | 可阻断？ |
|---|---|---|
| SessionStart | 会话开始/恢复 | 否 |
| UserPromptSubmit | 用户提交提示 | 是 |
| PreToolUse | 工具执行前 | 是 |
| PostToolUse | 工具成功后 | 否 |
| PostToolUseFailure | 工具失败后 | 否 |
| PermissionRequest | 权限对话框出现 | 是 |
| PermissionDenied | Auto 模式分类器拒绝 | 否 |
| Notification | 通知消息 | 否 |
| SubagentStart/Stop | 子代理生命周期 | 否 |
| TaskCreated | 任务创建 | 是（退出码 2） |
| TaskCompleted | 任务完成 | 是（退出码 2） |
| TeammateIdle | Teammate 即将空闲 | 是 |
| Stop/StopFailure | 响应结束 | 否 |
| InstructionsLoaded | 指令文件加载 | 否 |
| ConfigChange | 配置变更 | 否 |
| FileChanged | 监视文件变更 | 否 |
| WorktreeCreate/Remove | Worktree 生命周期 | 否 |
| PreCompact/PostCompact | 上下文压缩 | 否 |
| SessionEnd | 会话终止 | 否 |

_四种 Handler 类型_：`command`（shell 脚本）、`http`（POST 请求）、`prompt`（单轮 LLM 评估）、`agent`（多轮验证，最多 50 轮）

_Source: https://code.claude.com/docs/en/hooks-guide_

#### D. Skill 发现与加载机制

_发现优先级_
1. Enterprise（managed settings）
2. Personal（`~/.claude/skills/`）
3. Project（`.claude/skills/`）
4. Plugin（命名空间 `plugin-name:skill-name`）

_上下文预算_
- 所有 skill 名称始终在上下文中
- description 截断至 250 字符/条目
- 总预算：上下文窗口的 1%（默认回退 8,000 字符）
- 完整 SKILL.md 仅在调用时加载

_调用控制矩阵_

| frontmatter | 用户调用 | Claude 调用 | 上下文存在 |
|---|---|---|---|
| 默认 | 是 | 是 | description 始终；调用时加载全部 |
| `disable-model-invocation: true` | 是 | 否 | description 不在上下文中 |
| `user-invocable: false` | 否 | 是 | description 始终 |

_Source: https://code.claude.com/docs/en/skills_

#### E. Worktree 配置继承

- **Auto Memory 共享**：同一 git repo 的所有 worktree 共享 `~/.claude/projects/<project>/memory/`
- **CLAUDE.local.md 隔离**：每个 worktree 独立，不跨 worktree 共享
- **跨 worktree 共享个人指令**：使用 `@~/.claude/my-project-instructions.md` 导入
- **WorktreeCreate Hook**：可完全替代默认 `git worktree` 逻辑
- **配置发现基于 CWD**：worktree 的 `.claude/` 加载取决于 worktree 内的工作目录

_Source: https://code.claude.com/docs/en/memory, https://code.claude.com/docs/en/sub-agents_

#### F. Permission 规则语法

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)", "Bash(git commit *)", "Skill(bmad-dev-story *)"],
    "deny": ["Bash(curl *)", "Read(./.env)", "Read(./.env.*)"]
  }
}
```

- **优先级**：deny > ask > allow（deny 总是赢）
- **PreToolUse Hook** 在 permission-mode 检查之前触发——Hook `deny` 即使在 `bypassPermissions` 下也生效
- **但 Hook `allow` 不能绕过** settings deny 规则——Hook 只能收紧限制
- **MCP 工具规则**：`mcp__<server>__<tool>` 模式，支持通配符

_Source: https://code.claude.com/docs/en/permissions_

## Architectural Patterns and Design

### R5 架构决策：交付链路架构

#### A. Anthropic 推荐的代理架构模式层级

| 模式 | 适用场景 | 复杂度 | SE Harness 适用性 |
|---|---|---|---|
| **增强 LLM** | 基础层：LLM + 工具 + 记忆 | 最低 | 单步操作基础 |
| **Prompt Chaining** | 固定可分解任务 | 低 | Story → Review 固定链路 |
| **Routing** | 输入分类→专家路由 | 低 | 按 story 类型路由 |
| **并行化** | 独立子任务 / 投票 | 中 | 多 Agent 并行 story |
| **Orchestrator-Workers** | 不可预测子任务结构 | 中高 | Sprint 级别协调 |
| **Evaluator-Optimizer** | 迭代优化（有明确标准） | 中高 | 代码审查反馈循环 |
| **完全代理** | 开放式不可预测步骤 | 最高 | 复杂 bug 修复 |

**核心决策原则**："Start simple. Only add complexity when it demonstrably improves outcomes."

_Source: https://www.anthropic.com/research/building-effective-agents_

#### B. Claude Code Agentic Loop 架构

每个 Claude Code 任务循环三阶段：**收集上下文 → 执行动作 → 验证结果**，迭代直到完成。

_核心架构组件_
- **Agentic Harness**：Claude Code 是模型的执行外壳——提供工具、上下文管理、执行环境
- **上下文窗口是核心资源**：所有对话、文件读取、命令输出、CLAUDE.md、Skills 共享一个上下文窗口
- **会话隔离**：`~/.claude/projects/<encoded-cwd>/*.jsonl`，会话间零上下文泄露
- **检查点系统**：每次文件编辑创建快照，支持回退（但外部副作用无法检查点化）

_推荐自动化工作流_
1. Plan Mode（只读）：Claude 探索代码库，提问题
2. Claude 提出实现计划
3. Normal Mode：Claude 按计划实现 + 测试验证
4. Claude 提交并开 PR

_Source: https://code.claude.com/docs/en/how-claude-code-works, https://code.claude.com/docs/en/best-practices_

#### C. Story-to-PR 管线架构

```
Story/Issue
    │
    ▼
[1. Readiness Check] ← bmad-check-implementation-readiness
    • 子代理验证 story 有 AC、无阻塞
    • Hook: PreToolUse 在检查通过前阻止写入
    │
    ▼
[2. Exploration Phase] ← Plan Mode / Read-only
    • Agent 读取代码库，识别影响范围
    • Agent 提出实现计划
    • 人工或自动化门控审批计划
    │
    ▼
[3. Implementation Phase] ← Orchestrator-Workers
    • 编排者分解子任务
    • Workers 并行实现（Agent Teams 如果独立）
    • 每个 Worker 隔离到自有文件域
    │
    ▼
[4. Verification Phase] ← Evaluator-Optimizer 循环
    • Hook: Stop hook 在测试通过前阻止停止
    • Hook: Agent hook 运行 linter、类型检查
    • 失败 → 反馈 → 修复 → 重新验证
    │
    ▼
[5. PR Creation + Review]
    • Agent 提交 + gh pr create
    • GitHub Action 触发 bmad-code-review
    • Merge Queue 验证 + 合并
```

#### D. 子代理 vs Agent Teams 选择

| 维度 | 子代理 | Agent Teams |
|---|---|---|
| 通信 | 只向主代理报告 | Teammate 间直接消息 |
| 协调 | 集中式（主代理管理） | 分布式（共享任务列表 + 自认领） |
| 上下文 | 隔离 | 隔离 |
| Token 成本 | 较低（结果摘要回传） | 较高（每个 Teammate 是完整实例，约 7x） |
| 适用场景 | 聚焦任务，只关心结果 | 需要代理间讨论/辩论的工作 |

**SE Harness 推荐**：首期使用子代理（Prompt Chaining 模式）；仅在并行 story 实现时升级到 Agent Teams。

_Source: https://code.claude.com/docs/en/agent-teams, https://code.claude.com/docs/en/sub-agents_

#### E. 编排者模型选择

- **编排者**使用最强模型（Opus）——处理分解和合成
- **Workers** 可使用更快/更便宜的模型（Haiku、Sonnet）执行专业任务
- **Routing 模式**：按难度路由——简单查询 → Haiku，复杂推理 → Opus

_Source: https://www.anthropic.com/research/building-effective-agents_

### R6 架构决策：多 Agent 协调架构

#### A. 协调拓扑选择

| 模式 | 优势 | 劣势 | 推荐场景 |
|---|---|---|---|
| 集中式编排者 | 子任务运行时动态确定 | 编排者成为瓶颈 | 子任务不可预测时 |
| 分布式对等 | 吞吐量高 | 协调开销 | 任务边界明确时 |
| SOP 流水线 | 阶段间结构化验证 | 灵活性低 | 角色专业化工作流 |

**SE Harness 推荐**：集中式编排者（Harness 本身）+ SOP 流水线（BMad Method 的阶段化流程）。

_Source: https://www.anthropic.com/research/building-effective-agents, https://arxiv.org/abs/2308.08155 (MetaGPT)_

#### B. Git 隔离架构权衡

| 隔离方式 | 并行性 | 隔离强度 | 开销 | SE Harness 适用性 |
|---|---|---|---|---|
| **Worktree** | 真正并行签出 | 文件系统级 | 低（共享对象存储） | 首选——原生支持 |
| **Branch** | 需顺序切换 | 逻辑级 | 最低 | 单 Agent 回退方案 |
| **Fork/Clone** | 完全独立 | 最强 | 高（完整 clone） | 不信任场景 |
| **Container** | 完全独立 | 最强 + 安全边界 | 最高 | 企业安全要求 |

**SE Harness 推荐**：Worktree 为主（`claude -w`），子模块项目回退到 Branch。

_Source: https://git-scm.com/docs/git-worktree, https://arxiv.org/abs/2403.08299 (AutoDev)_

#### C. 冲突检测架构

_乐观 vs 悲观并发控制_
- **乐观**（主流）：Agent 独立工作，Merge 时检测冲突。GitHub Merge Queue 实现此模式
- **悲观**（文件锁定）：Agent Teams 用文件锁保障任务认领原子性

_语义冲突 vs 文本冲突_
- Git 3-way merge 只检测**文本冲突**
- 语义冲突（代码合并后可编译但行为错误）需要：
  - CI 测试套件验证
  - 静态分析覆盖（类型检查器、linter）
  - 依赖图分析

**SE Harness 推荐**：乐观并发 + Merge Queue + TaskCompleted Hook 强制测试通过。

_Source: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue, https://git-scm.com/docs/merge-strategies_

#### D. Token 成本架构

_水平扩展 + 有界上下文_
- Self-Organized Agents (SoA) 研究证明：Agent 按问题复杂度自动增殖，**每个 Agent 工作量保持恒定**
- 即使代码规模无限增长，单 Agent Token 消耗不增长

_Model Routing 优化_
- 编排者：Opus（分解/合成）
- Workers：Sonnet/Haiku（执行/搜索）
- 预估 Agent Teams 成本：~7x 标准会话

_Fan-out/Fan-in 模式_
- **Sectioning**：独立子任务并行（前端/后端/测试各一个 Agent）
- **Voting**：相同任务多次执行取共识（多角度代码审查）
- Nx/Turborepo 的依赖图驱动分发可直接映射到 Agent 任务分配

_Source: https://arxiv.org/abs/2404.02183 (SoA), https://www.anthropic.com/research/building-effective-agents_

#### E. Monorepo 工具集成

Nx 和 Turborepo 都支持依赖图驱动的任务分发：
- `dependsOn` + `^` 微语法表达跨 package 依赖
- 任务调度器自动最大化并行度
- Nx Agents（云功能）基于历史执行时间 + 依赖图分配任务到 Agent
- **自然映射**：每个 AI Agent 拥有一个或一组 package，依赖图自动编排执行顺序

_Source: https://nx.dev/ci/features/distribute-task-execution, https://turborepo.dev/repo/docs/crafting-your-repository/configuring-tasks_

### R7 架构决策：配置化架构

#### A. 渐进式披露架构（SE Harness 六层模型）

| 层级 | 能力 | 用户操作 | 复杂度 |
|---|---|---|---|
| L0 零配置 | `cd project && claude` 即可工作 | 无 | 零 |
| L1 项目指令 | 添加 `CLAUDE.md`（构建命令、编码规范） | `/init` 自动生成 | 极低 |
| L2 Skills | 添加 `.claude/skills/` 自定义命令 | 创建 SKILL.md | 低 |
| L3 Hooks | 添加 `.claude/settings.json` 自动化 | 配置 JSON | 中 |
| L4 MCP | 连接外部工具和数据源 | 注册 MCP 服务器 | 中高 |
| L5 Plugins | 打包分发配置 | 创建 plugin.json | 高 |
| L6 Managed | 企业策略强制 | IT 部署 | 最高 |

**每层独立可采用**——L1 足够时无需进入 L3。

_Source: https://code.claude.com/docs/en/settings, https://code.claude.com/docs/en/skills, https://code.claude.com/docs/en/plugins_

#### B. 配置合并策略

| 策略 | 行为 | 使用场景 |
|---|---|---|
| **Override** | 高优先级完全替换低优先级 | 标量值、settings.json 中的字符串 |
| **Deep Merge** | 对象递归合并，标量冲突高优先级赢 | VS Code 对象设置、managed-settings.d/ |
| **Append** | 数组拼接去重 | Claude Code CLAUDE.md（拼接而非覆盖） |

**CLAUDE.md 的关键区别**：祖先目录的文件**全部加载拼接**——没有覆盖。最近（最低层级）的文件最后被读取，利用近因效应但不压制上层上下文。

_Source: https://code.claude.com/docs/en/memory_

#### C. 自动发现 vs 显式配置

| 行为 | 自动发现 | 显式配置 |
|---|---|---|
| CLAUDE.md | 向上遍历目录树 | - |
| Skills | `.claude/skills/` 扫描 | - |
| Plugin 组件 | 默认目录名 | - |
| Hooks | - | 必须写入 settings |
| MCP 服务器 | - | 必须注册 |
| Plugin 目录 | - | `--plugin-dir` |

**设计原则**：自动发现用于提供上下文（无副作用）；显式配置用于有副作用的能力（阻断动作、调用外部服务、执行代码）。

_Source: https://code.claude.com/docs/en/settings_

#### D. 硬编码 vs 配置化决策框架

| 配置类别 | 动态性 | 生命周期 | SE Harness 决策 |
|---|---|---|---|
| 安全约束 | 静态 | 永久 | **硬编码** |
| 核心行为逻辑 | 静态 | 永久 | **硬编码** |
| 模型选择 | 会话级 | 每次运行 | **环境变量** |
| API 密钥 | 部署级 | 永久 | **环境变量** |
| 编码规范 | 项目级 | 长期 | **CLAUDE.md** |
| 权限规则 | 项目级 | 长期 | **settings.json** |
| 组织策略 | 组织级 | 永久 | **managed settings** |

**12-Factor 边界测试**：代码库能否开源而不暴露凭证？如果能，代码/配置分离正确。

_Source: https://12factor.net/config, https://martinfowler.com/articles/feature-toggles.html_

#### E. 架构决策记录（ADR）格式

对于 SE Harness 的每个配置边界决策，推荐使用 Y-statement 格式：

> "In the context of **\<situation\>**, facing **\<concern\>**, we decided **\<option\>**, to achieve **\<quality\>**, accepting **\<downside\>**."

示例：
> "In the context of **story implementation automation**, facing **need for session isolation between create-story and dev-story**, we decided **to enforce context:fork on all BMad execution skills**, to achieve **zero context pollution between phases**, accepting **loss of inter-phase context continuity (must explicitly pass via files)**."

_Source: https://adr.github.io/_

### 架构决策总结表

| 决策 | 选项 A | 选项 B | SE Harness 选择 | 理由 |
|---|---|---|---|---|
| 代理模式 | Prompt Chaining | Orchestrator-Workers | 首期 Chaining，扩展时 O-W | 简单优先 |
| 隔离方式 | Worktree | Container | Worktree | 原生支持，低开销 |
| 并发控制 | 乐观 | 悲观 | 乐观 + Merge Queue | 行业标准 |
| 配置格式 | 纯 YAML | Markdown + YAML frontmatter | Markdown + YAML | 行业趋势 |
| 质量门控 | Hook（确定性） | Prompt（概率性） | 二进制规则用 Hook，判断用 Prompt | 分层使用 |
| 模型路由 | 单一模型 | 按难度路由 | 按难度路由 | 成本优化 |
| 配置发现 | 全自动 | 全显式 | 无副作用自动，有副作用显式 | 安全性 |

_Source: 综合以上所有调研来源_

## Implementation Approaches and Technology Adoption

### 渐进式采纳策略（三阶段）

| 阶段 | 模式 | 人工参与 | SE Harness 对应 |
|---|---|---|---|
| Phase 1 辅助 | `@claude` 评论触发 | 每次动作人工发起 | MVP 验证 |
| Phase 2 半自动 | PR 事件自动触发 | 人工审查+合并 | 首期目标 |
| Phase 3 全自动 | Cron + 事件驱动 | 仅异常介入 | 远期目标 |

_Phase 1 详细_
- 安装 GitHub app，添加 `ANTHROPIC_API_KEY`，复制 `claude.yml` 到 `.github/workflows/`
- `@claude` 在 PR/Issue 评论中触发
- Claude 响应分析或代码变更到分支
- 零自动化，完全人工控制

_Phase 2 详细_
- 触发器：`pull_request: [opened, synchronize]`
- Claude 自动运行但仅限范围任务（代码审查、标签、文档同步）
- `--max-turns 5` 限定执行深度
- `--allowedTools "Bash(gh pr comment:*),Read"` 限制可用工具
- CLAUDE.md 自动强制项目规范

_Phase 3 详细_
- Cron 触发：夜间 CI 失败分析、每周依赖审计
- `claude -p "prompt" --permission-mode dontAsk` 完全非交互
- 预算控制：`--max-turns`、`--max-budget-usd`

**关键设计决策**：Claude **不会自动创建 PR**——它提交代码到新分支并提供 PR 创建链接。人工必须手动创建 PR。这是 Anthropic 强制的人工监督检查点。

_Source: https://code.claude.com/docs/en/github-actions, https://code.claude.com/docs/en/best-practices_

### AI 生成代码的测试策略

_核心原则_："Claude performs dramatically better when it can verify its own work."

_推荐做法_
1. **测试优先生成**：先请求测试，再请求实现——建立行为契约
2. **增量测试**："Write one file, test it, then continue"——尽早发现问题
3. **子代理审查**：实现后，生成无偏见的审查代理检查边缘情况
4. **分离安全审查**：`src/auth/**` 路径触发专门的 OWASP Top-10 分析 workflow

_AI PR 更严格的 CI_
- 外部贡献者工作流自动应用"更严格的审查标准"（`author_association == FIRST_TIME_CONTRIBUTOR`）
- 同一机制可条件应用于 AI agent 的 PR
- `track_progress: true` 提供视觉进度勾选框 + 完整审计轨迹

_Source: https://code.claude.com/docs/en/best-practices, https://github.com/anthropics/claude-code-action/blob/main/docs/solutions.md_

### 成本管理

#### 当前定价（2026 年 4 月）

| 模型 | 输入 | 输出 | 缓存命中 |
|---|---|---|---|
| Claude Opus 4.6 | $5/MTok | $25/MTok | $0.50/MTok |
| Claude Sonnet 4.6 | $3/MTok | $15/MTok | $0.30/MTok |
| Claude Haiku 4.5 | $1/MTok | $5/MTok | $0.10/MTok |

#### 经验成本基准

| 指标 | 数值 |
|---|---|
| 开发者日均成本 | ~$6 |
| 90 分位日成本上限 | <$12 |
| 月度估算（Sonnet 4.6） | ~$100-200/开发者 |
| Agent Teams 倍率 | ~7x 标准会话 |

#### 成本优化策略

1. **模型分层**：Haiku 做子代理搜索/检索（$1/$5），Sonnet 做 CI 审查（$3/$15），Opus 仅做架构决策（$5/$25）——最高 5 倍差距
2. **Prompt 缓存**：系统提示 + CLAUDE.md 自动缓存；5 分钟缓存 1.25x 写入 + 0.1x 读取 = 1 次读取即回本
3. **上下文卫生**：`/clear` 清理无关任务上下文；CLAUDE.md 控制在 200 行以内
4. **卸载到 Hook**：log 文件先用 grep 预处理再给 Claude（10k token 日志 → 100 token 错误摘要）
5. **CI 预算控制**：
   ```yaml
   claude_args: |
     --max-turns 5              # 审查任务
     --model claude-sonnet-4-6  # CI 用 Sonnet 不用 Opus
   ```

_Source: https://code.claude.com/docs/en/costs, https://platform.claude.com/docs/en/about-claude/pricing_

### 安全风险评估

| 风险 | 严重性 | 缓解措施 |
|---|---|---|
| PR 内容注入提示 | 高 | Action 剥离 HTML 注释、不可见字符；`include_comments_by_actor` 过滤 |
| API 密钥暴露 | 严重 | 始终用 `${{ secrets.ANTHROPIC_API_KEY }}`；禁止硬编码 |
| 公共仓库允许所有 bot | 高 | 显式 `allowed_bots` 列表，公共仓库禁用 `'*'` |
| `show_full_output: true` 泄露密钥 | 高 | 默认 `false`；仅私有仓库调试时启用 |
| Agent 修改受保护基础设施 | 高 | Auto 模式分类器默认阻止：生产部署、迁移、force push main |

#### 爆炸半径控制

- **工具白名单**（最重要的控制）：`--allowedTools "Bash(gh pr comment:*),Read"` 阻止文件写入、shell 执行、网络调用
- **`dontAsk` 模式**：`claude --permission-mode dontAsk` —— 自动拒绝所有未预批准的工具
- **Auto 模式分类器**（研究预览）：独立分类器模型审查每个动作；阻止范围升级和未知基础设施操作
- **受保护路径**（任何模式下都不自动批准）：`.git`、`.vscode`、`.idea`、`.husky`

#### 回滚机制

- **检查点**：每次变更前自动快照，`Esc+Esc` 或 `/rewind` 恢复
- **Worktree 隔离**：每个自动化任务获得隔离分支，main 永不直接触碰
- **分支保护**：AI 生成代码通过 PR 流转；分支保护规则和审批要求仍然适用

_Source: https://github.com/anthropics/claude-code-action/blob/main/docs/security.md, https://code.claude.com/docs/en/permission-modes_

### Agent Teams 实现指南

#### 启用方式
```json
// .claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

#### 已知限制（2026 年）
- 实验性功能，默认禁用，需 Claude Code v2.1.32+
- 无会话恢复（`/resume` 和 `/rewind` 不恢复 Teammates）
- 不支持嵌套团队（Teammates 不能再创建团队）
- Lead 固定不可转让
- 分屏需要 tmux 或 iTerm2（VS Code Terminal 不支持）

#### 常见陷阱
- **文件冲突**：两个 Teammate 编辑同一文件导致覆写——分配不相交的文件集
- **Lead 亲自执行而非委托**：明确指示 "Wait for your teammates to complete"
- **任务状态滞后**：Teammate 有时忘记标记任务完成
- **Token 成本爆炸**：保持团队小规模（3-5）；Teammate 用 Sonnet 不用 Opus

_Source: https://code.claude.com/docs/en/agent-teams_

### Hooks 实战配置

#### Auto-Format（PostToolUse）
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{"type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"}]
    }]
  }
}
```

#### 文件保护（PreToolUse）
```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
PROTECTED=(".env" "package-lock.json" ".git/")
for p in "${PROTECTED[@]}"; do
  [[ "$FILE_PATH" == *"$p"* ]] && { echo "Blocked: $FILE_PATH" >&2; exit 2; }
done
```

#### 测试验证（Stop Hook - Agent 类型）
```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "agent",
        "prompt": "Verify all unit tests pass. Run the test suite.",
        "timeout": 120
      }]
    }]
  }
}
```

_Source: https://code.claude.com/docs/en/hooks-guide_

### CLAUDE.md 编写最佳实践

_内容建议_

| 应包含 | 不应包含 |
|---|---|
| Claude 无法猜到的 bash 命令 | Claude 能从代码推断的信息 |
| 不同于默认值的代码风格规则 | 标准语言约定 |
| 测试指令和首选测试运行器 | 详细 API 文档（改用链接） |
| 仓库礼仪（分支命名、PR 约定） | 频繁变化的信息 |
| 项目特定的架构决策 | 长篇解释或教程 |
| 开发环境怪癖（必需的环境变量） | 逐文件代码库描述 |

_关键规则_
- **目标 200 行以内**——更长的文件降低遵循度
- **用 Markdown 标题和列表**——比密集段落更易遵循
- **具体可验证**："Use 2-space indentation" > "Format code properly"
- **定期修剪**：对每行问"移除这行会导致 Claude 犯错吗？"

_Source: https://code.claude.com/docs/en/memory, https://code.claude.com/docs/en/best-practices_

### 成功指标和 KPI

#### 速度指标
- 任务完成时间（AI 采纳前后对比）——基准：AI 辅助下 **55% 更快**
- PR 从分支创建到合并的周期时间
- Story 从创建到完成的时间

#### 质量指标
- AI 首次提交的测试通过率
- 每 PR 代码审查迭代次数
- Hook 阻断次数/会话（衡量质量门控有效性）

#### 采纳指标
- Claude Code 日活用户数
- 每开发者每日会话数
- Claude 触及的代码变更百分比

#### 开发者体验指标（SPACE 框架对齐）
- 60-75% 开发者报告更高的工作满足感
- 87% 开发者在重复工作中保留了心智能量
- 73% 开发者维持了更好的专注和心流状态

#### 成本效率指标
- 每完成一个 story 的 Token 成本
- 每合并一个 PR 的成本
- API 支出 vs 开发者节省的时间

_Source: https://github.blog/ai-and-ml/github-copilot/research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/, https://code.claude.com/docs/en/costs_

## Technical Research Recommendations

### 实现路线图

**Phase 0（1-2 周）：环境验证**
- 在目标项目安装 BMad Method + 配置 CLAUDE.md
- 手动运行完整链路：`/bmad-create-story` → `claude -w` → `/bmad-dev-story` → `/bmad-code-review`
- 验证 worktree 创建/清理、PR 关联、代码审查流程

**Phase 1（2-4 周）：辅助自动化**
- 配置 `claude-code-action@v1`，启用 `@claude` PR 评论触发
- 实现基础 Hooks：PostToolUse auto-format + PreToolUse 文件保护
- 建立成本监控（日均 $6 基准）和安全审计

**Phase 2（4-8 周）：半自动交付**
- PR 事件自动触发代码审查
- Stop Hook 强制测试通过
- 实验 Agent Teams（小规模 2-3 Teammate）
- 建立 KPI 基线（PR 周期时间、测试通过率、成本/story）

**Phase 3（8-12 周）：全自动交付管线**
- Cron 触发定期维护任务
- `--permission-mode dontAsk` 完全非交互 CI
- Merge Queue 集成多 Agent PR
- 全面成本优化（模型分层 + Prompt 缓存）

### 技术栈推荐

| 组件 | 推荐 | 理由 |
|---|---|---|
| 执行引擎 | Claude Code CLI + Agent SDK | 原生支持，最完整的工具链 |
| 开发方法论 | BMad Method | 结构化 Story 流程，门控审查 |
| CI/CD | GitHub Actions + claude-code-action@v1 | 官方支持，事件驱动 |
| 隔离 | Git Worktree | 原生支持，低开销 |
| 合并 | GitHub Merge Queue | 多 Agent PR 冲突预检测 |
| 配置 | CLAUDE.md + settings.json + Skills | 行为引导 + 技术强制 + 工作流封装 |
| 成本模型 | Sonnet 主力 + Haiku 子代理 + Opus 架构决策 | 5x 成本差异的分层路由 |

### 技能发展要求

| 角色 | 需掌握 | 优先级 |
|---|---|---|
| 所有开发者 | CLAUDE.md 编写、`@claude` PR 交互 | P0 |
| Tech Lead | Hook 配置、settings.json 权限模型、成本监控 | P0 |
| DevOps | claude-code-action 配置、Merge Queue、安全加固 | P1 |
| 架构师 | Agent SDK 编排、Agent Teams 设计、模型路由 | P1 |

## Research Synthesis and Conclusion

### R5 可行性验证结论

**结论：可行（高置信度）**

Claude Code + BMad Method 已具备 Story 完整交付链路的所有技术原语：

| 链路环节 | 技术支撑 | 成熟度 |
|---|---|---|
| Story 生成 | BMad `create-story` skill | GA |
| 隔离执行 | `claude -w` worktree 隔离 | GA |
| Story 实现 | BMad `dev-story` + `context: fork` | GA |
| 自动提交 | `gh pr create` 自动关联会话 | GA |
| 对抗性审查 | BMad `code-review`（新会话） | GA |
| CI 集成 | `claude-code-action@v1` | GA |
| 自动合并 | GitHub Merge Queue | GA |
| 成本控制 | `--max-turns`, `--max-budget-usd`, 模型分层 | GA |

_关键约束_
- Claude 不会自动创建 PR（Anthropic 强制的人工检查点）
- 每个 BMad skill 需独立新会话（`context: fork`）防止上下文污染
- CLAUDE.md 控制在 200 行以内以保证遵循度

_推荐架构_：首期 Prompt Chaining（固定链路），扩展时升级到 Orchestrator-Workers。

### R6 可行性验证结论

**结论：可行（中高置信度）**

Claude Code 是目前唯一具有文档化多 Agent 同 Repo 支持的 AI 编码工具。四种并行机制提供从轻量到完全隔离的选择：

| 机制 | 隔离级别 | 开销 | 成熟度 | 推荐阶段 |
|---|---|---|---|---|
| Worktree (`-w`) | 文件系统 | 低 | GA | 首期 |
| 子代理 (`isolation: worktree`) | 文件系统 | 低 | GA | 首期 |
| Agent Teams | 文件系统 + 任务锁 | 中（7x token） | 实验性 | Phase 2 |
| 云 VM (`--remote`) | 完全独立 | 高 | GA | 企业级 |

_核心原则验证_
- "按文件所有权分配任务，避免两个 Agent 编辑同一文件" —— 已被多个来源证实为有效策略
- 五层防冲突方案（任务域划分 → Hook 验证 → Worktree → 子代理隔离 → 目录 CLAUDE.md）提供纵深防御
- GitHub Merge Queue 在合并前检测跨 Agent 集成冲突

_风险与缓解_
| 风险 | 严重性 | 缓解 |
|---|---|---|
| Agent Teams 实验性功能可能变动 | 中 | 首期不依赖 Agent Teams，用 Worktree + 子代理 |
| 文件所有权划分不清导致冲突 | 高 | TaskCreated Hook 验证文件域 + CODEOWNERS |
| Token 成本随 Agent 数线性增长 | 中 | 模型分层（Teammate 用 Sonnet）+ 控制团队规模 3-5 |
| 语义冲突（文本合并无冲突但行为错误） | 高 | TaskCompleted Hook 强制测试 + 集成测试套件 |

### R7 可行性验证结论

**结论：可行（高置信度）**

行业已收敛到成熟的配置模式，SE Harness 可直接采用：

| 配置层 | 格式 | 用途 | SE Harness 映射 |
|---|---|---|---|
| 行为引导 | Markdown (CLAUDE.md) | LLM 消费的指令 | 项目编码规范、架构决策 |
| 技术强制 | JSON (settings.json) | 客户端强制的规则 | 权限、Hook、工具白名单 |
| 工作流封装 | SKILL.md (YAML + Markdown) | 可复用的任务指令 | BMad skills、自定义 skills |
| 组织策略 | managed-settings.json | 不可覆盖的强制策略 | 安全策略、合规要求 |

_配置化 vs 硬编码决策_

| 首期硬编码 | 可配置化 | 远期配置化 |
|---|---|---|
| Story 链路步骤顺序 | 目标项目路径 | Agent 团队规模 |
| 质量门控规则（测试必须通过） | 模型选择 | 自定义审查规则 |
| Worktree 隔离策略 | API 密钥 | 并行策略 |
| BMad skill 新会话要求 | CLAUDE.md 内容 | Hook 配置 |

_渐进式披露路径_：L0 零配置 → L1 CLAUDE.md → L2 Skills → L3 Hooks → L4 MCP → L5 Plugins → L6 Managed。每层独立可采用。

### 综合风险评估

| 风险类别 | 具体风险 | 影响 | 概率 | 缓解策略 |
|---|---|---|---|---|
| 技术 | Agent Teams API 变动 | 中 | 中 | 不依赖实验性 API，用 GA 原语 |
| 技术 | 上下文窗口耗尽导致质量下降 | 高 | 中 | `context: fork` 隔离 + `/clear` + 200 行 CLAUDE.md |
| 安全 | PR 内容提示注入 | 高 | 中 | `include_comments_by_actor` + 工具白名单 |
| 成本 | Token 成本超预期 | 中 | 低 | 模型分层 + `--max-turns` + 预算监控 |
| 流程 | BMad skill 链路断裂 | 高 | 低 | 每步验证 + Hook 质量门控 |
| 人员 | 团队学习曲线 | 中 | 高 | Phase 1 辅助模式建立肌肉记忆 |

### 下一步行动建议

1. **立即（本周）**：在目标项目手动走通完整 Story 链路（Phase 0 验证），记录每步耗时和问题
2. **短期（2 周内）**：配置 `claude-code-action@v1` + 基础 Hooks（auto-format + 文件保护 + 测试验证）
3. **中期（4 周内）**：建立成本监控 + KPI 基线，验证 Worktree 并行 2 个 Story 的可行性
4. **中期（8 周内）**：评估 Agent Teams 实验，建立 Merge Queue 多 PR 工作流
5. **持续**：每月回顾 KPI（PR 周期时间、测试通过率、Token 成本/story），调整配置

---

### Source Documentation

**Primary Sources (Official Documentation)**
- Claude Code: https://code.claude.com/docs/en/overview
- Claude Agent SDK: https://platform.claude.com/docs/en/agent-sdk/overview
- Claude Code GitHub Actions: https://code.claude.com/docs/en/github-actions
- Claude Code Hooks: https://code.claude.com/docs/en/hooks-guide
- Claude Code Settings: https://code.claude.com/docs/en/settings
- Claude Code Agent Teams: https://code.claude.com/docs/en/agent-teams
- Claude Code Skills: https://code.claude.com/docs/en/skills
- Claude Code Permissions: https://code.claude.com/docs/en/permissions
- Claude Code Cost Management: https://code.claude.com/docs/en/costs
- Claude Code Best Practices: https://code.claude.com/docs/en/best-practices

**Secondary Sources**
- Anthropic "Building Effective Agents": https://www.anthropic.com/research/building-effective-agents
- Agent Skills Open Standard: https://agentskills.io/specification
- BMad Method: https://github.com/bmadcode/bmad-method
- GitHub Merge Queue: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue
- Git Worktree: https://git-scm.com/docs/git-worktree
- 12-Factor App: https://12factor.net/config
- GitHub Copilot Productivity Research: https://github.blog/ai-and-ml/github-copilot/research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/
- claude-code-action Security Guide: https://github.com/anthropics/claude-code-action/blob/main/docs/security.md

**Research Methodology Sources**
- CrewAI: https://docs.crewai.com/concepts/crews
- LangGraph: https://blog.langchain.com/langgraph-multi-agent-workflows
- AutoGen: https://microsoft.github.io/autogen/stable/
- MetaGPT: https://arxiv.org/abs/2308.08155
- Self-Organized Agents: https://arxiv.org/abs/2404.02183

---

**Technical Research Completion Date:** 2026-04-08
**Research Period:** Comprehensive technical analysis with current (2025-2026) sources
**Source Verification:** All technical facts cited with URLs to authoritative sources
**Confidence Level:** High — based on multiple independent authoritative sources with cross-validation
