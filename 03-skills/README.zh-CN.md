<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# Agent 技能指南

Agent 技能是 可复用的、基于文件系统的能力，用于扩展 Claude 的功能。它们将特定领域的专业知识、工作流程和最佳实践打包成可发现的组件，Claude 会在相关场景下自动使用。

## 概述

**Agent 技能**（Skills）是模块化能力，将通用代理转变为专家。与提示词（针对一次性任务的对话级指令）不同，技能按需加载，无需在多个对话中重复提供相同的指导。

### 关键优势

- **专业化 Claude**：为特定领域任务定制能力
- **减少重复**：一次创建，跨对话自动使用
- **组合能力**：组合多个技能构建复杂工作流程
- **扩展工作流程**：跨多个项目和团队复用技能
- **保持质量**：将最佳实践直接嵌入工作流程

技能遵循 [Agent Skills](https://agentskills.io) 开放标准，可在多个 AI 工具中使用。Claude Code 在此标准基础上扩展了额外功能，如调用控制、子代理执行和动态上下文注入。

> **注意**：自定义斜杠命令已合并到技能中。`.claude/commands/` 文件仍然有效并支持相同的前置matter字段。建议新开发使用技能。当同一路径同时存在两者时（例如 `.claude/commands/review.md` 和 `.claude/skills/review/SKILL.md`），技能优先。

## 技能如何工作：渐进式披露

技能利用**渐进式披露**架构——Claude 按需分阶段加载信息，而非预先消耗上下文。这实现了高效的上下文管理，同时保持无限的可扩展性。

### 三级加载

```mermaid
graph TB
    subgraph "级别 1：元数据（始终加载）"
        A["YAML Frontmatter"]
        A1["每个技能约 100 tokens"]
        A2["name + description"]
    end

    subgraph "级别 2：指令（触发时加载）"
        B["SKILL.md 正文"]
        B1["5k tokens 以内"]
        B2["工作流程和指导"]
    end

    subgraph "级别 3+：资源（按需加载）"
        C["捆绑文件"]
        C1["实际上无限制"]
        C2["脚本、模板、文档"]
    end

    A --> B
    B --> C
```

| 级别 | 何时加载 | Token 成本 | 内容 |
|-------|------------|------------|---------|
| **级别 1：元数据** | 始终（启动时） | 每个技能约 100 tokens | YAML frontmatter 中的 `name` 和 `description` |
| **级别 2：指令** | 技能被触发时 | 5k tokens 以内 | 带指令和指导的 SKILL.md 正文 |
| **级别 3+：资源** | 按需 | 实际上无限制 | 捆绑文件通过 bash 执行，不将内容加载到上下文 |

这意味着你可以在不增加上下文负担的情况下安装许多技能——Claude 仅在技能被实际触发前知道每个技能的存在及其使用时机。

## 技能加载流程

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude
    participant System as System
    participant Skill as Skill

    User->>Claude: "Review this code for security issues"
    Claude->>System: Check available skills (metadata)
    System-->>Claude: Skill descriptions loaded at startup
    Claude->>Claude: Match request to skill description
    Claude->>Skill: bash: read code-review/SKILL.md
    Skill-->>Claude: Instructions loaded into context
    Claude->>Claude: Determine: Need templates?
    Claude->>Skill: bash: read templates/checklist.md
    Skill-->>Claude: Template loaded
    Claude->>Claude: Execute skill instructions
    Claude->>User: Comprehensive code review
```

## 技能类型与位置

| 类型 | 位置 | 范围 | 共享 | 适用场景 |
|------|----------|-------|--------|----------|
| **企业级** | 托管设置 | 所有组织用户 | 是 | 组织级标准 |
| **个人** | `~/.claude/skills/<skill-name>/SKILL.md` | 个人 | 否 | 个人工作流程 |
| **项目** | `.claude/skills/<skill-name>/SKILL.md` | 团队 | 是（通过 git） | 团队标准 |
| **插件** | `<plugin>/skills/<skill-name>/SKILL.md` | 启用处 | 取决于插件 | 与插件捆绑 |

当多个级别存在同名技能时，优先级高的位置优先：**企业 > 个人 > 项目**。插件技能使用 `plugin-name:skill-name` 命名空间，因此不会冲突。

### 自动发现

**嵌套目录**：当你在子目录中处理文件时，Claude Code 会自动从嵌套的 `.claude/skills/` 目录中发现技能。例如，如果你正在编辑 `packages/frontend/` 中的文件，Claude Code 也会在 `packages/frontend/.claude/skills/` 中查找技能。这支持 monorepo 中各包拥有自己的技能。

**`--add-dir` 目录**：通过 `--add-dir` 添加的目录中的技能会自动加载并实时检测变更。对这些目录中技能文件的任何编辑会立即生效，无需重启 Claude Code。

**描述预算**：技能描述（级别 1 元数据）上限为上下文窗口的 **2%**（后备：**16,000 个字符**）。如果你安装了很多技能，有些可能被排除。运行 `/context` 检查警告。使用 `SLASH_COMMAND_TOOL_CHAR_BUDGET` 环境变量覆盖预算。

## 创建自定义技能

### 基本目录结构

```
my-skill/
├── SKILL.md           # 主要指令（必需）
├── template.md        # Claude 填充的模板
├── examples/
│   └── sample.md      # 显示预期格式的示例输出
└── scripts/
    └── validate.sh    # Claude 可执行的脚本
```

### SKILL.md 格式

```yaml
---
name: your-skill-name
description: 简要描述此技能的作用及使用时机
---

# Your Skill Name

## Instructions
为 Claude 提供清晰的分步指导。

## Examples
展示使用此技能的具体示例。
```

### 必需字段

- **name**：仅小写字母、数字和连字符（最多 64 个字符）。不能包含"anthropic"或"claude"。
- **description**：技能的作用**及**使用时机（最多 1024 个字符）。这对于 Claude 知道何时激活技能至关重要。

### 可选的前置matter字段

```yaml
---
name: my-skill
description: 此技能的作用及使用时机
argument-hint: "[filename] [format]"        # 自动完成提示
disable-model-invocation: true              # 仅用户可调用
user-invocable: false                       # 从斜杠菜单隐藏
allowed-tools: Read, Grep, Glob             # 限制工具访问
model: opus                                 # 要使用的特定模型
effort: high                                # 工作量级别覆盖（low, medium, high, max）
context: fork                               # 在隔离的子代理中运行
agent: Explore                              # 代理类型（配合 context: fork 使用）
shell: bash                                 # 命令使用的 shell：bash（默认）或 powershell
hooks:                                      # 技能作用域钩子
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
---
```

| 字段 | 描述 |
|-------|-------------|
| `name` | 仅小写字母、数字和连字符（最多 64 个字符）。不能包含"anthropic"或"claude"。 |
| `description` | 技能的作用**及**使用时机（最多 1024 个字符）。对自动调用匹配至关重要。 |
| `argument-hint` | `/` 自动完成菜单中显示的提示（例如 `"[filename] [format]"`）。 |
| `disable-model-invocation` | `true` = 仅用户可通过 `/name` 调用。Claude 永远不会自动调用。 |
| `user-invocable` | `false` = 从 `/` 菜单隐藏。仅 Claude 可自动调用。 |
| `allowed-tools` | 技能可使用而无需权限提示的工具的逗号分隔列表。 |
| `model` | 技能激活时的模型覆盖（例如 `opus`、`sonnet`）。 |
| `effort` | 技能激活时的工作量级别覆盖：`low`、`medium`、`high` 或 `max`。 |
| `context` | `fork` 在分叉的子代理上下文中运行技能，拥有自己的上下文窗口。 |
| `agent` | `context: fork` 时的子代理类型（例如 `Explore`、`Plan`、`general-purpose`）。 |
| `shell` | 用于 `` !`command` `` 替换和脚本的 shell：`bash`（默认）或 `powershell`。 |
| `hooks` | 此技能生命周期内的钩子（与全局钩子格式相同）。 |

## 技能内容类型

技能可包含两种类型的内容，各适用于不同目的：

### 参考内容

为 Claude 添加应用于当前工作的知识——约定、模式、样式指南、领域知识。与对话上下文内联运行。

```yaml
---
name: api-conventions
description: 此代码库的 API 设计模式
---

编写 API 端点时：
- 使用 RESTful 命名约定
- 返回一致的错误格式
- 包含请求验证
```

### 任务内容

特定操作的分步指令。通常直接用 `/skill-name` 调用。

```yaml
---
name: deploy
description: 将应用部署到生产环境
context: fork
disable-model-invocation: true
---

部署应用：
1. 运行测试套件
2. 构建应用
3. 推送到部署目标
```

## 控制技能调用

默认情况下，你和 Claude 都可以调用任何技能。两个前置matter字段控制三种调用模式：

| 前置matter | 你可以调用 | Claude 可以调用 |
|---|---|---|
| （默认） | 是 | 是 |
| `disable-model-invocation: true` | 是 | 否 |
| `user-invocable: false` | 否 | 是 |

**使用 `disable-model-invocation: true`** 用于有副作用的工作流程：`/commit`、`/deploy`、`/send-slack-message`。你不会希望 Claude 因为你的代码看起来准备好了就决定部署。

**使用 `user-invocable: false`** 用于非可操作命令的背景知识。`legacy-system-context` 技能解释旧系统如何工作——对 Claude 有用，但对用户不是有意义的操作。

## 字符串替换

技能支持在技能内容到达 Claude 之前解析的动态值：

| 变量 | 描述 |
|----------|-------------|
| `$ARGUMENTS` | 调用技能时传递的所有参数 |
| `$ARGUMENTS[N]` 或 `$N` | 按索引（从 0 开始）访问特定参数 |
| `${CLAUDE_SESSION_ID}` | 当前会话 ID |
| `${CLAUDE_SKILL_DIR}` | 包含技能 SKILL.md 文件的目录 |
| `` !`command` `` | 动态上下文注入——运行 shell 命令并内联输出 |

**示例：**

```yaml
---
name: fix-issue
description: 修复 GitHub issue
---

按照我们的编码标准修复 GitHub issue $ARGUMENTS。
1. 阅读 issue 描述
2. 实现修复
3. 编写测试
4. 创建提交
```

运行 `/fix-issue 123` 会将 `$ARGUMENTS` 替换为 `123`。

## 注入动态上下文

`` !`command` `` 语法在技能内容发送到 Claude 之前运行 shell 命令：

```yaml
---
name: pr-summary
description: 汇总 Pull Request 中的变更
context: fork
agent: Explore
---

## Pull Request 上下文
- PR diff: !`gh pr diff`
- PR 评论: !`gh pr view --comments`
- 变更的文件: !`gh pr diff --name-only`

## 你的任务
汇总此 Pull Request...
```

命令立即执行；Claude 只看到最终输出。默认情况下，命令在 `bash` 中运行。在前置matter中设置 `shell: powershell` 可改用 PowerShell。

## 在子代理中运行技能

添加 `context: fork` 可在隔离的子代理上下文中运行技能。技能内容成为专用子代理的任务，该代理拥有自己的上下文窗口，保持主对话整洁。

`agent` 字段指定要使用的代理类型：

| 代理类型 | 适用场景 |
|---|---|
| `Explore` | 只读研究、代码库分析 |
| `Plan` | 创建实现计划 |
| `general-purpose` | 需要所有工具的广泛任务 |
| 自定义代理 | 在配置中定义的专业代理 |

**示例前置matter：**

```yaml
---
context: fork
agent: Explore
---
```

**完整技能示例：**

```yaml
---
name: deep-research
description: 彻底研究一个主题
context: fork
agent: Explore
---

彻底研究 $ARGUMENTS：
1. 使用 Glob 和 Grep 查找相关文件
2. 阅读并分析代码
3. 总结发现，包含具体文件引用
```

## 实践示例

### 示例 1：代码审查技能

**目录结构：**

```
~/.claude/skills/code-review/
├── SKILL.md
├── templates/
│   ├── review-checklist.md
│   └── finding-template.md
└── scripts/
    ├── analyze-metrics.py
    └── compare-complexity.py
```

**文件：** `~/.claude/skills/code-review/SKILL.md`

```yaml
---
name: code-review-specialist
description: 包含安全性、性能和质量分析的全面代码审查。当用户要求审查代码、分析代码质量、评估 Pull Request，或提及代码审查、安全分析或性能优化时使用。
---

# 代码审查技能

此技能提供全面的代码审查能力，重点关注：

1. **安全分析**
   - 认证/授权问题
   - 数据暴露风险
   - 注入漏洞
   - 密码学弱点

2. **性能审查**
   - 算法效率（Big O 分析）
   - 内存优化
   - 数据库查询优化
   - 缓存机会

3. **代码质量**
   - SOLID 原则
   - 设计模式
   - 命名约定
   - 测试覆盖率

4. **可维护性**
   - 代码可读性
   - 函数大小（应小于 50 行）
   - 圈复杂度
   - 类型安全

## 审查模板

对于每个被审查的代码，提供：

### 总结
- 整体质量评估（1-5）
- 关键发现数量
- 推荐的优先领域

### 关键问题（如果有）
- **问题**：清晰描述
- **位置**：文件及行号
- **影响**：为什么这很重要
- **严重性**：Critical/High/Medium
- **修复**：代码示例

有关详细检查清单，参见 [检查清单模板](code-review/templates/review-checklist.zh-CN.md)。
```

### 示例 2：代码库可视化技能

生成交互式 HTML 可视化的技能：

**目录结构：**

```
~/.claude/skills/codebase-visualizer/
├── SKILL.md
└── scripts/
    └── visualize.py
```

**文件：** `~/.claude/skills/codebase-visualizer/SKILL.md`

```yaml
---
name: codebase-visualizer
description: 生成代码库的交互式可折叠树可视化。在探索新仓库、理解项目结构或识别大文件时使用。
allowed-tools: Bash(python *)
---

# 代码库可视化

生成显示项目文件结构的交互式 HTML 树视图。

## 使用方法

从项目根目录运行可视化脚本：

```bash
python ~/.claude/skills/codebase-visualizer/scripts/visualize.py .
```

这会创建 `codebase-map.html` 并在默认浏览器中打开。

## 可视化内容

- **可折叠目录**：点击文件夹展开/折叠
- **文件大小**：显示在每个文件旁边
- **颜色**：不同文件类型用不同颜色
- **目录总计**：显示每个文件夹的总大小
```

捆绑的 Python 脚本做重活，Claude 处理编排。

### 示例 3：部署技能（仅用户调用）

```yaml
---
name: deploy
description: 将应用部署到生产环境
disable-model-invocation: true
allowed-tools: Bash(npm *), Bash(git *)
---

将 $ARGUMENTS 部署到生产环境：

1. 运行测试套件：`npm test`
2. 构建应用：`npm run build`
3. 推送到部署目标
4. 验证部署成功
5. 报告部署状态
```

### 示例 4：品牌语音技能（背景知识）

```yaml
---
name: brand-voice
description: 确保所有沟通符合品牌语音和语调指南。在创建营销文案、客户沟通、面向公众的内容时使用。
user-invocable: false
---

## 语调
- **友好但专业** - 平易近人但不随意
- **清晰简洁** - 避免行话
- **自信** - 我们知道自己在做什么
- **同理心** - 理解用户需求

## 写作指南
- 用"你"称呼读者
- 使用主动语态
- 保持句子在 20 个词以内
- 从价值主张开始

有关模板，参见 [templates/](templates/)。
```

### 示例 5：CLAUDE.md 生成器技能

```yaml
---
name: claude-md
description: 根据 AI 代理入职最佳实践创建或更新 CLAUDE.md 文件。当用户提及 CLAUDE.md、项目文档或 AI 入职时使用。
---

## 核心原则

**LLM 是无状态的**：CLAUDE.md 是自动包含在每次对话中的唯一文件。

### 黄金法则

1. **少即是多**：保持在 300 行以下（最好低于 100 行）
2. **通用适用性**：只包含与每次会话都相关的信息
3. **不要把 Claude 当作 Linter 使用**：使用确定性工具
4. **永远不要自动生成**：用心手动制作

## 必要章节

- **项目名称**：简短的一行描述
- **技术栈**：主要语言、框架、数据库
- **开发命令**：安装、测试、构建命令
- **关键约定**：仅非显而易见、高影响的约定
- **已知问题 / 陷阱**：经常绊倒开发者的事情
```

### 示例 6：带脚本的重构技能

**目录结构：**

```
refactor/
├── SKILL.md
├── references/
│   ├── code-smells.md
│   └── refactoring-catalog.md
├── templates/
│   └── refactoring-plan.md
└── scripts/
    ├── analyze-complexity.py
    └── detect-smells.py
```

**文件：** `refactor/SKILL.md`

```yaml
---
name: code-refactor
description: 基于 Martin Fowler 方法论的系统性代码重构。当用户要求重构代码、改进代码结构、减少技术债务或消除代码异味时使用。
---

# 代码重构技能

强调以测试为保障的安全增量变更的分阶段方法。

## 工作流程

阶段 1：研究与分析 → 阶段 2：测试覆盖率评估 →
阶段 3：代码异味识别 → 阶段 4：重构计划制定 →
阶段 5：增量实现 → 阶段 6：审查与迭代

## 核心原则

1. **行为保持**：外部行为必须保持不变
2. **小步前进**：进行微小的、可测试的变更
3. **测试驱动**：测试是安全网
4. **持续进行**：重构是持续的过程

有关代码异味目录和重构技术，参见 [重构技能](refactor/SKILL.zh-CN.md)。
```

## 支持文件

技能可以在其目录中包含 SKILL.md 之外的多个文件。这些支持文件（模板、示例、脚本、参考文档）让你保持主技能文件聚焦，同时为 Claude 提供它可以按需加载的额外资源。

```
my-skill/
├── SKILL.md              # 主要指令（必需，保持在 500 行以下）
├── templates/            # Claude 填充的模板
│   └── output-format.md
├── examples/             # 显示预期格式的示例输出
│   └── sample-output.md
├── references/           # 领域知识和规格说明
│   └── api-spec.md
└── scripts/              # Claude 可执行的脚本
    └── validate.sh
```

支持文件指南：

- 保持 `SKILL.md` 在 **500 行**以下。将详细参考材料、大型示例和规格说明移到单独文件。
- 使用**相对路径**从 `SKILL.md` 引用额外文件。
- 支持文件在级别 3（按需）加载，因此在 Claude 实际读取之前不消耗上下文。

## 管理技能

### 查看可用技能

直接问 Claude：
```
What Skills are available?
```

或检查文件系统：
```bash
# 列出个人技能
ls ~/.claude/skills/

# 列出项目技能
ls .claude/skills/
```

### 测试技能

两种测试方式：

**让 Claude 自动调用**通过询问匹配描述的内容：
```
Can you help me review this code for security issues?
```

**或直接调用**技能名称：
```
/code-review src/auth/login.ts
```

### 更新技能

直接编辑 `SKILL.md` 文件。更改在下次 Claude Code 启动时生效。

```bash
# 个人技能
code ~/.claude/skills/my-skill/SKILL.md

# 项目技能
code .claude/skills/my-skill/SKILL.md
```

### 限制 Claude 的技能访问

三种方式控制 Claude 可调用哪些技能：

在 `/permissions` 中**禁用所有技能**：
```
# 添加到拒绝规则：
Skill
```

**允许或拒绝特定技能**：
```
# 仅允许特定技能
Skill(commit)
Skill(review-pr *)

# 拒绝特定技能
Skill(deploy *)
```

**隐藏单个技能**在它们的前置matter中添加 `disable-model-invocation: true`。

## 最佳实践

### 1. 使描述具体

- **差（模糊）**："帮助处理文档"
- **好（具体）**："从 PDF 文件提取文本和表格、填写表单、合并文档。在处理 PDF 文件或用户提及 PDF、表单或文档提取时使用。"

### 2. 保持技能聚焦

- 一个技能 = 一种能力
- 好："PDF 表单填写"
- 差："文档处理"（太宽泛）

### 3. 包含触发词

在描述中添加用户自然会说出的关键词：
```yaml
description: 分析 Excel 电子表格、生成数据透视表、创建图表。在处理 Excel 文件、电子表格或 .xlsx 文件时使用。
```

### 4. 保持 SKILL.md 在 500 行以下

将详细参考材料移到 Claude 按需加载的单独文件。

### 5. 引用支持文件

```markdown
## 附加资源
```

### 应该做的事

- 使用清晰、描述性的名称
- 包含全面指导
- 添加具体示例
- 打包相关脚本和模板
- 用真实场景测试
- 记录依赖

### 不应该做的事

- 不要为一次性任务创建技能
- 不要复制现有功能
- 不要让技能太宽泛
- 不要跳过描述字段
- 不要在未审核的情况下从不信任来源安装技能

## 故障排查

### 快速参考

| 问题 | 解决方案 |
|-------|----------|
| Claude 不使用技能 | 使描述更具体，包含触发词 |
| 找不到技能文件 | 验证路径：`~/.claude/skills/name/SKILL.md` |
| YAML 错误 | 检查 `---` 标记、缩进、不使用 Tab |
| 技能冲突 | 在描述中使用不同的触发词 |
| 脚本不运行 | 检查权限：`chmod +x scripts/*.py` |
| Claude 看不到所有技能 | 技能太多；在 `/context` 检查警告 |

### 技能未触发

如果 Claude 没有按预期使用你的技能：

1. 检查描述是否包含用户自然会说出的关键词
2. 验证在询问"What skills are available?"时技能是否出现
3. 尝试重新表达你的请求以匹配描述
4. 直接用 `/skill-name` 调用测试

### 技能触发过于频繁

如果 Claude 在你不想要时使用你的技能：

1. 使描述更具体
2. 为仅手动调用添加 `disable-model-invocation: true`

### Claude 看不到所有技能

技能描述在上下文窗口的 **2%**（后备：**16,000 个字符**）时加载。运行 `/context` 检查关于排除技能的警告。使用 `SLASH_COMMAND_TOOL_CHAR_BUDGET` 环境变量覆盖预算。

## 安全注意事项

**仅使用来自可信来源的技能。**技能通过指令和代码为 Claude 提供能力——恶意技能可以指导 Claude 以有害方式调用工具或执行代码。

**关键安全注意事项：**

- **彻底审核**：审查技能目录中的所有文件
- **外部来源有风险**：从外部 URL 获取内容的技能可能被篡改
- **工具滥用**：恶意技能可能以有害方式调用工具
- **像安装软件一样对待**：仅使用来自可信来源的技能

## 技能与其他功能对比

| 功能 | 调用方式 | 适用场景 |
|---------|------------|----------|
| **技能** | 自动或 `/name` | 可复用专业知识、工作流程 |
| **斜杠命令** | 用户触发的 `/name` | 快速快捷方式（已合并到技能） |
| **子代理** | 自动委托 | 隔离任务执行 |
| **内存（CLAUDE.md）** | 始终加载 | 持久项目上下文 |
| **MCP** | 实时 | 外部数据/服务访问 |
| **钩子** | 事件驱动 | 自动化副作用 |

## 捆绑技能

Claude Code 附带多个内置技能，无需安装即可始终使用：

| 技能 | 描述 |
|-------|-------------|
| `/simplify` | 审查变更文件以实现复用、质量和效率；生成 3 个并行审查代理 |
| `/batch <instruction>` | 使用 git worktree 在代码库中编排大规模并行变更 |
| `/debug [description]` | 通过读取调试日志排查当前会话 |
| `/loop [interval] <prompt>` | 按间隔重复运行提示（例如 `/loop 5m check the deploy`） |
| `/claude-api` | 加载 Claude API/SDK 参考；在 `anthropic`/`@anthropic-ai/sdk` 导入时自动激活 |

这些技能开箱即用，无需安装或配置。它们遵循与自定义技能相同的 SKILL.md 格式。

## 分享技能

### 项目技能（团队分享）

1. 在 `.claude/skills/` 创建技能
2. 提交到 git
3. 团队成员拉取变更——技能立即可用

### 个人技能

```bash
# 复制到个人目录
cp -r my-skill ~/.claude/skills/

# 使脚本可执行
chmod +x ~/.claude/skills/my-skill/scripts/*.py
```

### 插件分发

在插件的 `skills/` 目录中打包技能以进行更广泛的分发。

## 进一步探索：技能集合和技能管理器

一旦你开始认真构建技能，有两样东西变得必不可少：一个经过验证的技能库和一个管理它们的工具。

**[luongnv89/skills](https://github.com/luongnv89/skills)** — 我在几乎所有项目中每天使用的技能集合。亮点包括 `logo-designer`（动态生成项目 logo）和 `ollama-optimizer`（为您的硬件调优本地 LLM 性能）。如果你想要可直接使用的技能，这是很好的起点。

**[luongnv89/asm](https://github.com/luongnv89/asm)** — 代理技能管理器。处理技能开发、重复检测和测试。`asm link` 命令让你在任何项目中测试技能而无需四处复制文件——一旦你有了十几个技能，这就必不可少。

## 附加资源

- [官方技能文档](https://code.claude.com/docs/en/skills)
- [代理技能架构博客](https://claude.com/blog/equipping-agents-for-the-real-world-with-agent-skills)
- [技能仓库](https://github.com/luongnv89/skills) - 可直接使用的技能集合
- [斜杠命令指南](../01-slash-commands/) - 用户触发的快捷方式
- [子代理指南](../04-subagents/) - 委托的 AI 代理
- [内存指南](../02-memory/) - 持久上下文
- [MCP（模型上下文协议）](../05-mcp/) - 实时外部数据
- [钩子指南](../06-hooks/) - 事件驱动自动化
