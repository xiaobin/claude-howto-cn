<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 斜杠命令

## 概述

斜杠命令是控制 Claude 在交互会话中行为的快捷方式，分为多种类型：

- **内置命令**：由 Claude Code 提供（`/help`、`/clear`、`/model`）
- **技能**：用户自定义命令，创建为 `SKILL.md` 文件（`/optimize`、`/pr`）
- **插件命令**：来自已安装插件的命令（`/frontend-design:frontend-design`）
- **MCP 提示词**：来自 MCP 服务器的命令（`/mcp__github__list_prs`）

> **注意**：自定义斜杠命令已合并到技能中。`.claude/commands/` 中的文件仍然有效，但技能（`.claude/skills/`）现在是推荐方式。两者都会创建 `/command-name` 快捷方式。详见 [技能指南](../03-skills/)。

## 内置命令参考

内置命令是常见操作的快捷方式。目前有 **55+ 个内置命令**和 **5 个捆绑技能**。在 Claude Code 中输入 `/` 可查看完整列表，或输入 `/` 后跟任意字母进行过滤。

| 命令 | 用途 |
|---------|---------|
| `/add-dir <path>` | 添加工作目录 |
| `/agents` | 管理代理配置 |
| `/branch [name]` | 将对话分支到新会话（别名：`/fork`）。注意：v2.1.77 中 `/fork` 已更名为 `/branch` |
| `/btw <question>` | 附带问题，不加入历史记录 |
| `/chrome` | 配置 Chrome 浏览器集成 |
| `/clear` | 清除对话（别名：`/reset`、`/new`） |
| `/color [color\|default]` | 设置提示词栏颜色 |
| `/compact [instructions]` | 压缩对话，可选带聚焦指令 |
| `/config` | 打开设置（别名：`/settings`） |
| `/context` | 可视化上下文使用情况为彩色网格 |
| `/copy [N]` | 复制助手回复到剪贴板；`w` 写入文件 |
| `/cost` | 显示 Token 使用统计 |
| `/desktop` | 在桌面应用中继续（别名：`/app`） |
| `/diff` | 对未提交更改进行交互式 Diff 查看 |
| `/doctor` | 诊断安装健康状态 |
| `/effort [low\|medium\|high\|max\|auto]` | 设置努力程度。`max` 需要 Opus 4.6 |
| `/exit` | 退出 REPL（别名：`/quit`） |
| `/export [filename]` | 将会话导出到文件或剪贴板 |
| `/extra-usage` | 为速率限制配置额外用量 |
| `/fast [on\|off]` | 切换快速模式 |
| `/feedback` | 提交反馈（别名：`/bug`） |
| `/help` | 显示帮助 |
| `/hooks` | 查看钩子配置 |
| `/ide` | 管理 IDE 集成 |
| `/init` | 初始化 `CLAUDE.md`。设置 `CLAUDE_CODE_NEW_INIT=true` 可启用交互式流程 |
| `/insights` | 生成会话分析报告 |
| `/install-github-app` | 设置 GitHub Actions 应用 |
| `/install-slack-app` | 安装 Slack 应用 |
| `/keybindings` | 打开按键绑定配置 |
| `/login` | 切换 Anthropic 账户 |
| `/logout` | 退出 Anthropic 账户 |
| `/mcp` | 管理 MCP 服务器和 OAuth |
| `/memory` | 编辑 `CLAUDE.md`，切换自动记忆 |
| `/mobile` | 移动应用二维码（别名：`/ios`、`/android`） |
| `/model [model]` | 选择模型，左右箭头调整努力程度 |
| `/passes` | 分享免费周 Claude Code |
| `/permissions` | 查看/更新权限（别名：`/allowed-tools`） |
| `/plan [description]` | 进入计划模式 |
| `/plugin` | 管理插件 |
| `/pr-comments [PR]` | 获取 GitHub PR 评论 |
| `/privacy-settings` | 隐私设置（仅 Pro/Max） |
| `/release-notes` | 查看更新日志 |
| `/reload-plugins` | 重新加载活动插件 |
| `/remote-control` | 从 claude.ai 远程控制（别名：`/rc`） |
| `/remote-env` | 配置默认远程环境 |
| `/rename [name]` | 重命名会话 |
| `/resume [session]` | 恢复对话（别名：`/continue`） |
| `/review` | **已弃用** — 请安装 `code-review` 插件 |
| `/rewind` | 回退对话和/或代码（别名：`/checkpoint`） |
| `/sandbox` | 切换沙箱模式 |
| `/schedule [description]` | 创建/管理计划任务 |
| `/security-review` | 分析分支安全漏洞 |
| `/skills` | 列出可用技能 |
| `/stats` | 可视化每日用量、会话、连续记录 |
| `/status` | 显示版本、模型、账户 |
| `/statusline` | 配置状态栏 |
| `/tasks` | 列出/管理后台任务 |
| `/terminal-setup` | 配置终端按键绑定 |
| `/theme` | 更改颜色主题 |
| `/vim` | 切换 Vim/普通模式 |
| `/voice` | 切换按键说话语音听写 |

