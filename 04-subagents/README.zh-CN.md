<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 子代理 - 完整参考指南

子代理是专门的 AI 助手，Claude Code 可以将任务委托给它们。每个子代理有特定目的，使用与主对话分离的自己的上下文窗口，并可配置特定工具和自定义系统提示词。

## 目录

1. [概述](#概述)
2. [关键优势](#关键优势)
3. [文件位置](#文件位置)
4. [配置](#配置)
5. [内置子代理](#内置子代理)
6. [管理子代理](#管理子代理)
7. [使用子代理](#使用子代理)
8. [可恢复代理](#可恢复代理)
9. [链接子代理](#链接子代理)
10. [子代理的持久内存](#子代理的持久内存)
11. [后台子代理](#后台子代理)
12. [工作树隔离](#工作树隔离)
13. [限制可生成的子代理](#限制可生成的子代理)
14. [`claude agents` CLI 命令](#claude-agents-cli-命令)
15. [代理团队（实验性）](#代理团队实验性)
16. [插件子代理安全](#插件子代理安全)
17. [架构](#架构)
18. [上下文管理](#上下文管理)
19. [何时使用子代理](#何时使用子代理)
20. [最佳实践](#最佳实践)
21. [本文件夹中的示例子代理](#本文件夹中的示例子代理)
22. [安装说明](#安装说明)
23. [相关概念](#相关概念)

---

## 概述

子代理通过以下方式实现 Claude Code 中的委托任务执行：

- 创建**隔离的 AI 助手**，拥有独立的上下文窗口
- 提供**定制的系统提示词**，以获得专门专业知识
- 强制**工具访问控制**，限制能力
- 防止复杂任务的**上下文污染**
- 实现多个专门任务的**并行执行**

每个子代理独立运作，从干净的状态开始，只接收其任务所需的特定上下文，然后将结果返回给主代理进行综合。

**快速开始**：使用 `/agents` 命令交互式创建、查看、编辑和管理您的子代理。

---

## 关键优势

| 优势 | 描述 |
|---------|-------------|
| **上下文保持** | 在独立上下文中运作，防止主对话污染 |
| **专门专业知识** | 针对特定领域微调，成功率更高 |
| **可复用性** | 跨不同项目使用并与团队共享 |
| **灵活权限** | 不同子代理类型有不同的工具访问级别 |
| **可扩展性** | 多个代理同时处理不同方面 |

---

## 文件位置

子代理文件可存储在多个位置，具有不同范围：

| 优先级 | 类型 | 位置 | 范围 |
|----------|------|----------|-------|
| 1（最高） | **CLI 定义** | 通过 `--agents` 标志（JSON） | 仅会话 |
| 2 | **项目子代理** | `.claude/agents/` | 当前项目 |
| 3 | **用户子代理** | `~/.claude/agents/` | 所有项目 |
| 4（最低） | **插件代理** | 插件 `agents/` 目录 | 通过插件 |

当存在重复名称时，优先级高的来源优先。

---

## 配置

### 文件格式

子代理在 YAML 前置matter中定义，后跟 markdown 中的系统提示词：

```yaml
---
name: your-sub-agent-name
description: 描述何时应调用此子代理
tools: tool1, tool2, tool3  # 可选 - 省略则继承所有工具
disallowedTools: tool4  # 可选 - 明确禁止的工具
model: sonnet  # 可选 - sonnet、opus、haiku 或 inherit
permissionMode: default  # 可选 - 权限模式
maxTurns: 20  # 可选 - 限制代理轮次
skills: skill1, skill2  # 可选 - 预加载到上下文的技能
mcpServers: server1  # 可选 - 可用的 MCP 服务器
memory: user  # 可选 - 持久内存范围（user、project、local）
background: false  # 可选 - 作为后台任务运行
effort: high  # 可选 - 推理努力程度（low、medium、high、max）
isolation: worktree  # 可选 - git worktree 隔离
initialPrompt: "首先分析代码库"  # 可选 - 子代理运行时的自动提交第一轮
hooks:  # 可选 - 组件作用域钩子
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---

您的子代理系统提示词在此处。这可以是多个段落，
应清楚定义子代理的角色、能力，
以及解决问题的方法。
```

### 配置字段

| 字段 | 必需 | 描述 |
|-------|----------|-------------|
| `name` | 是 | 唯一标识符（小写字母和连字符） |
| `description` | 是 | 目的的自然语言描述。包含"use PROACTIVELY"以鼓励自动调用 |
| `tools` | 否 | 特定工具的逗号分隔列表。省略以继承所有工具。支持 `Agent(agent_name)` 语法限制可生成的子代理 |
| `disallowedTools` | 否 | 子代理不能使用的工具的逗号分隔列表 |
| `model` | 否 | 要使用的模型：`sonnet`、`opus`、`haiku`、完整模型 ID 或 `inherit`。默认为配置的子代理模型 |
| `permissionMode` | 否 | `default`、`acceptEdits`、`dontAsk`、`bypassPermissions`、`plan` |
| `maxTurns` | 否 | 子代理可采取的最大代理轮次数 |
| `skills` | 否 | 预加载的技能逗号分隔列表。将完整技能内容注入到子代理启动时的上下文中 |
| `mcpServers` | 否 | 可供子代理使用的 MCP 服务器 |
| `hooks` | 否 | 组件作用域钩子（PreToolUse、PostToolUse、Stop） |
| `memory` | 否 | 持久内存目录范围：`user`、`project` 或 `local` |
| `background` | 否 | 设为 `true` 始终将此子代理作为后台任务运行 |
| `effort` | 否 | 推理努力级别：`low`、`medium`、`high` 或 `max` |
| `isolation` | 否 | 设为 `worktree` 为子代理提供自己的 git worktree |
| `initialPrompt` | 否 | 子代理作为主代理运行时的自动提交第一轮 |

### 工具配置选项

**选项 1：继承所有工具（省略字段）**
```yaml
---
name: full-access-agent
description: 具有所有可用工具的代理
---
```

**选项 2：指定单个工具**
```yaml
---
name: limited-agent
description: 仅具有特定工具的代理
tools: Read, Grep, Glob, Bash
---
```

**选项 3：条件工具访问**
```yaml
---
name: conditional-agent
description: 具有过滤工具访问的代理
tools: Read, Bash(npm:*), Bash(test:*)
---
```

### 基于 CLI 的配置

使用 `--agents` 标志以 JSON 格式为单个会话定义子代理：

```bash
claude --agents '{
  "code-reviewer": {
    "description": "专业代码审查员。在代码变更后主动使用。",
    "prompt": "你是一位资深代码审查员。专注于代码质量、安全性和最佳实践。",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

**`--agents` 标志的 JSON 格式：**

```json
{
  "agent-name": {
    "description": "必需：何时调用此代理",
    "prompt": "必需：代理的系统提示词",
    "tools": ["可选", "工具", "数组"],
    "model": "可选：sonnet|opus|haiku"
  }
}
```

**代理定义优先级：**

代理定义按此优先级顺序加载（首次匹配优先）：
1. **CLI 定义** - `--agents` 标志（仅会话，JSON）
2. **项目级** - `.claude/agents/`（当前项目）
3. **用户级** - `~/.claude/agents/`（所有项目）
4. **插件级** - 插件 `agents/` 目录

这允许 CLI 定义为单个会话覆盖所有其他来源。

---

## 内置子代理

Claude Code 包含多个始终可用的内置子代理：

| 代理 | 模型 | 用途 |
|-------|-------|---------|
| **general-purpose** | 继承 | 复杂、多步骤任务 |
| **Plan** | 继承 | 计划模式研究 |
| **Explore** | Haiku | 只读代码库探索（快速/中等/非常彻底） |
| **Bash** | 继承 | 在独立上下文中运行终端命令 |
| **statusline-setup** | Sonnet | 配置状态行显示 |
| **Claude Code Guide** | Haiku | 回答 Claude Code 功能问题 |

### General-Purpose 子代理

| 属性 | 值 |
|----------|-------|
| **模型** | 继承自父代理 |
| **工具** | 所有工具 |
| **用途** | 复杂研究任务、多步骤操作、代码修改 |

**使用场景**：需要探索和修改以及复杂推理的任务。

### Plan 子代理

| 属性 | 值 |
|----------|-------|
| **模型** | 继承自父代理 |
| **工具** | Read、Glob、Grep、Bash |
| **用途** | 在计划模式中自动用于研究代码库 |

**使用场景**：当 Claude 需要在提出计划之前理解代码库时。

### Explore 子代理

| 属性 | 值 |
|----------|-------|
| **模型** | Haiku（快速、低延迟） |
| **模式** | 严格只读 |
| **工具** | Glob、Grep、Read、Bash（仅只读命令） |
| **用途** | 快速代码库搜索和分析 |

**使用场景**：在不进行更改时搜索/理解代码。

**彻底程度级别** - 指定探索深度：
- **"quick"** - 快速搜索，最小探索，适合查找特定模式
- **"medium"** - 中等探索，平衡速度和彻底性，默认方法
- **"very thorough"** - 跨多个位置和命名约定的全面分析，可能需要更长时间

### Bash 子代理

| 属性 | 值 |
|----------|-------|
| **模型** | 继承自父代理 |
| **工具** | Bash |
| **用途** | 在独立的上下文窗口中执行终端命令 |

**使用场景**：当运行的 shell 命令受益于隔离上下文时。

### Statusline Setup 子代理

| 属性 | 值 |
|----------|-------|
| **模型** | Sonnet |
| **工具** | Read、Write、Bash |
| **用途** | 配置 Claude Code 状态行显示 |

**使用场景**：设置或自定义状态行时。

### Claude Code Guide 子代理

| 属性 | 值 |
|----------|-------|
| **模型** | Haiku（快速、低延迟） |
| **工具** | 只读 |
| **用途** | 回答有关 Claude Code 功能和用法的问题 |

**使用场景**：当用户询问 Claude Code 如何工作或如何使用特定功能时。

---

## 管理子代理

### 使用 `/agents` 命令（推荐）

```bash
/agents
```

这提供交互式菜单：
- 查看所有可用子代理（内置、用户和项目）
- 通过引导设置创建新子代理
- 编辑现有自定义子代理和工具访问
- 删除自定义子代理
- 查看存在重复时哪些子代理处于活动状态

### 直接文件管理

```bash
# 创建项目子代理
mkdir -p .claude/agents
cat > .claude/agents/test-runner.md << 'EOF'
---
name: test-runner
description: 主动使用运行测试并修复失败
---

你是一位测试自动化专家。当您看到代码变更时，主动运行适当的测试。
如果测试失败，分析失败并修复，同时保留原始测试意图。
EOF

# 创建用户子代理（在所有项目中可用）
mkdir -p ~/.claude/agents
```

---

## 使用子代理

### 自动委托

Claude 根据以下内容主动委托任务：
- 任务描述中的您的请求
- 子代理配置中的 `description` 字段
- 当前上下文和可用工具

为鼓励主动使用，在 `description` 字段中包含"use PROACTIVELY"或"MUST BE USED"：

```yaml
---
name: code-reviewer
description: 专业代码审查专家。在编写或修改代码后主动使用。
---
```

### 显式调用

您可以显式请求特定子代理：

```
> 使用 test-runner 子代理修复失败的测试
> 让 code-reviewer 子代理查看我最近的变更
> 让 debugger 子代理调查此错误
```

### @-Mention 调用

使用 `@` 前缀保证调用特定子代理（绕过自动委托启发式）：

```
> @"code-reviewer (代理)" 审查 auth 模块
```

### 会话级代理

使用特定代理作为主代理运行整个会话：

```bash
# 通过 CLI 标志
claude --agent code-reviewer

# 通过 settings.json
{
  "agent": "code-reviewer"
}
```

### 列出可用代理

使用 `claude agents` 命令列出所有配置来源的代理：

```bash
claude agents
```

---

## 可恢复代理

子代理可以继续之前的对话，保留完整上下文：

```bash
# 初始调用
> 使用 code-analyzer 代理开始审查身份验证模块
# 返回 agentId: "abc123"

# 稍后恢复代理
> 恢复代理 abc123，现在也分析授权逻辑
```

**使用场景**：
- 跨多个会话的长期研究
- 不丢失上下文的迭代改进
- 保持上下文的多步骤工作流程

---

## 链接子代理

按顺序执行多个子代理：

```bash
> 首先使用 code-analyzer 子代理查找性能问题，
  然后使用 optimizer 子代理修复它们
```

这支持复杂工作流程，其中一个子代理的输出作为另一个的输入。

---

## 子代理的持久内存

`memory` 字段为子代理提供跨对话持久化的目录。这允许子代理随着时间积累知识，存储在会话之间持久化的笔记、发现和上下文。

### 内存范围

| 范围 | 目录 | 使用场景 |
|-------|-----------|----------|
| `user` | `~/.claude/agent-memory/<name>/` | 跨所有项目的个人笔记和偏好 |
| `project` | `.claude/agent-memory/<name>/` | 与团队共享的项目特定知识 |
| `local` | `.claude/agent-memory-local/<name>/` | 不提交到版本控制的本地项目知识 |

### 工作原理

- 内存目录中 `MEMORY.md` 的前 200 行自动加载到子代理的系统提示词中
- `Read`、`Write` 和 `Edit` 工具自动为子代理启用以管理其内存文件
- 子代理可以根据需要在其内存目录中创建额外文件

### 配置示例

```yaml
---
name: researcher
memory: user
---

你是一位研究助理。使用你的内存目录存储发现、
跨会话跟踪进度，并随着时间积累知识。

在每个会话开始时检查你的 MEMORY.md 文件以回忆之前的上下文。
```

```mermaid
graph LR
    A["子代理<br/>会话 1"] -->|写入| M["MEMORY.md<br/>(持久化)"]
    M -->|加载到| B["子代理<br/>会话 2"]
    B -->|更新| M
    M -->|加载到| C["子代理<br/>会话 3"]

    style A fill:#e1f5fe,stroke:#333,color:#333
    style B fill:#e1f5fe,stroke:#333,color:#333
    style C fill:#e1f5fe,stroke:#333,color:#333
    style M fill:#f3e5f5,stroke:#333,color:#333
```

---

## 后台子代理

子代理可以在后台运行，释放主对话处理其他任务。

### 配置

在前置matter中设置 `background: true` 始终将子代理作为后台任务运行：

```yaml
---
name: long-runner
background: true
description: 在后台执行长期运行的分析任务
---
```

### 键盘快捷键

| 快捷键 | 操作 |
|----------|--------|
| `Ctrl+B` | 将当前运行的子代理任务放到后台 |
| `Ctrl+F` | 终止所有后台代理（按两次确认） |

### 禁用后台任务

设置环境变量完全禁用后台任务支持：

```bash
export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1
```

---

## 工作树隔离

`isolation: worktree` 设置为子代理提供自己的 git worktree，允许它独立进行更改而不影响主工作树。

### 配置

```yaml
---
name: feature-builder
isolation: worktree
description: 在隔离的 git worktree 中实现功能
tools: Read, Write, Edit, Bash, Grep, Glob
---
```

### 工作原理

```mermaid
graph TB
    Main["主工作树"] -->|生成| Sub["带隔离 Worktree 的子代理"]
    Sub -->|在...中进行更改| WT["独立的 Git<br/>Worktree + 分支"]
    WT -->|无更改| Clean["自动清理"]
    WT -->|有更改| Return["返回 worktree<br/>路径和分支"]

    style Main fill:#e1f5fe,stroke:#333,color:#333
    style Sub fill:#f3e5f5,stroke:#333,color:#333
    style WT fill:#e8f5e9,stroke:#333,color:#333
    style Clean fill:#fff3e0,stroke:#333,color:#333
    style Return fill:#fff3e0,stroke:#333,color:#333
```

- 子代理在自己的 git worktree 中的独立分支上运作
- 如果子代理未进行任何更改，worktree 自动清理
- 如果存在更改，worktree 路径和分支名称返回给主代理审查或合并

---

## 限制可生成的子代理

您可以使用 `tools` 字段中的 `Agent(agent_type)` 语法控制给定子代理允许生成哪些子代理。这提供了一种方式来允许列表特定的子代理进行委托。

> **注意**：在 v2.1.63 中，`Task` 工具被重命名为 `Agent`。现有的 `Task(...)` 引用仍然作为别名工作。

### 示例

```yaml
---
name: coordinator
description: 协调专门代理之间的工作
tools: Agent(worker, researcher), Read, Bash
---

你是一个协调器代理。你只能委托给"worker"和"researcher"子代理。
使用 Read 和 Bash 进行你自己的探索。
```

在此示例中，`coordinator` 子代理只能生成 `worker` 和 `researcher` 子代理。它不能生成任何其他子代理，即使它们在其他地方定义。

---

## `claude agents` CLI 命令

`claude agents` 命令列出所有配置来源的代理，按来源分组（内置、用户级、项目级）：

```bash
claude agents
```

此命令：
- 显示来自所有来源的所有可用代理
- 按其来源位置对代理分组
- 指示当较高优先级级别的代理遮盖较低级别的代理时的**覆盖**（例如，与用户级代理同名项目级代理）

---

## 代理团队（实验性）

代理团队协调多个 Claude Code 实例共同处理复杂任务。与子代理（委托返回结果的子任务）不同，团队成员独立运作，拥有自己的上下文并通过共享邮箱系统直接通信。

> **注意**：代理团队是实验性功能，需要 Claude Code v2.1.32+。使用前需启用。

### 子代理 vs 代理团队

| 方面 | 子代理 | 代理团队 |
|--------|-----------|-------------|
| **委托模型** | 父代理委托子任务，等待结果 | 团队负责人分配工作，团队成员独立执行 |
| **上下文** | 每个子任务全新上下文，结果提炼回来 | 每个团队成员维护自己的持久上下文 |
| **协调** | 顺序或并行，由父代理管理 | 共享任务列表，自动依赖管理 |
| **通信** | 仅返回值 | 通过邮箱的代理间消息 |
| **会话恢复** | 支持 | 不支持使用进程内团队成员 |
| **适用场景** | 聚焦、定义明确的子任务 | 需要并行工作的大型多文件项目 |

### 启用代理团队

设置环境变量或添加到 `settings.json`：

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

或在 `settings.json` 中：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 启动团队

启用后，在提示词中要求 Claude 与团队成员合作：

```
用户：构建身份验证模块。使用团队——一个成员负责 API 端点，
      一个负责数据库 schema，一个负责测试套件。
```

Claude 将创建团队、分配任务并自动协调工作。

### 显示模式

控制团队成员活动的显示方式：

| 模式 | 标志 | 描述 |
|------|------|-------------|
| **自动** | `--teammate-mode auto` | 自动为您的终端选择最佳显示模式 |
| **进程中** | `--teammate-mode in-process` | 在当前终端内联显示团队成员输出（默认） |
| **分窗格** | `--teammate-mode tmux` | 在单独的 tmux 或 iTerm2 窗格中打开每个团队成员 |

```bash
claude --teammate-mode tmux
```

您也可以在 `settings.json` 中设置显示模式：

```json
{
  "teammateMode": "tmux"
}
```

> **注意**：分窗格模式需要 tmux 或 iTerm2。在 VS Code 终端、Windows Terminal 或 Ghostty 中不可用。

### 导航

在分窗格模式中使用 `Shift+Down` 在团队成员之间导航。

### 团队配置

团队配置存储在 `~/.claude/teams/{team-name}/config.json`。

### 架构

```mermaid
graph TB
    Lead["团队负责人<br/>(协调器)"]
    TaskList["共享任务列表<br/>(依赖关系)"]
    Mailbox["邮箱<br/>(消息)"]
    T1["团队成员 1<br/>(自己的上下文)"]
    T2["团队成员 2<br/>(自己的上下文)"]
    T3["团队成员 3<br/>(自己的上下文)"]

    Lead -->|分配任务| TaskList
    Lead -->|发送消息| Mailbox
    TaskList -->|接取工作| T1
    TaskList -->|接取工作| T2
    TaskList -->|接取工作| T3
    T1 -->|读取/写入| Mailbox
    T2 -->|读取/写入| Mailbox
    T3 -->|读取/写入| Mailbox
    T1 -->|更新状态| TaskList
    T2 -->|更新状态| TaskList
    T3 -->|更新状态| TaskList

    style Lead fill:#e1f5fe,stroke:#333,color:#333
    style TaskList fill:#fff9c4,stroke:#333,color:#333
    style Mailbox fill:#f3e5f5,stroke:#333,color:#333
    style T1 fill:#e8f5e9,stroke:#333,color:#333
    style T2 fill:#e8f5e9,stroke:#333,color:#333
    style T3 fill:#e8f5e9,stroke:#333,color:#333
```

**关键组件**：

- **团队负责人**：创建团队、分配任务和协调的主 Claude Code 会话
- **共享任务列表**：具有自动依赖跟踪的同步任务列表
- **邮箱**：团队成员通过其通信状态和协调的代理间消息系统
- **团队成员**：每个拥有自己上下文窗口的独立 Claude Code 实例

### 任务分配和消息传递

团队负责人将工作分解为任务并分配给团队成员。共享任务列表处理：

- **自动依赖管理** — 任务等待其依赖完成
- **状态跟踪** — 团队成员更新任务状态
- **代理间消息** — 团队成员通过邮箱发送消息进行协调（例如，"数据库 schema 已就绪，您可以开始编写查询"）

### 计划审批工作流程

对于复杂任务，团队负责人在团队成员开始工作之前创建执行计划。用户审查并批准计划，确保团队的方法在开始任何代码更改之前符合预期。

### 团队钩子事件

代理团队引入两个额外的 [钩子事件](../06-hooks/)：

| 事件 | 触发时机 | 使用场景 |
|-------|-----------|-----------|
| `TeammateIdle` | 团队成员完成当前任务且无待处理工作时 | 触发通知，分配后续任务 |
| `TaskCompleted` | 共享任务列表中的任务标记为完成时 | 运行验证、更新仪表板、链接依赖工作 |

### 最佳实践

- **团队规模**：保持 3-5 名团队成员以获得最佳协调
- **任务大小**：分解为每个 5-15 分钟的任务——小到可以并行化，大到有意义
- **避免文件冲突**：为不同团队成员分配不同文件或目录以防止合并冲突
- **从简单开始**：首次使用进程内模式；熟悉后再切换到分窗格
- **清晰的任务描述**：提供具体、可操作的任务描述，以便团队成员独立工作

### 限制

- **实验性**：功能行为可能在未来版本中更改
- **无会话恢复**：进程内团队成员在会话结束后无法恢复
- **每个会话一个团队**：无法在单个会话中创建嵌套团队或多个团队
- **固定领导**：团队负责人角色无法转移到团队成员
- **分窗格限制**：需要 tmux/iTerm2；在 VS Code 终端、Windows Terminal 或 Ghostty 中不可用
- **无跨会话团队**：团队成员仅在当前会话内存在

> **警告**：代理团队是实验性功能。首先用非关键工作测试，并监控团队协调是否有意外行为。

---

## 插件子代理安全

插件提供的子代理出于安全原因具有受限的前置matter功能。以下字段在插件子代理定义中**不允许**：

- `hooks` - 不能定义生命周期钩子
- `mcpServers` - 不能配置 MCP 服务器
- `permissionMode` - 不能覆盖权限设置

这防止插件通过子代理钩子提升权限或执行任意命令。

---

## 架构

### 高层架构

```mermaid
graph TB
    User["用户"]
    Main["主代理<br/>(协调器)"]
    Reviewer["代码审查员<br/>子代理"]
    Tester["测试工程师<br/>子代理"]
    Docs["文档<br/>子代理"]

    User -->|请求| Main
    Main -->|委托| Reviewer
    Main -->|委托| Tester
    Main -->|委托| Docs
    Reviewer -->|返回结果| Main
    Tester -->|返回结果| Main
    Docs -->|返回结果| Main
    Main -->|综合| User
```

### 子代理生命周期

```mermaid
sequenceDiagram
    participant User
    participant MainAgent as 主代理
    participant CodeReviewer as 代码审查员<br/>子代理
    participant Context as 独立<br/>上下文窗口

    User->>MainAgent: "构建新的 auth 功能"
    MainAgent->>MainAgent: 分析任务
    MainAgent->>CodeReviewer: "审查此代码"
    CodeReviewer->>Context: 初始化干净上下文
    Context->>CodeReviewer: 加载审查员指令
    CodeReviewer->>CodeReviewer: 执行审查
    CodeReviewer-->>MainAgent: 返回发现
    MainAgent->>MainAgent: 合并结果
    MainAgent-->>User: 提供综合结果
```

---

## 上下文管理

```mermaid
graph TB
    A["主代理上下文<br/>50,000 tokens"]
    B["子代理 1 上下文<br/>20,000 tokens"]
    C["子代理 2 上下文<br/>20,000 tokens"]
    D["子代理 3 上下文<br/>20,000 tokens"]

    A -->|干净状态| B
    A -->|干净状态| C
    A -->|干净状态| D

    B -->|仅结果| A
    C -->|仅结果| A
    D -->|仅结果| A

    style A fill:#e1f5fe
    style B fill:#fff9c4
    style C fill:#fff9c4
    style D fill:#fff9c4
```

### 关键点

- 每个子代理获得**全新的上下文窗口**，没有主对话历史
- 只有**相关上下文**传递给子代理处理其特定任务
- 结果**提炼**回主代理
- 这防止长时间项目中的**上下文 token 耗尽**

### 性能考虑

- **上下文效率** - 代理保留主上下文，实现更长会话
- **延迟** - 子代理从干净状态开始，收集初始上下文可能增加延迟

### 关键行为

- **无嵌套生成** - 子代理不能生成其他子代理
- **后台权限** - 后台子代理自动拒绝任何未预先批准的权限
- **后台化** - 按 `Ctrl+B` 将当前运行的任务放到后台
- **转录** - 子代理转录存储在 `~/.claude/projects/{project}/{sessionId}/subagents/agent-{agentId}.jsonl`
- **自动压缩** - 子代理上下文在约 95% 容量时自动压缩（使用 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 环境变量覆盖）

---

## 何时使用子代理

| 场景 | 使用子代理 | 原因 |
|----------|--------------|-------|
| 复杂功能，步骤多 | 是 | 分离关注点，防止上下文污染 |
| 快速代码审查 | 否 | 不必要的开销 |
| 并行任务执行 | 是 | 每个子代理有自己的上下文 |
| 需要专门专业知识 | 是 | 自定义系统提示词 |
| 长期分析 | 是 | 防止主上下文耗尽 |
| 单个任务 | 否 | 不必要地增加延迟 |

---

## 最佳实践

### 设计原则

**应该做的事：**
- 从 Claude 生成的代理开始 - 用 Claude 生成初始子代理，然后迭代定制
- 设计聚焦的子代理 - 单一、清晰的职责而非一个做所有事
- 编写详细提示词 - 包含具体说明、示例和约束
- 限制工具访问 - 仅授予子代理目的所需的必要工具
- 版本控制 - 将项目子代理提交到版本控制以便团队协作

**不应该做的事：**
- 创建角色相同的重叠子代理
- 给子代理不必要的工具访问
- 对简单、单个步骤的任务使用子代理
- 在一个子代理的提示词中混合关注点
- 忘记传递必要的上下文

### 系统提示词最佳实践

1. **明确角色**
   ```
   你是一位专注于 [特定领域] 的专业代码审查员
   ```

2. **明确优先级**
   ```
   审查优先级（按顺序）：
   1. 安全问题
   2. 性能问题
   3. 代码质量
   ```

3. **指定输出格式**
   ```
   对每个问题提供：严重性、类别、位置、描述、修复、影响
   ```

4. **包含行动步骤**
   ```
   调用时：
   1. 运行 git diff 查看最近的变更
   2. 专注于修改的文件
   3. 立即开始审查
   ```

### 工具访问策略

1. **从限制开始**：仅从必要的工具开始
2. **仅在需要时扩展**：根据需求添加工具
3. **尽可能只读**：对分析代理使用 Read/Grep
4. **沙箱执行**：将 Bash 命令限制为特定模式

---

## 本文件夹中的示例子代理

此文件夹包含可直接使用的示例子代理：

### 1. 代码审查员 (`code-reviewer.md`)

**用途**：全面的代码质量和可维护性分析

**工具**：Read、Grep、Glob、Bash

**专长**：
- 安全漏洞检测
- 性能优化识别
- 代码可维护性评估
- 测试覆盖率分析

**使用场景**：需要关注质量和安全性的自动化代码审查时

---

### 2. 测试工程师 (`test-engineer.md`)

**用途**：测试策略、覆盖率分析和自动化测试

**工具**：Read、Write、Bash、Grep

**专长**：
- 单元测试创建
- 集成测试设计
- 边界情况识别
- 覆盖率分析（目标 80%+）

**使用场景**：需要全面测试套件创建或覆盖率分析时

---

### 3. 文档写作员 (`documentation-writer.md`)

**用途**：技术文档、API 文档和用户指南

**工具**：Read、Write、Grep

**专长**：
- API 端点文档
- 用户指南创建
- 架构文档
- 代码注释改进

**使用场景**：需要创建或更新项目文档时

---

### 4. 安全审查员 (`secure-reviewer.md`)

**用途**：具有最小权限的安全重点代码审查

**工具**：Read、Grep

**专长**：
- 安全漏洞检测
- 身份验证/授权问题
- 数据暴露风险
- 注入攻击识别

**使用场景**：需要安全审计但无需修改能力时

---

### 5. 实现代理 (`implementation-agent.md`)

**用途**：功能开发的全栈实现

**工具**：Read、Write、Edit、Bash、Grep、Glob

**专长**：
- 功能实现
- 代码生成
- 构建和测试执行
- 代码库修改

**使用场景**：需要子代理端到端实现功能时

---

### 6. 调试器 (`debugger.md`)

**用途**：错误、测试失败和异常行为的调试专家

**工具**：Read、Edit、Bash、Grep、Glob

**专长**：
- 根本原因分析
- 错误调查
- 测试失败解决
- 最小修复实现

**使用场景**：遇到 bug、错误或异常行为时

---

### 7. 数据科学家 (`data-scientist.md`)

**用途**：SQL 查询和数据分析专家

**工具**：Bash、Read、Write

**专长**：
- SQL 查询优化
- BigQuery 操作
- 数据分析和可视化
- 统计洞察

**使用场景**：需要数据分析、SQL 查询或 BigQuery 操作时

---

## 安装说明

### 方法 1：使用 /agents 命令（推荐）

```bash
/agents
```

然后：
1. 选择"创建新代理"
2. 选择项目级或用户级
3. 详细描述您的子代理
4. 选择授予访问的工具（或留空继承全部）
5. 保存并使用

### 方法 2：复制到项目

将代理文件复制到项目的 `.claude/agents/` 目录：

```bash
# 导航到您的项目
cd /path/to/your/project

# 如果不存在，创建 agents 目录
mkdir -p .claude/agents

# 从此文件夹复制所有代理文件
cp /path/to/04-subagents/*.md .claude/agents/

# 移除 README（.claude/agents 中不需要）
rm .claude/agents/README.md
```

### 方法 3：复制到用户目录

对于所有项目中可用的代理：

```bash
# 创建用户代理目录
mkdir -p ~/.claude/agents

# 复制代理
cp /path/to/04-subagents/code-reviewer.md ~/.claude/agents/
cp /path/to/04-subagents/debugger.md ~/.claude/agents/
# ... 根据需要复制其他
```

### 验证

安装后，验证代理被识别：

```bash
/agents
```

您应该看到已安装的代理与内置代理一起列出。

---

## 文件结构

```
project/
├── .claude/
│   └── agents/
│       ├── code-reviewer.md
│       ├── test-engineer.md
│       ├── documentation-writer.md
│       ├── secure-reviewer.md
│       ├── implementation-agent.md
│       ├── debugger.md
│       └── data-scientist.md
└── ...
```

---

## 相关概念

### 相关功能

- **[斜杠命令](../01-slash-commands/)** - 快速用户触发的快捷方式
- **[内存](../02-memory/)** - 持久跨会话上下文
- **[技能](../03-skills/)** - 可复用的自主能力
- **[MCP 协议](../05-mcp/)** - 实时外部数据访问
- **[钩子](../06-hooks/)** - 事件驱动的 shell 命令自动化
- **[插件](../07-plugins/)** - 捆绑的扩展包

### 与其他功能对比

| 功能 | 用户触发 | 自动触发 | 持久 | 外部访问 | 隔离上下文 |
|---------|--------------|--------------|-----------|------------------|------------------|
| **斜杠命令** | 是 | 否 | 否 | 否 | 否 |
| **子代理** | 是 | 是 | 否 | 否 | 是 |
| **内存** | 自动 | 自动 | 是 | 否 | 否 |
| **MCP** | 自动 | 是 | 否 | 是 | 否 |
| **技能** | 是 | 是 | 否 | 否 | 否 |

### 集成模式

```mermaid
graph TD
    User["用户请求"] --> Main["主代理"]
    Main -->|使用| Memory["内存<br/>(上下文)"]
    Main -->|查询| MCP["MCP<br/>(实时数据)"]
    Main -->|调用| Skills["技能<br/>(自动工具)"]
    Main -->|委托| Subagents["子代理<br/>(专家)"]

    Subagents -->|使用| Memory
    Subagents -->|查询| MCP
    Subagents -->|隔离| Context["干净上下文<br/>窗口"]
```

---

## 附加资源

- [官方子代理文档](https://code.claude.com/docs/en/sub-agents)
- [CLI 参考](https://code.claude.com/docs/en/cli-reference) - `--agents` 标志和其他 CLI 选项
- [插件指南](../07-plugins/) - 用于捆绑代理及其他功能
- [技能指南](../03-skills/) - 用于自动调用能力
- [内存指南](../02-memory/) - 用于持久上下文
- [钩子指南](../06-hooks/) - 用于事件驱动自动化

---

*最后更新：2026 年 3 月*

*本指南涵盖 Claude Code 的完整子代理配置、委托模式和最佳实践。*
