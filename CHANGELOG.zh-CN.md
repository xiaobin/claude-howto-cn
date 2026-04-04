# 变更日志

## v2.2.0 — 2026-03-26

### 文档

- 与 Claude Code v2.1.84 同步所有教程和参考（f78c094）@luongnv89
  - 更新斜杠命令至 55+ 内置 + 5 个捆绑技能，标记 3 个已弃用
  - 扩展钩子事件从 18 到 25，添加 `agent` 钩子类型（现在 4 种类型）
  - 将 Auto Mode、频道、语音听写添加到高级功能
  - 添加 `effort`、`shell` 技能前置matter字段；`initialPrompt`、`disallowedTools` 代理字段
  - 添加 WebSocket MCP 传输、征求、2KB 工具上限
  - 添加插件 LSP 支持、`userConfig`、`${CLAUDE_PLUGIN_DATA}`
  - 更新所有参考文档（CATALOG、QUICK_REFERENCE、LEARNING-ROADMAP、INDEX）
- 将 README 重写为落地页结构化指南（32a0776）@luongnv89

### Bug 修复

- 添加缺失的 cSpell 单词和 README 部分以符合 CI 要求（93f9d51）@luongnv89
- 将 `Sandboxing` 添加到 cSpell 字典（b80ce6f）@luongnv89

**完整变更日志**：https://github.com/luongnv89/claude-howto/compare/v2.1.1...v2.2.0

---

## v2.1.1 — 2026-03-13

### Bug 修复

- 移除导致 CI 链接检查失败的已失效 marketplace 链接（3fdf0d6）@luongnv89
- 将 `sandboxed` 和 `pycache` 添加到 cSpell 字典（dc64618）@luongnv89

**完整变更日志**：https://github.com/luongnv89/claude-howto/compare/v2.1.0...v2.1.1

---

## v2.1.0 — 2026-03-13

### 功能

- 添加带自我评估和课程测验技能的自适应学习路径（1ef46cd）@luongnv89
  - `/self-assessment` —— 跨 10 个功能领域的互动熟练程度测验，生成个性化学习路径
  - `/lesson-quiz [lesson]` —— 每课知识点检查，包含 8-10 个有针对性的问题

### Bug 修复

- 更新失效 URL、弃用和过时引用（8fe4520）@luongnv89
- 修复资源和自我评估技能中的失效链接（7a05863）@luongnv89
- 在概念指南中对嵌套代码块使用波浪号围栏（5f82719）@VikalpP
- 将缺失单词添加到 cSpell 字典（8df7572）@luongnv89

### 文档

- 第 5 阶段 QA —— 修复文档中的一致性、URL 和术语（00bbe4c）@luongnv89
- 完成第 3-4 阶段 —— 新功能覆盖和参考文档更新（132de29）@luongnv89
- 将 MCPorter 运行时添加到 MCP 上下文膨胀部分（ef52705）@luongnv89
- 在 6 个指南中添加缺失的命令、功能和设置（4bc8f15）@luongnv89
- 添加基于现有仓库约定的风格指南（84141d0）@luongnv89
- 在指南对比表中添加自我评估行（8fe0c96）@luongnv89
- 将 VikalpP 添加到 PR #7 的贡献者列表（d5b4350）@luongnv89
- 在 README 和路线图中添加自我评估和 lesson-quiz 技能参考（d5a6106）@luongnv89

### 新贡献者

- @VikalpP 首次在 #7 中贡献

**完整变更日志**：https://github.com/luongnv89/claude-howto/compare/v2.0.0...v2.1.0

---

## v2.0.0 — 2026-02-01

### 功能

- 与 Claude Code 2026 年 2 月功能同步所有文档（487c96d）
  - 更新跨 10 个教程目录和 7 个参考文档的 26 个文件
  - 添加 **Auto Memory** 文档 —— 每个项目的持久化学习
  - 添加 **Remote Control**、**Web Sessions** 和 **Desktop App** 文档
  - 添加 **Agent Teams**（实验性多代理协作）文档
  - 添加 **MCP OAuth 2.0**、**Tool Search** 和 **Claude.ai Connectors** 文档
  - 添加 **Persistent Memory** 和 **Worktree Isolation** 文档（用于子代理）
  - 添加 **Background Subagents**、**Task List**、**Prompt Suggestions** 文档
  - 添加 **Sandboxing** 和 **Managed Settings**（企业版）文档
  - 添加 **HTTP Hooks** 和 7 个新钩子事件文档
  - 添加 **Plugin Settings**、**LSP Servers** 和 Marketplace 更新文档
  - 添加 **Summarize from Checkpoint** 回退选项文档
  - 记录 17 个新斜杠命令（`/fork`、`/desktop`、`/teleport`、`/tasks`、`/fast` 等）
  - 记录新 CLI 标志（`--worktree`、`--from-pr`、`--remote`、`--teleport`、`--teammate-mode` 等）
  - 记录用于自动记忆、努力级别、代理团队等的新环境变量

### 设计

- 重新设计 logo，采用指南针括号标记和简约配色方案（20779db）

### Bug 修复/更正

- 更新模型名称：Sonnet 4.5 → **Sonnet 4.6**，Opus 4.5 → **Opus 4.6**
- 修复权限模式名称：用实际的 `default`/`acceptEdits`/`plan`/`dontAsk`/`bypassPermissions` 替换虚构的"Unrestricted/Confirm/Read-only"
- 修复钩子事件：移除虚构的 `PreCommit`/`PostCommit`/`PrePush`，添加真实事件（`SubagentStart`、`WorktreeCreate`、`ConfigChange` 等）
- 修复 CLI 语法：用 `claude -p`（打印模式）替换 `claude-code --headless`
- 修复检查点命令：用实际的 `Esc+Esc` / `/rewind` 界面替换虚构的 `/checkpoint save/list/rewind/diff`
- 修复会话管理：用真实的 `/resume`/`/rename`/`/fork` 替换虚构的 `/session list/new/switch/save`
- 修复插件清单格式：`plugin.yaml` → `.claude-plugin/plugin.json`
- 修复 MCP 配置路径：`~/.claude/mcp.json` → `.mcp.json`（项目）/ `~/.claude.json`（用户）
- 修复文档 URL：`docs.claude.com` → `docs.anthropic.com`；移除虚构的 `plugins.claude.com`
- 移除跨多个文件的虚构配置字段
- 将所有"最后更新"日期更新至 2026 年 2 月

**完整变更日志**：https://github.com/luongnv89/claude-howto/compare/20779db...v2.0.0
