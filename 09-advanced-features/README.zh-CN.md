<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 高级功能

Claude Code 高级能力的综合指南，包括规划模式、扩展思考、auto 模式、后台任务、权限模式、打印模式（非交互式）、会话管理、交互功能、语音输入、channels、远程控制、Web 会话、桌面应用、任务列表、提示词建议、git worktree、沙盒、托管设置和配置。

## 目录

1. [概述](#概述)
2. [规划模式](#规划模式)
3. [扩展思考](#扩展思考)
4. [Auto 模式](#auto-模式)
5. [后台任务](#后台任务)
6. [定时任务](#定时任务)
7. [权限模式](#权限模式)
8. [无头模式](#无头模式)
9. [会话管理](#会话管理)
10. [交互功能](#交互功能)
11. [语音输入](#语音输入)
12. [Channels](#channels)
13. [Chrome 集成](#chrome-集成)
14. [远程控制](#远程控制)
15. [Web 会话](#web-会话)
16. [桌面应用](#桌面应用)
17. [任务列表](#任务列表)
18. [提示词建议](#提示词建议)
19. [Git Worktrees](#git-worktrees)
20. [沙盒](#沙盒)
21. [托管设置（企业版）](#托管设置企业版)
22. [配置和设置](#配置和设置)
23. [最佳实践](#最佳实践)
24. [其他资源](#其他资源)

---

## 概述

Claude Code 中的高级功能扩展了核心能力，提供了规划、推理、自动化和控制机制。这些功能支持复杂开发任务、代码审查、自动化和多会话管理的复杂工作流。

**主要高级功能包括：**
- **规划模式**：编码前创建详细的实施计划
- **扩展思考**：为复杂问题进行深度推理
- **Auto 模式**：后台安全分类器在执行前审查每个操作（研究预览）
- **后台任务**：运行长时间操作而不阻塞对话
- **权限模式**：控制 Claude 可以做什么（`default`、`acceptEdits`、`plan`、`auto`、`dontAsk`、`bypassPermissions`）
- **打印模式**：非交互式运行 Claude Code，用于自动化和 CI/CD（`claude -p`）
- **会话管理**：管理多个工作会话
- **交互功能**：键盘快捷键、多行输入和命令历史
- **语音输入**：支持 20 种语言的按键说话语音输入
- **Channels**：MCP 服务器将消息推送到运行的会话中（研究预览）
- **远程控制**：从 claude.ai 或 Claude 应用控制 Claude Code
- **Web 会话**：在 claude.ai/code 的浏览器中运行 Claude Code
- **桌面应用**：用于可视化 diff 审查和多个会话的独立应用
- **任务列表**：跨上下文压缩持久化的任务跟踪
- **提示词建议**：基于上下文的智能命令建议
- **Git Worktrees**：用于并行工作的隔离 worktree 分支
- **沙盒**：操作系统级文件系统和网络隔离
- **托管设置**：通过 plist、Registry 或托管文件进行企业部署
- **配置**：使用 JSON 配置文件自定义行为

---

## 规划模式

规划模式允许 Claude 在实施之前仔细思考复杂任务，创建一个你可以审查和批准的详细计划。

### 什么是规划模式？

规划模式是一种两阶段方法：
1. **规划阶段**：Claude 分析任务并创建详细的实施计划
2. **实施阶段**：审批后，Claude 执行计划

### 何时使用规划模式

✅ 使用规划模式用于：
- 复杂的多文件重构
- 新功能实施
- 架构更改
- 数据库迁移
- 主要 API 重新设计

❌ 不要使用规划模式用于：
- 简单的 bug 修复
- 格式化更改
- 单文件编辑
- 快速查询

### 激活规划模式

**斜杠命令**：
```bash
/plan Implement user authentication system
```

**CLI 标志**：
```bash
claude --permission-mode plan
```

**设为默认**：
```json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

**键盘快捷键**：
- `Shift + Tab` - 切换权限模式（包括 plan）
- `Alt + M` - 切换权限模式（Windows/Linux）

### 规划模式的优势

- **结构化的实施方法**：详细的实施步骤分解
- **审查和批准**：执行前审查或调整计划
- **风险识别**：实施前识别潜在问题
- **清晰的阶段**：有组织的实施阶段和里程碑

### 示例：功能实施

```
User: /plan Add a real-time notification system

Claude: I'll create a plan for implementing real-time notifications.

## Implementation Plan

### Phase 1: Infrastructure Setup
1. Install WebSocket libraries (socket.io)
2. Set up WebSocket server
3. Configure connection management

### Phase 2: Backend Implementation
4. Create notification service
5. Implement notification queue
6. Add database schema for notifications
7. Create API endpoints

### Phase 3: Frontend Integration
8. Set up WebSocket client
9. Create notification UI components
10. Implement notification state management
11. Add notification permissions

### Phase 4: Testing & Deployment
12. Write unit tests
13. Write integration tests
14. Update documentation
15. Deploy to staging

Estimated time: 3-4 hours
Files to modify: 12
New files to create: 8

Ready to proceed? (yes/no/modify plan)

User: yes

Claude: [Begins implementation following the plan]
```

### 计划修改

```
User: Modify the plan - skip the queue for now, we'll add it later

Claude: Updated plan:
[Shows modified plan with queue removed]

User: Looks good, proceed

Claude: [Implements modified plan]
```

### 规划模式配置

通过 CLI 标志或斜杠命令激活规划模式：

```bash
# Activate plan mode via CLI
claude --permission-mode plan

# Or use the /plan slash command inside the REPL
/plan Implement user authentication system
```

**规划的模型别名**：使用 `opusplan` 作为模型别名，用 Opus 进行规划，Sonnet 执行：

```bash
claude --model opusplan "design and implement the new API"
```

**外部编辑计划**：按 `Ctrl+G` 在外部编辑器中打开当前计划进行详细修改。

---

## 扩展思考

扩展思考允许 Claude 在提供解决方案之前花更多时间推理复杂问题。

### 什么是扩展思考？

扩展思考是一种深思熟虑的逐步推理过程，Claude 在其中：
- 分解复杂问题
- 考虑多种方法
- 评估权衡
- 推理边缘情况

### 激活扩展思考

**键盘快捷键**：
- `Option + T`（macOS）/ `Alt + T`（Windows/Linux）- 切换扩展思考

**自动激活**：
- 默认情况下为所有模型启用（Opus 4.6、Sonnet 4.6、Haiku 4.5）
- Opus 4.6：具有努力级别的自适应推理：`low`（○）、`medium`（◐）、`high`（●）、`max`（仅 Opus 4.6）
- 其他模型：固定预算最多 31,999 tokens

**配置方法**：
- 切换：`Alt+T` / `Option+T`，或通过 `/config`
- 查看推理：`Ctrl+O`（详细模式）
- 设置努力级别：`/effort` 命令或 `--effort` 标志

**自定义预算**：
```bash
export MAX_THINKING_TOKENS=1024
```

**努力级别**（仅 Opus 4.6）：
```bash
export CLAUDE_CODE_EFFORT_LEVEL=high   # low (○), medium (◐), high (●), or max (仅 Opus 4.6)
```

**CLI 标志**：
```bash
claude --effort high "complex architectural review"
```

**斜杠命令**：
```
/effort high
```

> **注意：** 提示词中的关键字"ultrathink"激活深度推理模式。努力级别 `low`、`medium`、`high` 和 `max`（仅 Opus 4.6）控制 Claude 执行的推理量。

### 扩展思考的优势

- **深度推理**：花更多时间思考复杂问题
- **更好的架构决策**：通过综合评估做出明智选择
- **边缘情况考虑**：仔细思考潜在问题和场景
- **彻底分析**：复杂问题解决的结构化方法

### 示例：架构决策

```
User: /think Should we use microservices or a monolith for our e-commerce platform?

Claude: Let me think through this architectural decision carefully...

<extended_thinking>
## Context Analysis
- Team size: 5 developers
- Expected traffic: 10k daily users initially
- Need to scale to 100k+ users
- Limited DevOps resources
- 6-month timeline to MVP

## Approach 1: Microservices
Pros:
+ Independent scaling
+ Technology flexibility
+ Team independence
+ Fault isolation

Cons:
- Operational complexity
- Network overhead
- Distributed tracing needed
- Requires DevOps expertise
- Slower initial development

## Approach 2: Monolith
Pros:
+ Simpler deployment
+ Faster initial development
+ Easier debugging
+ Lower operational overhead
+ Single codebase

Cons:
- Scaling challenges later
- All-or-nothing deployments
- Technology lock-in
- Potential for tight coupling

## Decision Factors
1. Team size (5 devs) → Favors monolith
2. DevOps resources (limited) → Favors monolith
3. Timeline (6 months) → Favors monolith
4. Future scale (100k users) → Achievable with monolith + caching
5. Complexity budget → Limited

## Recommendation
Start with a **modular monolith**:
- Single deployment unit
- Clear module boundaries
- Clean interfaces between modules
- Prepare for future extraction to microservices if needed

This gives you:
✓ Fast initial development
✓ Low operational complexity
✓ Option to extract services later
✓ Clear architecture

Migration path: When you reach 50k+ users or specific modules need independent scaling, extract those modules into microservices.
</extended_thinking>

**Recommendation**: Implement a modular monolith with clean module boundaries.

This approach balances your current constraints (team size, timeline, DevOps resources) with future scalability needs. You can migrate specific modules to microservices later when there's a clear business need.
```

### 扩展思考配置

扩展思考通过环境变量、键盘快捷键和 CLI 标志控制：

```bash
# Set thinking token budget
export MAX_THINKING_TOKENS=16000

# Set effort level (仅 Opus 4.6): low (○), medium (◐), high (●), or max (仅 Opus 4.6)
export CLAUDE_CODE_EFFORT_LEVEL=high
```

在会话中使用 `Alt+T` / `Option+T` 切换，使用 `/effort` 设置努力级别，或通过 `/config` 配置。

---

## Auto 模式

Auto 模式是一种研究预览权限模式（2026 年 3 月），使用后台安全分类器在执行前审查每个操作。它允许 Claude 自主工作，同时阻止危险操作。

### 要求

- **计划**：Team 计划（企业和 API 正在推出）
- **模型**：Claude Sonnet 4.6 或 Opus 4.6
- **分类器**：在 Claude Sonnet 4.6 上运行（增加额外 token 成本）

### 启用 Auto 模式

```bash
# Unlock auto mode with CLI flag
claude --enable-auto-mode

# Then cycle to it with Shift+Tab in the REPL
```

或将其设置为默认权限模式：

```bash
claude --permission-mode auto
```

通过配置设置：
```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

### 分类器如何工作

后台分类器使用以下决策顺序评估每个操作：

1. **允许/拒绝规则** —— 首先检查显式权限规则
2. **只读/编辑自动批准** —— 文件读取和编辑自动通过
3. **分类器** —— 后台分类器审查操作
4. **回退** —— 连续 3 次或总共 20 次阻止后回退到提示

### 默认阻止的操作

Auto 模式默认阻止以下操作：

| 被阻止的操作 | 示例 |
|----------------|---------|
| 管道到 shell 安装 | `curl \| bash` |
| 外部发送敏感数据 | API 密钥、在网络上发送凭据 |
| 生产部署 | 针对生产环境的部署命令 |
| 批量删除 | 大目录上的 `rm -rf` |
| IAM 更改 | 权限和角色修改 |
| 强制推送到 main | `git push --force origin main` |

### 默认允许的操作

| 允许的操作 | 示例 |
|----------------|---------|
| 本地文件操作 | 读取、写入、编辑项目文件 |
| 声明的依赖安装 | `npm install`、`pip install` 从清单 |
| 只读 HTTP | `curl` 用于获取文档 |
| 推送到当前分支 | `git push origin feature-branch` |

### 配置 Auto 模式

**打印默认规则为 JSON**：
```bash
claude auto-mode defaults
```

**通过 `autoMode.environment` 托管设置为企业部署配置受信任的基础设施**。这允许管理员定义受信任的 CI/CD 环境、部署目标和基础设施模式。

### 回退行为

当分类器不确定时，auto 模式回退到提示用户：
- 连续 **3 次**分类器阻止后
- 会话中总共 **20 次**分类器阻止后

这确保当分类器无法自信地批准操作时，用户始终保持控制。

### 种子 Auto Mode 等效权限（无需 Team 计划）

如果你没有 Team 计划或想要一个更简单的方法而不使用后台分类器，你可以用保守的安全权限规则基线来种子 `~/.claude/settings.json`。脚本从只读和本地检查规则开始，然后让你选择何时加入编辑、测试、本地 git 写入、包安装和 GitHub 写入操作。

**文件：** `09-advanced-features/setup-auto-mode-permissions.py`

```bash
# Preview what would be added (no changes written)
python3 09-advanced-features/setup-auto-mode-permissions.py --dry-run

# Apply the conservative baseline
python3 09-advanced-features/setup-auto-mode-permissions.py

# Add more capability only when you need it
python3 09-advanced-features/setup-auto-mode-permissions.py --include-edits --include-tests
python3 09-advanced-features/setup-auto-mode-permissions.py --include-git-write --include-packages
```

脚本在以下类别中添加规则：

| 类别 | 示例 |
|----------|---------|
| 核心只读工具 | `Read(*)`、`Glob(*)`、`Grep(*)`、`Agent(*)`、`WebSearch(*)`、`WebFetch(*)` |
| 本地检查 | `Bash(git status:*)`、`Bash(git log:*)`、`Bash(git diff:*)`、`Bash(cat:*)` |
| 可选编辑 | `Edit(*)`、`Write(*)`、`NotebookEdit(*)` |
| 可选测试/构建 | `Bash(pytest:*)`、`Bash(python3 -m pytest:*)`、`Bash(cargo test:*)` |
| 可选 git 写入 | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git stash:*)` |
| Git（本地写入） | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git checkout:*)` |
| 包管理器 | `Bash(npm install:*)`、`Bash(pip install:*)`、`Bash(cargo build:*)` |
| 构建和测试 | `Bash(make:*)`、`Bash(pytest:*)`、`Bash(go test:*)` |
| 常用 shell | `Bash(ls:*)`、`Bash(cat:*)`、`Bash(find:*)`、`Bash(cp:*)`、`Bash(mv:*)` |
| GitHub CLI | `Bash(gh pr view:*)`、`Bash(gh pr create:*)`、`Bash(gh issue list:*)` |

危险操作（`rm -rf`、`sudo`、force push、`DROP TABLE`、`terraform destroy` 等）被有意排除。脚本是幂等的——运行两次不会重复添加规则。

---

## 后台任务

后台任务允许长时间运行的操作在不阻塞对话的情况下执行。

### 什么是后台任务？

后台任务异步运行，而你继续工作：
- 长时间测试套件
- 构建进程
- 数据库迁移
- 部署脚本
- 分析工具

**基本用法：**
```bash
User: Run tests in background

Claude: Started task bg-1234

/task list           # Show all tasks
/task status bg-1234 # Check progress
/task show bg-1234   # View output
/task cancel bg-1234 # Cancel task
```

### 启动后台任务

```
User: Run the full test suite in the background

Claude: Starting tests in background (task-id: bg-1234)
You can continue working while tests run.

[You can continue the conversation]

User: Meanwhile, let's refactor the auth module

Claude: [Works on auth module while tests run in background]

[Later, when tests complete]

Claude: 📢 Background task bg-1234 completed:
✅ 245 tests passed
❌ 3 tests failed
View results: /task show bg-1234
```

### 管理后台任务

**列出活动的任务**：
```
User: /task list

Active background tasks:
1. [bg-1234] Running tests (50% complete, 2min remaining)
2. [bg-1235] Building Docker image (25% complete, 8min remaining)
3. [bg-1236] Deploying to staging (90% complete, 30sec remaining)
```

**检查任务状态**：
```
User: /task status bg-1234

Task bg-1234: Running tests
Status: In progress
Progress: 120/245 tests (49%)
Started: 2025-11-08 10:30:15
Estimated completion: 2025-11-08 10:34:22
```

**查看任务输出**：
```
User: /task show bg-1234

[Shows live output from the test run]
```

**取消任务**：
```
User: /task cancel bg-1234

Cancelled background task bg-1234
```

### 示例：并行开发

```
User: Run the build in the background

Claude: Starting build... (task-id: bg-5001)

User: Also run the linter in background

Claude: Starting linter... (task-id: bg-5002)

User: While those run, let's implement the new API endpoint

Claude: [Implements API endpoint while build and linter run]

[10 minutes later]

Claude: 📢 Build completed successfully (bg-5001)
📢 Linter found 12 issues (bg-5002)

User: Show me the linter issues

Claude: [Shows linter output from bg-5002]
```

### 配置

```json
{
  "backgroundTasks": {
    "enabled": true,
    "maxConcurrentTasks": 5,
    "notifyOnCompletion": true,
    "autoCleanup": true,
    "logOutput": true
  }
}
```

---

## 定时任务

定时任务让你按重复计划或作为一次性提醒自动运行提示词。任务是会话作用域的——它们在 Claude Code 处于活动状态时运行，在会话结束时清除。自 v2.1.72+ 起可用。

### `/loop` 命令

```bash
# Explicit interval
/loop 5m check if the deployment finished

# Natural language
/loop check build status every 30 minutes
```

也支持标准 5 字段 cron 表达式以进行精确调度。

### 一次性提醒

设置在特定时间触发一次的提醒：

```
remind me at 3pm to push the release branch
in 45 minutes, run the integration tests
```

### 管理定时任务

| 工具 | 说明 |
|------|-------------|
| `CronCreate` | 创建新的定时任务 |
| `CronList` | 列出所有活动的定时任务 |
| `CronDelete` | 删除定时任务 |

**限制和行为**：
- 每个会话最多 **50 个**定时任务
- 会话作用域——会话结束时清除
- 重复任务在 **3 天**后自动过期
- 任务仅在 Claude Code 运行时触发——错过的触发不会补上

### 行为详情

| 方面 | 详情 |
|--------|--------|
| **重复抖动** | 间隔的最多 10%（最多 15 分钟） |
| **一次性抖动** | :00/:30 边界上最多 90 秒 |
| **错过的触发** | 不补上——如果 Claude Code 未运行则跳过 |
| **持久性** | 不会跨重启持久化 |

### Cloud 定时任务

使用 `/schedule` 创建在 Anthropic 基础设施上运行的 Cloud 定时任务：

```
/schedule daily at 9am run the test suite and report failures
```

Cloud 定时任务跨重启持久化，不需要 Claude Code 在本地运行。

### 禁用定时任务

```bash
export CLAUDE_CODE_DISABLE_CRON=1
```

### 示例：监控部署

```
/loop 5m check the deployment status of the staging environment.
        If the deploy succeeded, notify me and stop looping.
        If it failed, show the error logs.
```

> **提示**：定时任务是会话作用域的。对于跨重启持久化的自动化，请改用 CI/CD 管道、GitHub Actions 或桌面应用定时任务。

---

## 权限模式

权限模式控制 Claude 无需显式批准即可执行哪些操作。

### 可用的权限模式

| 模式 | 行为 |
|---|---|
| `default` | 仅读取文件；其他所有操作都提示 |
| `acceptEdits` | 读取和编辑文件；命令提示 |
| `plan` | 仅读取文件（研究模式，无编辑） |
| `auto` | 所有操作都有后台安全分类器检查（研究预览） |
| `bypassPermissions` | 所有操作，无权限检查（危险） |
| `dontAsk` | 仅预批准工具执行；其他都拒绝 |

使用 `Shift+Tab` 在 CLI 中循环切换模式。使用 `--permission-mode` 标志或 `permissions.defaultMode` 设置设置默认值。

### 激活方法

**键盘快捷键**：
```bash
Shift + Tab  # Cycle through all 6 modes
```

**斜杠命令**：
```bash
/plan                  # Enter plan mode
```

**CLI 标志**：
```bash
claude --permission-mode plan
claude --permission-mode auto
```

**设置**：
```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

### 权限模式示例

#### 默认模式
Claude 对重要操作请求确认：

```
User: Fix the bug in auth.ts

Claude: I need to modify src/auth.ts to fix the bug.
The change will update the password validation logic.

Approve this change? (yes/no/show)
```

#### 规划模式
执行前审查实施计划：

```
User: /plan Implement user authentication system

Claude: I'll create a plan for implementing authentication.

## Implementation Plan
[Detailed plan with phases and steps]

Ready to proceed? (yes/no/modify)
```

#### 接受编辑模式
自动接受文件修改：

```
User: acceptEdits
User: Fix the bug in auth.ts

Claude: [Makes changes without asking]
```

### 使用场景

**代码审查**：
```
User: claude --permission-mode plan
User: Review this PR and suggest improvements

Claude: [Reads code, provides feedback, but cannot modify]
```

**结对编程**：
```
User: claude --permission-mode default
User: Let's implement the feature together

Claude: [Asks for approval before each change]
```

**自动化任务**：
```
User: claude --permission-mode acceptEdits
User: Fix all linting issues in the codebase

Claude: [Auto-accepts file edits without asking]
```

---

## 无头模式

打印模式（`claude -p`）允许 Claude Code 无需交互式输入即可运行，非常适合自动化和 CI/CD。这是非交互模式，取代了旧的 `--headless` 标志。

### 什么是打印模式？

打印模式启用：
- 自动化脚本执行
- CI/CD 集成
- 批处理
- 定时任务

### 在打印模式下运行（非交互式）

```bash
# Run specific task
claude -p "Run all tests"

# Process piped content
cat error.log | claude -p "Analyze these errors"

# CI/CD integration (GitHub Actions)
- name: AI Code Review
  run: claude -p "Review PR"
```

### 其他打印模式用法示例

```bash
# Run a specific task with output capture
claude -p "Run all tests and generate coverage report"

# With structured output
claude -p --output-format json "Analyze code quality"

# With input from stdin
echo "Analyze code quality" | claude -p "explain this"
```

### 示例：CI/CD 集成

**GitHub Actions**：
```yaml
# .github/workflows/code-review.yml
name: AI Code Review

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Claude Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p --output-format json \
            --max-turns 3 \
            "Review this PR for:
            - Code quality issues
            - Security vulnerabilities
            - Performance concerns
            - Test coverage
            Output results as JSON" > review.json

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = JSON.parse(fs.readFileSync('review.json', 'utf8'));
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: JSON.stringify(review, null, 2)
            });
```

### 打印模式配置

打印模式（`claude -p`）支持多个用于自动化的标志：

```bash
# Limit autonomous turns
claude -p --max-turns 5 "refactor this module"

# Structured JSON output
claude -p --output-format json "analyze this codebase"

# With schema validation
claude -p --json-schema '{"type":"object","properties":{"issues":{"type":"array"}}}' \
  "find bugs in this code"

# Disable session persistence
claude -p --no-session-persistence "one-off analysis"
```

---

## 会话管理

有效地管理多个 Claude Code 会话。

### 会话管理命令

| 命令 | 说明 |
|---------|-------------|
| `/resume` | 按 ID 或名称恢复对话 |
| `/rename` | 命名当前会话 |
| `/fork` | 将当前会话分叉到新分支 |
| `claude -c` | 继续最近的对话 |
| `claude -r "session"` | 按名称或 ID 恢复会话 |

### 恢复会话

**继续最近的对话**：
```bash
claude -c
```

**恢复命名会话**：
```bash
claude -r "auth-refactor" "finish this PR"
```

**重命名当前会话**（在 REPL 内部）：
```
/rename auth-refactor
```

### 分叉会话

分叉会话以在不丢失原始会话的情况下尝试替代方法：

```
/fork
```

或从 CLI：
```bash
claude --resume auth-refactor --fork-session "try OAuth instead"
```

### 会话持久化

会话自动保存，可以恢复：

```bash
# Continue last conversation
claude -c

# Resume specific session by name or ID
claude -r "auth-refactor"

# Resume and fork for experimentation
claude --resume auth-refactor --fork-session "alternative approach"
```

---

## 交互功能

### 键盘快捷键

Claude Code 支持提高效率的键盘快捷键。以下是官方文档的完整参考：

| 快捷键 | 说明 |
|----------|-------------|
| `Ctrl+C` | 取消当前输入/生成 |
| `Ctrl+D` | 退出 Claude Code |
| `Ctrl+G` | 在外部编辑器中编辑计划 |
| `Ctrl+L` | 清除终端屏幕 |
| `Ctrl+O` | 切换详细输出（查看推理） |
| `Ctrl+R` | 反向搜索历史 |
| `Ctrl+T` | 切换任务列表视图 |
| `Ctrl+B` | 后台运行任务 |
| `Esc+Esc` | 回退代码/对话 |
| `Shift+Tab` / `Alt+M` | 切换权限模式 |
| `Option+P` / `Alt+P` | 切换模型 |
| `Option+T` / `Alt+T` | 切换扩展思考 |

**行编辑（标准 readline 快捷键）：**

| 快捷键 | 操作 |
|----------|---------|
| `Ctrl + A` | 移动到行首 |
| `Ctrl + E` | 移动到行尾 |
| `Ctrl + K` | 剪切到行尾 |
| `Ctrl + U` | 剪切到行首 |
| `Ctrl + W` | 向后删除单词 |
| `Ctrl + Y` | 粘贴（yank） |
| `Tab` | 自动完成 |
| `↑ / ↓` | 命令历史 |

### 自定义键绑定

通过运行 `/keybindings` 创建自定义键盘快捷键，这会打开 `~/.claude/keybindings.json` 进行编辑（v2.1.18+）。

**配置格式**：

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null,
        "ctrl+k ctrl+s": "chat:stash"
      }
    },
    {
      "context": "Confirmation",
      "bindings": {
        "ctrl+a": "confirmation:yes"
      }
    }
  ]
}
```

将绑定设置为 `null` 以取消绑定默认快捷键。

### 可用的上下文

键绑定作用域限定为特定的 UI 上下文：

| 上下文 | 关键操作 |
|---------|---------|
| **Chat** | `submit`、`cancel`、`cycleMode`、`modelPicker`、`thinkingToggle`、`undo`、`externalEditor`、`stash`、`imagePaste` |
| **Confirmation** | `yes`、`no`、`previous`、`next`、`nextField`、`cycleMode`、`toggleExplanation` |
| **Global** | `interrupt`、`exit`、`toggleTodos`、`toggleTranscript` |
| **Autocomplete** | `accept`、`dismiss`、`next`、`previous` |
| **HistorySearch** | `search`、`previous`、`next` |
| **Settings** | 上下文特定设置导航 |
| **Tabs** | 选项卡切换和管理 |
| **Help** | 帮助面板导航 |

共有 18 个上下文，包括 `Transcript`、`Task`、`ThemePicker`、`Attachments`、`Footer`、`MessageSelector`、`DiffDialog`、`ModelPicker` 和 `Select`。

### 和弦支持

键绑定支持和弦序列（多键组合）：

```
"ctrl+k ctrl+s"   → Two-key sequence: press ctrl+k, then ctrl+s
"ctrl+shift+p"    → Simultaneous modifier keys
```

**击键语法**：
- **修饰符**：`ctrl`、`alt`（或 `opt`）、`shift`、`meta`（或 `cmd`）
- **大写意味着 Shift**：`K` 等同于 `shift+k`
- **特殊键**：`escape`、`enter`、`return`、`tab`、`space`、`backspace`、`delete`、方向键

### 保留和冲突键

| 键 | 状态 | 备注 |
|-----|--------|-------|
| `Ctrl+C` | 保留 | 无法重新绑定（中断） |
| `Ctrl+D` | 保留 | 无法重新绑定（退出） |
| `Ctrl+B` | 终端冲突 | tmux 前缀键 |
| `Ctrl+A` | 终端冲突 | GNU Screen 前缀键 |
| `Ctrl+Z` | 终端冲突 | 进程挂起 |

> **提示**：如果快捷键不工作，检查是否有与终端模拟器或复用器的冲突。

### Tab 自动完成

Claude Code 提供智能 Tab 自动完成：

```
User: /rew<TAB>
→ /rewind

User: /plu<TAB>
→ /plugin

User: /plugin <TAB>
→ /plugin install
→ /plugin enable
→ /plugin disable
```

### 命令历史

访问之前的命令：

```
User: <↑>  # Previous command
User: <↓>  # Next command
User: Ctrl+R  # Search history

(reverse-i-search)`test': run all tests
```

### 多行输入

对于复杂查询，使用多行模式：

```bash
User: \
> Long complex prompt
> spanning multiple lines
> \end
```

**示例：**

```
User: \
> Implement a user authentication system
> with the following requirements:
> - JWT tokens
> - Email verification
> - Password reset
> - 2FA support
> \end

Claude: [Processes the multi-line request]
```

### 行内编辑

发送前编辑命令：

```
User: Deploy to prodcution<Backspace><Backspace>uction

[Edit in-place before sending]
```

### Vim 模式

启用 Vi/Vim 键绑定进行文本编辑：

**激活**：
- 使用 `/vim` 命令或 `/config` 启用
- 使用 `Esc` 切换到 NORMAL 模式，`i/a/o` 切换到 INSERT 模式

**导航键**：
- `h` / `l` - 左/右移动
- `j` / `k` - 下/上移动
- `w` / `b` / `e` - 按单词移动
- `0` / `$` - 移动到行首/行尾
- `gg` / `G` - 跳转到文本开头/结尾

**文本对象**：
- `iw` / `aw` - 内部/周围单词
- `i"` / `a"` - 内部/周围引号字符串
- `i(` / `a(` - 内部/周围括号

### Bash 模式

使用 `!` 前缀直接执行 shell 命令：

```bash
! npm test
! git status
! cat src/index.js
```

用于快速命令执行而不切换上下文。

---

## 语音输入

语音输入为 Claude Code 提供按键说话语音输入，允许你说话输入提示词而非打字。

### 激活语音输入

```
/voice
```

### 功能

| 功能 | 说明 |
|---------|-------------|
| **按键说话** | 按住一个键录制，松开发送 |
| **20 种语言** | 语音转文本支持 20 种语言 |
| **自定义键绑定** | 通过 `/keybindings` 配置按键说话键 |
| **账户要求** | 需要 Claude.ai 账户进行语音转文本处理 |

### 配置

在键绑定文件（`/keybindings`）中自定义按键说话键绑定。语音输入使用你的 Claude.ai 账户进行语音转文本处理。

---

## Channels

Channels（研究预览）允许 MCP 服务器将消息推送到运行的 Claude Code 会话中，实现与外部服务的实时集成。

### 订阅 Channels

```bash
# Subscribe to channel plugins at startup
claude --channels discord,telegram
```

### 支持的集成

| 集成 | 说明 |
|-------------|-------------|
| **Discord** | 在会话中接收和响应 Discord 消息 |
| **Telegram** | 在会话中接收和响应 Telegram 消息 |

### 配置

**企业部署的托管设置**：

```json
{
  "allowedChannelPlugins": ["discord", "telegram"]
}
```

`allowedChannelPlugins` 托管设置控制整个组织允许哪些 channel 插件。

### 工作原理

1. MCP 服务器充当 channel 插件，连接到外部服务
2. 传入的消息被推送到活动的 Claude Code 会话
3. Claude 可以在会话上下文中读取和响应消息
4. Channel 插件必须通过 `allowedChannelPlugins` 托管设置批准

---

## Chrome 集成

Chrome 集成将 Claude Code 连接到 Chrome 或 Microsoft Edge 浏览器，实现实时网页自动化和调试。这是自 v2.0.73+ 起提供的 beta 功能（Edge 支持在 v1.0.36+ 添加）。

### 启用 Chrome 集成

**启动时**：

```bash
claude --chrome      # Enable Chrome connection
claude --no-chrome   # Disable Chrome connection
```

**在会话中**：

```
/chrome
```

选择"Enabled by default"为所有未来会话激活 Chrome 集成。Claude Code 共享你浏览器的登录状态，因此它可以与已认证的网页应用交互。

### 功能

| 功能 | 说明 |
|------------|-------------|
| **实时调试** | 实时读取控制台日志、检查 DOM 元素、调试 JavaScript |
| **设计验证** | 将渲染页面与设计模型进行比较 |
| **表单验证** | 测试表单提交、输入验证和错误处理 |
| **网页应用测试** | 与已认证应用交互（Gmail、Google Docs、Notion 等） |
| **数据提取** | 从网页抓取和处理内容 |
| **会话录制** | 将浏览器交互录制为 GIF 文件 |

### 站点级权限

Chrome 扩展管理每个站点的访问。随时通过扩展弹出窗口授予或撤销特定站点的访问权限。Claude Code 仅与你明确允许的站点交互。

### 工作原理

Claude Code 在可见窗口中控制浏览器——你可以实时观看操作发生。当浏览器遇到登录页面或 CAPTCHA 时，Claude 会暂停，等待你手动处理，然后继续。

### 已知限制

- **浏览器支持**：仅 Chrome 和 Edge——不支持 Brave、Arc 和其他 Chromium 浏览器
- **WSL**：Windows Subsystem for Linux 中不可用
- **第三方提供商**：不支持 Bedrock、Vertex 或 Foundry API 提供商
- **Service worker 空闲**：Chrome 扩展 service worker 在长时间会话期间可能会空闲

> **提示**：Chrome 集成是一项 beta 功能。浏览器支持可能在未来版本中扩展。

---

## 远程控制

远程控制让你从手机、平板电脑或任何浏览器继续本地运行的 Claude Code 会话。你的本地会话继续在你的机器上运行——没有任何东西移到云端。仅适用于 Pro、Max、Team 和 Enterprise 计划（v2.1.51+）。

### 启动远程控制

**从 CLI**：

```bash
# Start with default session name
claude remote-control

# Start with a custom name
claude remote-control --name "Auth Refactor"
```

**在会话中**：

```
/remote-control
/remote-control "Auth Refactor"
```

**可用标志**：

| 标志 | 说明 |
|------|-------------|
| `--name "title"` | 自定义会话标题以便轻松识别 |
| `--verbose` | 显示详细连接日志 |
| `--sandbox` | 启用文件系统和网络隔离 |
| `--no-sandbox` | 禁用沙盒（默认） |

### 连接到会话

从另一台设备连接的三种方式：

1. **会话 URL** —— 会话启动时打印到终端；在任何浏览器中打开
2. **二维码** —— 启动后按空格键显示可扫描二维码
3. **按名称查找** —— 在 claude.ai/code 或 Claude 移动应用（iOS/Android）中浏览你的会话

### 安全性

- **没有Inbound端口**在你的机器上打开
- **仅出站 HTTPS** over TLS
- **范围限定的凭据** —— 多个短期、窄范围的令牌
- **会话隔离** —— 每个远程会话都是独立的

### 远程控制 vs Web 上的 Claude Code

| 方面 | 远程控制 | Web 上的 Claude Code |
|--------|---------------|-------------------|
| **执行** | 在你的机器上运行 | 在 Anthropic 云上运行 |
| **本地工具** | 完全访问本地 MCP 服务器、文件和 CLI | 无本地依赖 |
| **用例** | 从另一台设备继续本地工作 | 从任何浏览器开始全新会话 |

### 限制

- 每个 Claude Code 实例一个远程会话
- 终端必须在主机上保持打开
- 如果网络不可达，会话在约 10 分钟后超时

### 使用场景

- 在离开办公桌时从移动设备或平板电脑控制 Claude Code
- 在保持本地工具执行的同时使用更丰富的 claude.ai UI
- 用完整的本地开发环境随时进行代码审查

---

## Web 会话

Web 会话允许你直接在 claude.ai/code 的浏览器中运行 Claude Code，或从 CLI 创建 Web 会话。

### 创建 Web 会话

```bash
# Create a new web session from the CLI
claude --remote "implement the new API endpoints"
```

这会启动一个你可以在任何浏览器中访问的 claude.ai 会话。

### 在本地恢复 Web 会话

如果你在 Web 上启动了一个会话并想继续在本地：

```bash
# Resume a web session in the local terminal
claude --teleport
```

或从交互式 REPL 中：

```
/teleport
```

### 使用场景

- 在一台机器上开始工作，在另一台上继续
- 与团队成员共享会话 URL
- 使用 Web UI 进行可视化 diff 审查，然后切换到终端执行

---

## 桌面应用

Claude Code 桌面应用提供带有可视化 diff 审查、并行会话和集成连接器的独立应用程序。仅适用于 macOS 和 Windows（Pro、Max、Team 和 Enterprise 计划）。

### 安装

从 [claude.ai](https://claude.ai) 下载适合你平台的版本：
- **macOS**：通用构建（Apple Silicon 和 Intel）
- **Windows**：x64 和 ARM64 安装程序可用

请参阅[桌面快速入门](https://code.claude.com/docs/en/desktop-quickstart)了解设置说明。

### 从 CLI 交接

将当前 CLI 会话转移到桌面应用：

```
/desktop
```

### 核心功能

| 功能 | 说明 |
|---------|-------------|
| **Diff 视图** | 逐文件可视化审查，带内联评论；Claude 读取评论并修订 |
| **应用预览** | 自动启动开发服务器，带嵌入式浏览器以进行实时验证 |
| **PR 监控** | GitHub CLI 集成，自动修复 CI 失败并在检查通过时自动合并 |
| **并行会话** | 侧边栏中的多个会话，带自动 Git worktree 隔离 |
| **定时任务** | 重复任务（每小时、每天、工作日、每周），在应用打开时运行 |
| **丰富渲染** | 代码、markdown 和图表渲染，带语法高亮 |

### 应用预览配置

在 `.claude/launch.json` 中配置开发服务器行为：

```json
{
  "command": "npm run dev",
  "port": 3000,
  "readyPattern": "ready on",
  "persistCookies": true
}
```

### 连接器

连接外部服务以获得更丰富的上下文：

| 连接器 | 功能 |
|-----------|------------|
| **GitHub** | PR 监控、issue 跟踪、代码审查 |
| **Slack** | 通知、channel 上下文 |
| **Linear** | Issue 跟踪、冲刺管理 |
| **Notion** | 文档、知识库访问 |
| **Asana** | 任务管理、项目跟踪 |
| **日历** | 日程感知、会议上下文 |

> **注意**：连接器不适用于远程（云）会话。

### 远程和 SSH 会话

- **远程会话**：在 Anthropic 云基础设施上运行；关闭应用后继续。可从 claude.ai/code 或 Claude 移动应用访问
- **SSH 会话**：通过 SSH 连接到远程机器，完全访问远程文件系统和工具。远程机器上必须安装 Claude Code

### 桌面中的权限模式

桌面应用支持与 CLI 相同的 4 种权限模式：

| 模式 | 行为 |
|------|----------|
| **Ask permissions**（默认） | 审查和批准每个编辑和命令 |
| **Auto accept edits** | 文件编辑自动批准；命令需要手动批准 |
| **Plan mode** | 在进行任何更改之前审查方法 |
| **Bypass permissions** | 自动执行（仅沙盒、管理员控制） |

### 企业功能

- **管理控制台**：控制组织成员对 Code 选项卡的访问和权限设置
- **MDM 部署**：在 macOS 上通过 MDM 或在 Windows 上通过 MSIX 部署
- **SSO 集成**：要求组织成员进行单点登录
- **托管设置**：集中管理团队配置和模型可用性

---

## 任务列表

任务列表功能提供持久化的任务跟踪，在上下文压缩（对话历史被修剪以适应上下文窗口）时存活。

### 切换任务列表

在会话期间按 `Ctrl+T` 打开或关闭任务列表视图。

### 持久化任务

任务在上下文压缩中持久化，确保长时间工作项在对话上下文被修剪时不会丢失。这对于复杂的多步骤实施特别有用。

### 命名任务目录

使用 `CLAUDE_CODE_TASK_LIST_ID` 环境变量创建跨会话共享的命名任务目录：

```bash
export CLAUDE_CODE_TASK_LIST_ID=my-project-sprint-3
```

这允许多个会话共享相同的任务列表，对团队工作流或多会话项目很有用。

---

## 提示词建议

提示词建议根据你的 git 历史和当前对话上下文显示灰显的示例命令。

### 工作原理

- 建议显示在你输入提示词下方的灰显文本中
- 按 `Tab` 接受建议
- 按 `Enter` 接受并立即提交
- 建议是上下文感知的，从 git 历史和对话状态中提取

### 禁用提示词建议

```bash
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
```

---

## Git Worktrees

Git Worktrees 允许你在隔离的 worktree 中启动 Claude Code，启用在不同分支上并行工作，而无需 stash 或切换。

### 在 Worktree 中启动

```bash
# Start Claude Code in an isolated worktree
claude --worktree
# or
claude -w
```

### Worktree 位置

Worktree 创建在：
```
<repo>/.claude/worktrees/<name>
```

### Monorepo 的稀疏检出

使用 `worktree.sparsePaths` 设置在 monorepo 中执行稀疏检出，减少磁盘使用和克隆时间：

```json
{
  "worktree": {
    "sparsePaths": ["packages/my-package", "shared/"]
  }
}
```

### Worktree 工具和钩子

| 项目 | 说明 |
|------|-------------|
| `ExitWorktree` | 退出和清理当前 worktree 的工具 |
| `WorktreeCreate` | 创建 worktree 时触发的事件钩子 |
| `WorktreeRemove` | 移除 worktree 时触发的事件钩子 |

### 自动清理

如果在 worktree 中没有进行任何更改，它会在会话结束时自动清理。

### 使用场景

- 在保持 main 分支不变的情况下处理功能分支
- 在不影响工作目录的情况下隔离运行测试
- 在一次性环境中尝试实验性更改
- 在 monorepo 中稀疏检出特定包以加快启动速度

---

## 沙盒

沙盒为 Claude Code 执行的 Bash 命令提供操作系统级文件系统和网络隔离。这是对权限规则的补充，并提供额外的安全层。

### 启用沙盒

**斜杠命令**：
```
/sandbox
```

**CLI 标志**：
```bash
claude --sandbox       # Enable sandboxing
claude --no-sandbox    # Disable sandboxing
```

### 配置设置

| 设置 | 说明 |
|---------|-------------|
| `sandbox.enabled` | 启用或禁用沙盒 |
| `sandbox.failIfUnavailable` | 如果无法激活沙盒则失败 |
| `sandbox.filesystem.allowWrite` | 允许写入访问的路径 |
| `sandbox.filesystem.allowRead` | 允许读取访问的路径 |
| `sandbox.filesystem.denyRead` | 拒绝读取访问的路径 |
| `sandbox.enableWeakerNetworkIsolation` | 在 macOS 上启用较弱网络隔离 |

### 配置示例

```json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "filesystem": {
      "allowWrite": ["/Users/me/project"],
      "allowRead": ["/Users/me/project", "/usr/local/lib"],
      "denyRead": ["/Users/me/.ssh", "/Users/me/.aws"]
    },
    "enableWeakerNetworkIsolation": true
  }
}
```

### 工作原理

- Bash 命令在具有受限文件系统访问的沙盒环境中运行
- 网络访问可以隔离以防止意外的外部连接
- 与权限规则一起用于纵深防御
- 在 macOS 上，使用 `sandbox.enableWeakerNetworkIsolation` 进行网络限制（macOS 上无法使用完整网络隔离）

### 使用场景

- 安全地运行不受信任或生成的代码
- 防止意外修改项目外的文件
- 在自动化任务期间限制网络访问

---

## 托管设置（企业版）

托管设置使企业管理员能够使用平台原生管理工具跨组织部署 Claude Code 配置。

### 部署方法

| 平台 | 方法 | 自 |
|----------|--------|-------|
| macOS | 托管 plist 文件（MDM） | v2.1.51+ |
| Windows | Windows 注册表 | v2.1.51+ |
| 跨平台 | 托管配置文件 | v2.1.51+ |
| 跨平台 | 托管 drop-in（`managed-settings.d/` 目录） | v2.1.83+ |

### 托管 Drop-in

自 v2.1.83 起，管理员可以将多个托管设置文件部署到 `managed-settings.d/` 目录。文件按字母顺序合并，允许跨团队模块化配置：

```
~/.claude/managed-settings.d/
  00-org-defaults.json
  10-team-policies.json
  20-project-overrides.json
```

### 可用的托管设置

| 设置 | 说明 |
|---------|-------------|
| `disableBypassPermissionsMode` | 防止用户启用绕过权限 |
| `availableModels` | 限制用户可选择的模型 |
| `allowedChannelPlugins` | 控制允许的 channel 插件 |
| `autoMode.environment` | 为 auto 模式配置受信任的基础设施 |
| 自定义策略 | 组织特定的权限和工具策略 |

### 示例：macOS Plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>disableBypassPermissionsMode</key>
  <true/>
  <key>availableModels</key>
  <array>
    <string>claude-sonnet-4-6</string>
    <string>claude-haiku-4-5</string>
  </array>
</dict>
</plist>
```

---

## 配置和设置

### 配置文件位置

1. **全局配置**：`~/.claude/config.json`
2. **项目配置**：`./.claude/config.json`
3. **用户配置**：`~/.config/claude-code/settings.json`

### 完整配置示例

**核心高级功能配置：**

```json
{
  "permissions": {
    "mode": "default"
  },
  "hooks": {
    "PreToolUse:Edit": "eslint --fix ${file_path}",
    "PostToolUse:Write": "~/.claude/hooks/security-scan.sh"
  },
  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"]
      }
    }
  }
}
```

**扩展配置示例：**

```json
{
  "permissions": {
    "mode": "default",
    "allowedTools": ["Bash(git log:*)", "Read"],
    "disallowedTools": ["Bash(rm -rf:*)"]
  },

  "hooks": {
    "PreToolUse": [{ "matcher": "Edit", "hooks": ["eslint --fix ${file_path}"] }],
    "PostToolUse": [{ "matcher": "Write", "hooks": ["~/.claude/hooks/security-scan.sh"] }],
    "Stop": [{ "hooks": ["~/.claude/hooks/notify.sh"] }]
  },

  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      }
    }
  }
}
```

### 环境变量

使用环境变量覆盖配置：

```bash
# Model selection
export ANTHROPIC_MODEL=claude-opus-4-6
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-6
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-6
export ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-5

# API configuration
export ANTHROPIC_API_KEY=sk-ant-...

# Thinking configuration
export MAX_THINKING_TOKENS=16000
export CLAUDE_CODE_EFFORT_LEVEL=high

# Feature toggles
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=true
export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=true
export CLAUDE_CODE_DISABLE_CRON=1
export CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS=true
export CLAUDE_CODE_DISABLE_TERMINAL_TITLE=true
export CLAUDE_CODE_DISABLE_1M_CONTEXT=true
export CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK=true
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
export CLAUDE_CODE_ENABLE_TASKS=true
export CLAUDE_CODE_SIMPLE=true              # Set by --bare flag

# MCP configuration
export MAX_MCP_OUTPUT_TOKENS=50000
export ENABLE_TOOL_SEARCH=true

# Task management
export CLAUDE_CODE_TASK_LIST_ID=my-project-tasks

# Agent teams (experimental)
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=true

# Subagent and plugin configuration
export CLAUDE_CODE_SUBAGENT_MODEL=sonnet
export CLAUDE_CODE_PLUGIN_SEED_DIR=./my-plugins
export CLAUDE_CODE_NEW_INIT=true

# Subprocess and streaming
export CLAUDE_CODE_SUBPROCESS_ENV_SCRUB="SECRET_KEY,DB_PASSWORD"
export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=80
export CLAUDE_STREAM_IDLE_TIMEOUT_MS=30000
export ANTHROPIC_CUSTOM_MODEL_OPTION=my-custom-model
export SLASH_COMMAND_TOOL_CHAR_BUDGET=50000
```

### 配置管理命令

```
User: /config
[Opens interactive configuration menu]
```

`/config` 命令提供一个交互式菜单来切换设置，例如：
- 扩展思考开/关
- 详细输出
- 权限模式
- 模型选择

### 按项目配置

在你的项目中创建 `.claude/config.json`：

```json
{
  "hooks": {
    "PreToolUse": [{ "matcher": "Bash", "hooks": ["npm test && npm run lint"] }]
  },
  "permissions": {
    "mode": "default"
  },
  "mcp": {
    "servers": {
      "project-db": {
        "command": "mcp-postgres",
        "env": {
          "DATABASE_URL": "${PROJECT_DB_URL}"
        }
      }
    }
  }
}
```

---

## 最佳实践

### 规划模式
- ✅ 用于复杂的多步骤任务
- ✅ 审批前审查计划
- ✅ 需要时修改计划
- ❌ 不要用于简单任务

### 扩展思考
- ✅ 用于架构决策
- ✅ 用于复杂问题解决
- ✅ 审查思考过程
- ❌ 不要用于简单查询

### 后台任务
- ✅ 用于长时间运行的操作
- ✅ 监控任务进度
- ✅ 优雅处理任务失败
- ❌ 不要启动太多并发任务

### 权限
- ✅ 使用 `plan` 进行代码审查（只读）
- ✅ 使用 `default` 进行交互式开发
- ✅ 使用 `acceptEdits` 进行自动化工作流
- ✅ 使用 `auto` 进行带安全防护的自主工作
- ❌ 除非绝对必要，否则不要使用 `bypassPermissions`

### 会话
- ✅ 为不同任务使用单独的会话
- ✅ 保存重要的会话状态
- ✅ 清理旧会话
- ❌ 不要在单个会话中混合不相关的工作

---

## 其他资源

有关 Claude Code 和相关功能的更多信息：

- [Official Interactive Mode Documentation](https://code.claude.com/docs/en/interactive-mode)
- [Official Headless Mode Documentation](https://code.claude.com/docs/en/headless)
- [CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Checkpoints Guide](../08-checkpoints/) - 会话管理和回退
- [Slash Commands](../01-slash-commands/) - 命令参考
- [Memory Guide](../02-memory/) - 持久化上下文
- [Skills Guide](../03-skills/) - 自主能力
- [Subagents Guide](../04-subagents/) - 委托任务执行
- [MCP Guide](../05-mcp/) - 外部数据访问
- [Hooks Guide](../06-hooks/) - 事件驱动自动化
- [Plugins Guide](../07-plugins/) - 捆绑扩展
- [Official Scheduled Tasks Documentation](https://code.claude.com/docs/en/scheduled-tasks)
- [Official Chrome Integration Documentation](https://code.claude.com/docs/en/chrome)
- [Official Remote Control Documentation](https://code.claude.com/docs/en/remote-control)
- [Official Keybindings Documentation](https://code.claude.com/docs/en/keybindings)
- [Official Desktop App Documentation](https://code.claude.com/docs/en/desktop)