### 捆绑技能

这些技能随 Claude Code 一起提供，通过斜杠命令方式调用：

| 技能 | 用途 |
|-------|---------|
| `/batch <instruction>` | 使用工作树编排大规模并行更改 |
| `/claude-api` | 为项目语言加载 Claude API 参考 |
| `/debug [description]` | 启用调试日志 |
| `/loop [interval] <prompt>` | 按间隔重复运行提示词 |
| `/simplify [focus]` | 审查已更改文件的代码质量 |

### 已弃用的命令

| 命令 | 状态 |
|---------|--------|
| `/review` | 已弃用 — 由 `code-review` 插件替代 |
| `/output-style` | 自 v2.1.73 起已弃用 |
| `/fork` | 已更名为 `/branch`（别名仍可用，v2.1.77） |

### 最近更新

- `/fork` 更名为 `/branch`，保留 `/fork` 作为别名（v2.1.77）
- `/output-style` 已弃用（v2.1.73）
- `/review` 已弃用，改用 `code-review` 插件
- 新增 `effort` 命令，`max` 级别需要 Opus 4.6
- 新增 `voice` 命令，支持按键说话语音听写
- 新增 `schedule` 命令，用于创建/管理计划任务
- 新增 `color` 命令，用于自定义提示词栏
- `model` 选择器现在显示人类可读标签（如"Sonnet 4.6"）而非原始模型 ID
- `/resume` 支持 `/continue` 别名
- MCP 提示词可通过 `/mcp__<server>__<prompt>` 命令使用（见 [MCP 提示词作为命令](#mcp-提示词作为命令)）

## 自定义命令（现为技能）

自定义斜杠命令已**合并到技能中**。两种方式都会创建可使用 `/command-name` 调用的命令：

| 方式 | 位置 | 状态 |
|----------|----------|--------|
| **技能（推荐）** | `.claude/skills/<name>/SKILL.md` | 当前标准 |
| **旧版命令** | `.claude/commands/<name>.md` | 仍然有效 |

如果技能和命令同名，**技能优先**。例如，当 `.claude/commands/review.md` 和 `.claude/skills/review/SKILL.md` 同时存在时，使用技能版本。

### 迁移路径

现有的 `.claude/commands/` 文件无需更改即可继续使用。迁移到技能：

**迁移前（命令）：**
```
.claude/commands/optimize.md
```

**迁移后（技能）：**
```
.claude/skills/optimize/SKILL.md
```

### 为什么选择技能？

技能相比旧版命令提供额外功能：

- **目录结构**：打包脚本、模板和参考文件
- **自动调用**：Claude 可以在相关时自动触发技能
- **调用控制**：选择用户、Claude 或两者都可以调用
- **子代理执行**：使用 `context: fork` 在隔离上下文中运行技能
- **渐进式加载**：仅在需要时加载额外文件

### 创建技能格式的自定义命令

创建包含 `SKILL.md` 文件的目录：

```bash
mkdir -p .claude/skills/my-command
```

**文件：** `.claude/skills/my-command/SKILL.md`

```yaml
---
name: my-command
description: 此命令的用途及使用时机
---

# My Command

调用此命令时 Claude 应遵循的指令。

1. 第一步
2. 第二步
3. 第三步
```

### Frontmatter 参考

| 字段 | 用途 | 默认值 |
|-------|---------|---------|
| `name` | 命令名称（成为 `/name`） | 目录名 |
| `description` | 简短描述（帮助 Claude 判断使用时机） | 第一段 |
| `argument-hint` | 自动补全的预期参数 | 无 |
| `allowed-tools` | 命令可无需授权使用的工具 | 继承 |
| `model` | 使用的特定模型 | 继承 |
| `disable-model-invocation` | 若为 `true`，仅用户可调用（Claude 不可） | `false` |
| `user-invocable` | 若为 `false`，在 `/` 菜单中隐藏 | `true` |
| `context` | 设为 `fork` 可在隔离子代理中运行 | 无 |
| `agent` | 使用 `context: fork` 时的代理类型 | `general-purpose` |
| `hooks` | 技能范围的钩子（PreToolUse、PostToolUse、Stop） | 无 |

### 参数

命令可以接收参数：

**使用 `$ARGUMENTS` 接收所有参数：**

```yaml
---
name: fix-issue
description: 按编号修复 GitHub issue
---

修复 issue #$ARGUMENTS，遵循我们的编码标准
```

用法：`/fix-issue 123` → `$ARGUMENTS` 变为"123"

**使用 `$0`、`$1` 等接收独立参数：**

```yaml
---
name: review-pr
description: 按优先级审查 PR
---

以优先级 $1 审查 PR #$0
```

用法：`/review-pr 456 high` → `$0`="456"，`$1`="high"

### 使用 Shell 命令的动态上下文

在提示词前使用 `` !`command` `` 执行 bash 命令：

```yaml
---
name: commit
description: 创建带有上下文的 git 提交
allowed-tools: Bash(git *)
---

## Context

- 当前 git 状态：!`git status`
- 当前 git diff：!`git diff HEAD`
- 当前分支：!`git branch --show-current`
- 最近提交：!`git log --oneline -5`

## 你的任务

根据以上更改，创建单个 git 提交。
```

### 文件引用

使用 `@` 包含文件内容：

```markdown
审查 @src/utils/helpers.js 中的实现
对比 @src/old-version.js 与 @src/new-version.js
```

## 插件命令

插件可以提供自定义命令：

```
/plugin-name:command-name
```

或者当没有命名冲突时直接使用 `/command-name`。

**示例：**
```bash
/frontend-design:frontend-design
/commit-commands:commit
```

## MCP 提示词作为命令

MCP 服务器可以将提示词暴露为斜杠命令：

```
/mcp__<server-name>__<prompt-name> [arguments]
```

**示例：**
```bash
/mcp__github__list_prs
/mcp__github__pr_review 456
/mcp__jira__create_issue "Bug title" high
```

### MCP 权限语法

在权限中控制 MCP 服务器访问：

- `mcp__github` — 访问整个 GitHub MCP 服务器
- `mcp__github__*` — 通配符访问所有工具
- `mcp__github__get_issue` — 特定工具访问

## 命令架构

```mermaid
graph TD
    A["User Input: /command-name"] --> B{"Command Type?"}
    B -->|Built-in| C["Execute Built-in"]
    B -->|Skill| D["Load SKILL.md"]
    B -->|Plugin| E["Load Plugin Command"]
    B -->|MCP| F["Execute MCP Prompt"]

    D --> G["Parse Frontmatter"]
    G --> H["Substitute Variables"]
    H --> I["Execute Shell Commands"]
    I --> J["Send to Claude"]
    J --> K["Return Results"]
```

## 命令生命周期

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant FS as File System
    participant CLI as Shell/Bash

    User->>Claude: Types /optimize
    Claude->>FS: Searches .claude/skills/ and .claude/commands/
    FS-->>Claude: Returns optimize/SKILL.md
    Claude->>Claude: Parses frontmatter
    Claude->>CLI: Executes !`command` substitutions
    CLI-->>Claude: Command outputs
    Claude->>Claude: Substitutes $ARGUMENTS
    Claude->>User: Processes prompt
    Claude->>User: Returns results
```

## 本文件夹中的可用命令

这些示例命令可以作为技能或旧版命令安装。

### 1. `/optimize` - 代码优化

分析代码的性能问题、内存泄漏和优化机会。

**用法：**
```
/optimize
[Paste your code]
```

### 2. `/pr` - Pull Request 准备

引导完成 PR 准备清单，包括检查、测试和提交格式化。

**用法：**
```
/pr
```

**截图：**
![/pr](pr-slash-command.png)

### 3. `/generate-api-docs` - API 文档生成器

从源代码生成全面的 API 文档。

**用法：**
```
/generate-api-docs
```

### 4. `/commit` - 带有上下文的 Git 提交

从仓库动态获取上下文，创建 git 提交。

**用法：**
```
/commit [optional message]
```

### 5. `/push-all` - 暂存、提交并推送

暂存所有更改，创建提交，并带安全检查推送到远程。

**用法：**
```
/push-all
```

**安全检查：**
- 密钥：`.env*`、`*.key`、`*.pem`、`credentials.json`
- API 密钥：检测真实密钥与占位符
- 大文件：大于 10MB 且无 Git LFS
- 构建产物：`node_modules/`、`dist/`、`__pycache__/`

### 6. `/doc-refactor` - 文档重构

重构项目文档以提高清晰度和可访问性。

**用法：**
```
/doc-refactor
```

### 7. `/setup-ci-cd` - CI/CD 流水线设置

实现 pre-commit 钩子和 GitHub Actions 进行质量保证。

**用法：**
```
/setup-ci-cd
```

### 8. `/unit-test-expand` - 测试覆盖率扩展

通过针对未测试的分支和边界情况来增加测试覆盖率。

**用法：**
```
/unit-test-expand
```

## 安装

### 作为技能（推荐）

复制到技能目录：

```bash
# 创建技能目录
mkdir -p .claude/skills

# 对于每个命令文件，创建技能目录
for cmd in optimize pr commit; do
  mkdir -p .claude/skills/$cmd
  cp 01-slash-commands/$cmd.md .claude/skills/$cmd/SKILL.md
done
```

### 作为旧版命令

复制到命令目录：

```bash
# 项目级别（团队）
mkdir -p .claude/commands
cp 01-slash-commands/*.md .claude/commands/

# 个人使用
mkdir -p ~/.claude/commands
cp 01-slash-commands/*.md ~/.claude/commands/
```

## 创建你自己的命令

### 技能模板（推荐）

创建 `.claude/skills/my-command/SKILL.md`：

```yaml
---
name: my-command
description: 此命令的用途。使用时机：[触发条件]。
argument-hint: [可选参数]
allowed-tools: Bash(npm *), Read, Grep
---

# Command Title

## Context

- Current branch: !`git branch --show-current`
- Related files: @package.json

## Instructions

1. First step
2. Second step with argument: $ARGUMENTS
3. Third step

## Output Format

- How to format the response
- What to include
```

### 仅用户可调用的命令（无自动调用）

对于有副作用的命令，Claude 不应自动触发：

```yaml
---
name: deploy
description: 部署到生产环境
disable-model-invocation: true
allowed-tools: Bash(npm *), Bash(git *)
---

将应用部署到生产环境：

1. 运行测试
2. 构建应用
3. 推送到部署目标
4. 验证部署
```

## 最佳实践

| 宜 | 忌 |
|------|---------|
| 使用清晰、动作导向的名称 | 为一次性任务创建命令 |
| 在 `description` 中包含触发条件 | 在命令中构建复杂逻辑 |
| 保持命令专注于单一任务 | 硬编码敏感信息 |
| 使用 `disable-model-invocation` 处理副作用 | 跳过 description 字段 |
| 使用 `!` 前缀获取动态上下文 | 假设 Claude 知道当前状态 |
| 在技能目录中组织相关文件 | 将所有内容放在一个文件中 |

## 故障排除

### 命令未找到

**解决方案：**
- 检查文件是否在 `.claude/skills/<name>/SKILL.md` 或 `.claude/commands/<name>.md`
- 验证 frontmatter 中的 `name` 字段与预期的命令名称匹配
- 重启 Claude Code 会话
- 运行 `/help` 查看可用命令

### 命令未按预期执行

**解决方案：**
- 添加更具体的指令
- 在技能文件中包含示例
- 如果使用 bash 命令，检查 `allowed-tools`
- 先用简单输入测试

### 技能与命令冲突

如果两者同名存在，**技能优先**。删除其中一个或重命名。

## 相关指南

- **[技能](../03-skills/)** — 技能完整参考（自动调用的能力）
- **[记忆](../02-memory/)** — 使用 CLAUDE.md 持久化上下文
- **[子代理](../04-subagents/)** — 委托 AI 代理
- **[插件](../07-plugins/)** — 捆绑的命令集合
- **[钩子](../06-hooks/)** — 事件驱动的自动化

## 其他资源

- [官方交互模式文档](https://code.claude.com/docs/en/interactive-mode) — 内置命令参考
- [官方技能文档](https://code.claude.com/docs/en/skills) — 完整技能参考
- [CLI 参考](https://code.claude.com/docs/en/cli-reference) — 命令行选项

---

*属于 [Claude How To](../) 指南系列*
