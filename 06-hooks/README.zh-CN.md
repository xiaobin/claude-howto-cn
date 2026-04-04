<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 钩子（Hooks）

钩子是在 Claude Code 会话期间响应特定事件自动执行的自动化脚本。它们支持自动化、验证、权限管理和自定义工作流。

## 概述

钩子是自动化操作（shell 命令、HTTP webhook、LLM 提示词或子代理评估），在 Claude Code 中发生特定事件时自动执行。它们接收 JSON 输入并通过退出码和 JSON 输出进行通信。

**主要特点：**
- 事件驱动的自动化
- 基于 JSON 的输入/输出
- 支持命令、提示词、HTTP 和代理钩子类型
- 工具特定钩子的模式匹配

## 配置

钩子在设置文件中配置，具有特定结构：

- `~/.claude/settings.json` - 用户设置（所有项目）
- `.claude/settings.json` - 项目设置（可共享、已提交）
- `.claude/settings.local.json` - 本地项目设置（不提交）
- 托管策略 - 组织范围的设置
- 插件 `hooks/hooks.json` - 插件作用域的钩子
- 技能/代理 frontmatter - 组件生命周期钩子

### 基本配置结构

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

**关键字段：**

| 字段 | 说明 | 示例 |
|-------|-------------|---------|
| `matcher` | 匹配工具名称的模式（区分大小写） | `"Write"`、`"Edit\|Write"`、`"*"` |
| `hooks` | 钩子定义数组 | `[{ "type": "command", ... }]` |
| `type` | 钩子类型：`"command"`（bash）、`"prompt"`（LLM）、`"http"`（webhook）或 `"agent"`（子代理） | `"command"` |
| `command` | 要执行的 shell 命令 | `"$CLAUDE_PROJECT_DIR/.claude/hooks/format.sh"` |
| `timeout` | 可选的超时时间（秒），默认 60 | `30` |
| `once` | 如果为 `true`，钩子仅在每个会话中运行一次 | `true` |

### Matcher 模式

| 模式 | 说明 | 示例 |
|---------|-------------|---------|
| 精确字符串 | 匹配特定工具 | `"Write"` |
| 正则模式 | 匹配多个工具 | `"Edit\|Write"` |
| 通配符 | 匹配所有工具 | `"*"` 或 `""` |
| MCP 工具 | 服务器和工具模式 | `"mcp__memory__.*"` |

## 钩子类型

Claude Code 支持四种钩子类型：

### 命令钩子

默认钩子类型。通过 JSON stdin/stdout 和退出码进行通信执行 shell 命令。

```json
{
  "type": "command",
  "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate.py\"",
  "timeout": 60
}
```

### HTTP 钩子

> 添加于 v2.1.63。

远程 webhook 端点，接收与命令钩子相同的 JSON 输入。HTTP 钩子 POST JSON 到 URL 并接收 JSON 响应。启用沙盒时，HTTP 钩子通过沙盒路由。为安全起见，URL 中的环境变量插值需要显式的 `allowedEnvVars` 列表。

```json
{
  "hooks": {
    "PostToolUse": [{
      "type": "http",
      "url": "https://my-webhook.example.com/hook",
      "matcher": "Write"
    }]
  }
}
```

**关键属性：**
- `"type": "http"` —— 标识为 HTTP 钩子
- `"url"` —— webhook 端点 URL
- 启用沙盒时通过沙盒路由
- URL 中的任何环境变量插值需要显式的 `allowedEnvVars` 列表

### 提示词钩子

LLM 评估的提示词，其中钩子内容是 Claude 评估的提示词。主要与 `Stop` 和 `SubagentStop` 事件一起使用，用于智能任务完成检查。

```json
{
  "type": "prompt",
  "prompt": "Evaluate if Claude completed all requested tasks.",
  "timeout": 30
}
```

LLM 评估提示词并返回结构化决策（详见 [基于提示词的钩子](#基于提示词的钩子)）。

### 代理钩子

基于子代理的验证钩子，生成专用代理来评估条件或执行复杂检查。与提示词钩子（单轮 LLM 评估）不同，代理钩子可以使用工具并执行多步推理。

```json
{
  "type": "agent",
  "prompt": "Verify the code changes follow our architecture guidelines. Check the relevant design docs and compare.",
  "timeout": 120
}
```

**关键属性：**
- `"type": "agent"` —— 标识为代理钩子
- `"prompt"` —— 子代理的任务描述
- 代理可以使用工具（Read、Grep、Bash 等）执行其评估
- 返回与提示词钩子类似的结构化决策

## 钩子事件

Claude Code 支持 **25 个钩子事件**：

| 事件 | 触发时机 | Matcher 输入 | 可阻塞 | 常见用途 |
|-------|---------------|---------------|-----------|------------|
| **SessionStart** | 会话开始/恢复/清除/压缩 | startup/resume/clear/compact | 否 | 环境设置 |
| **InstructionsLoaded** | CLAUDE.md 或规则文件加载后 | （无） | 否 | 修改/过滤说明 |
| **UserPromptSubmit** | 用户提交提示词时 | （无） | 是 | 验证提示词 |
| **PreToolUse** | 工具执行前 | 工具名称 | 是（allow/deny/ask） | 验证、修改输入 |
| **PermissionRequest** | 显示权限对话框时 | 工具名称 | 是 | 自动批准/拒绝 |
| **PostToolUse** | 工具成功后 | 工具名称 | 否 | 添加上下文、反馈 |
| **PostToolUseFailure** | 工具执行失败时 | 工具名称 | 否 | 错误处理、日志 |
| **Notification** | 发送通知时 | 通知类型 | 否 | 自定义通知 |
| **SubagentStart** | 生成子代理时 | 代理类型名称 | 否 | 子代理设置 |
| **SubagentStop** | 子代理完成时 | 代理类型名称 | 是 | 子代理验证 |
| **Stop** | Claude 响应完成时 | （无） | 是 | 任务完成检查 |
| **StopFailure** | API 错误结束回合时 | （无） | 否 | 错误恢复、日志 |
| **TeammateIdle** | 代理团队队友空闲时 | （无） | 是 | 队友协调 |
| **TaskCompleted** | 任务标记完成时 | （无） | 是 | 任务后操作 |
| **TaskCreated** | 通过 TaskCreate 创建任务时 | （无） | 否 | 任务跟踪、日志 |
| **ConfigChange** | 配置文件更改时 | （无） | 是（策略除外） | 响应配置更新 |
| **CwdChanged** | 工作目录更改时 | （无） | 否 | 目录特定设置 |
| **FileChanged** | 监视的文件更改时 | （无） | 否 | 文件监控、重建 |
| **PreCompact** | 上下文压缩前 | manual/auto | 否 | 压缩前操作 |
| **PostCompact** | 压缩完成后 | （无） | 否 | 压缩后操作 |
| **WorktreeCreate** | 正在创建 worktree 时 | （无） | 是（返回路径） | Worktree 初始化 |
| **WorktreeRemove** | 正在移除 worktree 时 | （无） | 否 | Worktree 清理 |
| **Elicitation** | MCP 服务器请求用户输入时 | （无） | 是 | 输入验证 |
| **ElicitationResult** | 用户响应 elicitation 时 | （无） | 是 | 响应处理 |
| **SessionEnd** | 会话终止时 | （无） | 否 | 清理、最终日志 |

### PreToolUse

在 Claude 创建工具参数后、处理前运行。使用此钩子来验证或修改工具输入。

**配置：**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py"
          }
        ]
      }
    ]
  }
}
```

**常用 matchers：** `Task`、`Bash`、`Glob`、`Grep`、`Read`、`Edit`、`Write`、`WebFetch`、`WebSearch`

**输出控制：**
- `permissionDecision`：`"allow"`、`"deny"` 或 `"ask"`
- `permissionDecisionReason`：决策说明
- `updatedInput`：修改后的工具输入参数

### PostToolUse

在工具完成后立即运行。用于验证、日志记录或向 Claude 提供上下文。

**配置：**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py"
          }
        ]
      }
    ]
  }
}
```

**输出控制：**
- `"block"` 决策向 Claude 提示反馈
- `additionalContext`：添加到 Claude 的上下文

### UserPromptSubmit

在用户提交提示词时、Claude 处理之前运行。

**配置：**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py"
          }
        ]
      }
    ]
  }
}
```

**输出控制：**
- `decision`：`"block"` 阻止处理
- `reason`：如果被阻止则说明原因
- `additionalContext`：添加到提示词的上下文

### Stop 和 SubagentStop

在 Claude 响应完成（Stop）或子代理完成（SubagentStop）时运行。支持基于提示词的评估以进行智能任务完成检查。

**附加输入字段：** `Stop` 和 `SubagentStop` 钩子在其 JSON 输入中接收 `last_assistant_message` 字段，包含 Claude 或子代理在停止之前的最终消息。这对于评估任务完成情况很有用。

**配置：**
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if Claude completed all requested tasks.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### SubagentStart

在子代理开始执行时运行。matcher 输入是代理类型名称，允许钩子针对特定子代理类型。

**配置：**
```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "code-review",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/subagent-init.sh"
          }
        ]
      }
    ]
  }
}
```

### SessionStart

在会话开始或恢复时运行。可以持久化环境变量。

**Matchers：** `startup`、`resume`、`clear`、`compact`

**特殊功能：** 使用 `CLAUDE_ENV_FILE` 来持久化环境变量（也可在 `CwdChanged` 和 `FileChanged` 钩子中使用）：

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=development' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

### SessionEnd

在会话结束时运行以执行清理或最终日志记录。无法阻止终止。

**Reason 字段值：**
- `clear` - 用户清除了会话
- `logout` - 用户登出
- `prompt_input_exit` - 用户通过提示词输入退出
- `other` - 其他原因

**配置：**
```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-cleanup.sh\""
          }
        ]
      }
    ]
  }
}
```

### Notification 事件

更新了通知事件的 matchers：
- `permission_prompt` - 权限请求通知
- `idle_prompt` - 空闲状态通知
- `auth_success` - 认证成功
- `elicitation_dialog` - 显示给用户的对话框

## 组件作用域钩子

钩子可以附加到 frontmatter 中特定组件（技能、代理、命令）：

**在 SKILL.md、agent.md 或 command.md 中：**

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/check.sh"
          once: true  # Only run once per session
---
```

**组件钩子支持的事件：** `PreToolUse`、`PostToolUse`、`Stop`

这允许在与使用它们的组件中直接定义钩子，将相关代码保持在一起。

### 子代理 Frontmatter 中的钩子

当 `Stop` 钩子定义在子代理的 frontmatter 中时，它会自动转换为作用域限定为该子代理的 `SubagentStop` 钩子。这确保该停止钩子仅在该特定子代理完成时触发，而非主会话停止时。

```yaml
---
name: code-review-agent
description: Automated code review subagent
hooks:
  Stop:
    - hooks:
        - type: prompt
          prompt: "Verify the code review is thorough and complete."
  # The above Stop hook auto-converts to SubagentStop for this subagent
---
```

## PermissionRequest 事件

使用自定义输出格式处理权限请求：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow|deny",
      "updatedInput": {},
      "message": "Custom message",
      "interrupt": false
    }
  }
}
```

## 钩子输入和输出

### JSON 输入（通过 stdin）

所有钩子通过 stdin 接收 JSON 输入：

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.js",
    "content": "..."
  },
  "tool_use_id": "toolu_01ABC123...",
  "agent_id": "agent-abc123",
  "agent_type": "main",
  "worktree": "/path/to/worktree"
}
```

**常用字段：**

| 字段 | 说明 |
|-------|-------------|
| `session_id` | 唯一会话标识符 |
| `transcript_path` | 对话记录文件路径 |
| `cwd` | 当前工作目录 |
| `hook_event_name` | 触发钩子的事件名称 |
| `agent_id` | 运行此钩子的代理标识符 |
| `agent_type` | 代理类型（`"main"`、子代理类型名称等） |
| `worktree` | git worktree 路径（如果代理在其中运行） |

### 退出码

| 退出码 | 含义 | 行为 |
|-----------|-----------|----------|
| **0** | 成功 | 继续，解析 JSON stdout |
| **2** | 阻塞性错误 | 阻止操作，stderr 显示为错误 |
| **其他** | 非阻塞性错误 | 继续，stderr 在详细模式显示 |

### JSON 输出（stdout，退出码 0）

```json
{
  "continue": true,
  "stopReason": "Optional message if stopping",
  "suppressOutput": false,
  "systemMessage": "Optional warning message",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "File is in allowed directory",
    "updatedInput": {
      "file_path": "/modified/path.js"
    }
  }
}
```

## 环境变量

| 变量 | 可用性 | 说明 |
|----------|-------------|-------------|
| `CLAUDE_PROJECT_DIR` | 所有钩子 | 项目根目录的绝对路径 |
| `CLAUDE_ENV_FILE` | SessionStart、CwdChanged、FileChanged | 用于持久化环境变量的文件路径 |
| `CLAUDE_CODE_REMOTE` | 所有钩子 | 如果在远程环境中运行则为 `"true"` |
| `${CLAUDE_PLUGIN_ROOT}` | 插件钩子 | 插件目录路径 |
| `${CLAUDE_PLUGIN_DATA}` | 插件钩子 | 插件数据目录路径 |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd 钩子 | SessionEnd 钩子的可配置超时时间（毫秒），覆盖默认值 |

## 基于提示词的钩子

对于 `Stop` 和 `SubagentStop` 事件，可以使用基于 LLM 的评估：

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if all tasks are complete. Return your decision.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**LLM 响应模式：**
```json
{
  "decision": "approve",
  "reason": "All tasks completed successfully",
  "continue": false,
  "stopReason": "Task complete"
}
```

## 示例

### 示例 1：Bash 命令验证器（PreToolUse）

**文件：** `.claude/hooks/validate-bash.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

BLOCKED_PATTERNS = [
    (r"\brm\s+-rf\s+/", "Blocking dangerous rm -rf / command"),
    (r"\bsudo\s+rm", "Blocking sudo rm command"),
]

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    if tool_name != "Bash":
        sys.exit(0)

    command = input_data.get("tool_input", {}).get("command", "")

    for pattern, message in BLOCKED_PATTERNS:
        if re.search(pattern, command):
            print(message, file=sys.stderr)
            sys.exit(2)  # Exit 2 = blocking error

    sys.exit(0)

if __name__ == "__main__":
    main()
```

**配置：**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py\""
          }
        ]
      }
    ]
  }
}
```

### 示例 2：安全扫描器（PostToolUse）

**文件：** `.claude/hooks/security-scan.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

SECRET_PATTERNS = [
    (r"password\s*=\s*['\"][^'\"]+['\"]", "Potential hardcoded password"),
    (r"api[_-]?key\s*=\s*['\"][^'\"]+['\"]", "Potential hardcoded API key"),
]

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    if tool_name not in ["Write", "Edit"]:
        sys.exit(0)

    tool_input = input_data.get("tool_input", {})
    content = tool_input.get("content", "") or tool_input.get("new_string", "")
    file_path = tool_input.get("file_path", "")

    warnings = []
    for pattern, message in SECRET_PATTERNS:
        if re.search(pattern, content, re.IGNORECASE):
            warnings.append(message)

    if warnings:
        output = {
            "hookSpecificOutput": {
                "hookEventName": "PostToolUse",
                "additionalContext": f"Security warnings for {file_path}: " + "; ".join(warnings)
            }
        }
        print(json.dumps(output))

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### 示例 3：自动格式化代码（PostToolUse）

**文件：** `.claude/hooks/format-code.sh`

```bash
#!/bin/bash

# Read JSON from stdin
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | python3 -c "import sys, json; print(json.load(sys.stdin).get('tool_name', ''))")
FILE_PATH=$(echo "$INPUT" | python3 -c "import sys, json; print(json.load(sys.stdin).get('tool_input', {}).get('file_path', ''))")

if [ "$TOOL_NAME" != "Write" ] && [ "$TOOL_NAME" != "Edit" ]; then
    exit 0
fi

# Format based on file extension
case "$FILE_PATH" in
    *.js|*.jsx|*.ts|*.tsx|*.json)
        command -v prettier &>/dev/null && prettier --write "$FILE_PATH" 2>/dev/null
        ;;
    *.py)
        command -v black &>/dev/null && black "$FILE_PATH" 2>/dev/null
        ;;
    *.go)
        command -v gofmt &>/dev/null && gofmt -w "$FILE_PATH" 2>/dev/null
        ;;
esac

exit 0
```

### 示例 4：提示词验证器（UserPromptSubmit）

**文件：** `.claude/hooks/validate-prompt.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

BLOCKED_PATTERNS = [
    (r"delete\s+(all\s+)?database", "Dangerous: database deletion"),
    (r"rm\s+-rf\s+/", "Dangerous: root deletion"),
]

def main():
    input_data = json.load(sys.stdin)
    prompt = input_data.get("user_prompt", "") or input_data.get("prompt", "")

    for pattern, message in BLOCKED_PATTERNS:
        if re.search(pattern, prompt, re.IGNORECASE):
            output = {
                "decision": "block",
                "reason": f"Blocked: {message}"
            }
            print(json.dumps(output))
            sys.exit(0)

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### 示例 5：智能 Stop 钩子（基于提示词）

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if Claude completed all requested tasks. Check: 1) Were all files created/modified? 2) Were there unresolved errors? If incomplete, explain what's missing.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 示例 6：上下文使用追踪器（钩子对）

使用 `UserPromptSubmit`（消息前）和 `Stop`（消息后）钩子一起追踪每个请求的 token 消耗。

**文件：** `.claude/hooks/context-tracker.py`

```python
#!/usr/bin/env python3
"""
Context Usage Tracker - Tracks token consumption per request.

Uses UserPromptSubmit as "pre-message" hook and Stop as "post-response" hook
to calculate the delta in token usage for each request.

Token Counting Methods:
1. Character estimation (default): ~4 chars per token, no dependencies
2. tiktoken (optional): More accurate (~90-95%), requires: pip install tiktoken
"""
import json
import os
import sys
import tempfile

# Configuration
CONTEXT_LIMIT = 128000  # Claude's context window (adjust for your model)
USE_TIKTOKEN = False    # Set True if tiktoken is installed for better accuracy


def get_state_file(session_id: str) -> str:
    """Get temp file path for storing pre-message token count, isolated by session."""
    return os.path.join(tempfile.gettempdir(), f"claude-context-{session_id}.json")


def count_tokens(text: str) -> int:
    """
    Count tokens in text.

    Uses tiktoken with p50k_base encoding if available (~90-95% accuracy),
    otherwise falls back to character estimation (~80-90% accuracy).
    """
    if USE_TIKTOKEN:
        try:
            import tiktoken
            enc = tiktoken.get_encoding("p50k_base")
            return len(enc.encode(text))
        except ImportError:
            pass  # Fall back to estimation

    # Character-based estimation: ~4 characters per token for English
    return len(text) // 4


def read_transcript(transcript_path: str) -> str:
    """Read and concatenate all content from transcript file."""
    if not transcript_path or not os.path.exists(transcript_path):
        return ""

    content = []
    with open(transcript_path, "r") as f:
        for line in f:
            try:
                entry = json.loads(line.strip())
                # Extract text content from various message formats
                if "message" in entry:
                    msg = entry["message"]
                    if isinstance(msg.get("content"), str):
                        content.append(msg["content"])
                    elif isinstance(msg.get("content"), list):
                        for block in msg["content"]:
                            if isinstance(block, dict) and block.get("type") == "text":
                                content.append(block.get("text", ""))
            except json.JSONDecodeError:
                continue

    return "\n".join(content)


def handle_user_prompt_submit(data: dict) -> None:
    """Pre-message hook: Save current token count before request."""
    session_id = data.get("session_id", "unknown")
    transcript_path = data.get("transcript_path", "")

    transcript_content = read_transcript(transcript_path)
    current_tokens = count_tokens(transcript_content)

    # Save to temp file for later comparison
    state_file = get_state_file(session_id)
    with open(state_file, "w") as f:
        json.dump({"pre_tokens": current_tokens}, f)


def handle_stop(data: dict) -> None:
    """Post-response hook: Calculate and report token delta."""
    session_id = data.get("session_id", "unknown")
    transcript_path = data.get("transcript_path", "")

    transcript_content = read_transcript(transcript_path)
    current_tokens = count_tokens(transcript_content)

    # Load pre-message count
    state_file = get_state_file(session_id)
    pre_tokens = 0
    if os.path.exists(state_file):
        try:
            with open(state_file, "r") as f:
                state = json.load(f)
                pre_tokens = state.get("pre_tokens", 0)
        except (json.JSONDecodeError, IOError):
            pass

    # Calculate delta
    delta_tokens = current_tokens - pre_tokens
    remaining = CONTEXT_LIMIT - current_tokens
    percentage = (current_tokens / CONTEXT_LIMIT) * 100

    # Report usage
    method = "tiktoken" if USE_TIKTOKEN else "estimated"
    print(f"Context ({method}): ~{current_tokens:,} tokens ({percentage:.1f}% used, ~{remaining:,} remaining)", file=sys.stderr)
    if delta_tokens > 0:
        print(f"This request: ~{delta_tokens:,} tokens", file=sys.stderr)


def main():
    data = json.load(sys.stdin)
    event = data.get("hook_event_name", "")

    if event == "UserPromptSubmit":
        handle_user_prompt_submit(data)
    elif event == "Stop":
        handle_stop(data)

    sys.exit(0)


if __name__ == "__main__":
    main()
```

**配置：**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/context-tracker.py\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/context-tracker.py\""
          }
        ]
      }
    ]
  }
}
```

**工作原理：**
1. `UserPromptSubmit` 在处理提示词前触发——保存当前 token 计数
2. `Stop` 在 Claude 响应后触发——计算 delta 并报告使用情况
3. 每个会话通过 temp 文件名中的 `session_id` 隔离

**Token 计数方法：**

| 方法 | 准确性 | 依赖 | 速度 |
|--------|----------|--------------|-------|
| 字符估算 | ~80-90% | 无 | <1ms |
| tiktoken (p50k_base) | ~90-95% | `pip install tiktoken` | <10ms |

> **注意：** Anthropic 尚未发布官方离线分词器。两种方法都是近似值。记录包含用户提示词、Claude 的响应和工具输出，但不包含系统提示词或内部上下文。

### 示例 7：种子 Auto Mode 权限（一次性设置脚本）

一次性设置脚本，用约 67 条安全权限规则种子 `~/.claude/settings.json`，相当于 Claude Code 的 auto-mode 基线——无需任何钩子，无需记住未来选择。运行一次；可安全重新运行（跳过已存在的规则）。

**文件：** `09-advanced-features/setup-auto-mode-permissions.py`

```bash
# Preview what would be added
python3 09-advanced-features/setup-auto-mode-permissions.py --dry-run

# Apply
python3 09-advanced-features/setup-auto-mode-permissions.py
```

**添加的内容：**

| 类别 | 示例 |
|----------|---------|
| 内置工具 | `Read(*)`、`Edit(*)`、`Write(*)`、`Glob(*)`、`Grep(*)`、`Agent(*)`、`WebSearch(*)` |
| Git 读取 | `Bash(git status:*)`、`Bash(git log:*)`、`Bash(git diff:*)` |
| Git 写入（本地） | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git checkout:*)` |
| 包管理器 | `Bash(npm install:*)`、`Bash(pip install:*)`、`Bash(cargo build:*)` |
| 构建和测试 | `Bash(make:*)`、`Bash(pytest:*)`、`Bash(go test:*)` |
| 常用 shell | `Bash(ls:*)`、`Bash(cat:*)`、`Bash(find:*)`、`Bash(cp:*)`、`Bash(mv:*)` |
| GitHub CLI | `Bash(gh pr view:*)`、`Bash(gh pr create:*)`、`Bash(gh issue list:*)` |

**有意排除的内容**（此脚本永不添加）：
- `rm -rf`、`sudo`、force push、`git reset --hard`
- `DROP TABLE`、`kubectl delete`、`terraform destroy`
- `npm publish`、`curl | bash`、生产部署

## 插件钩子

插件可以在其 `hooks/hooks.json` 文件中包含钩子：

**文件：** `plugins/hooks/hooks.json`

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

**插件钩子中的环境变量：**
- `${CLAUDE_PLUGIN_ROOT}` - 插件目录路径
- `${CLAUDE_PLUGIN_DATA}` - 插件数据目录路径

这允许插件包含自定义验证和自动化钩子。

## MCP 工具钩子

MCP 工具遵循模式 `mcp__<server>__<tool>`：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"systemMessage\": \"Memory operation logged\"}'"
          }
        ]
      }
    ]
  }
}
```

## 安全注意事项

### 免责声明

**使用风险自负**：钩子执行任意 shell 命令。你全权负责：
- 你配置的命令
- 文件访问/修改权限
- 潜在的数据丢失或系统损坏
- 在生产环境使用前在安全环境中测试钩子

### 安全说明

- **需要工作区信任：** `statusLine` 和 `fileSuggestion` 钩子输出命令现在需要在生效前接受工作区信任。
- **HTTP 钩子和环境变量：** HTTP 钩子需要显式的 `allowedEnvVars` 列表才能在 URL 中使用环境变量插值。这可以防止敏感环境变量意外泄露到远程端点。
- **托管设置层次结构：** `disableAllHooks` 设置现在遵循托管设置层次结构，意味着组织级设置可以强制执行个人用户无法覆盖的钩子禁用。

### 最佳实践

| 宜 | 不宜 |
|-----|-------|
| 验证和清理所有输入 | 盲目信任输入数据 |
| 引用 shell 变量：`"$VAR"` | 不加引用：`$VAR` |
| 阻止路径遍历（`..`） | 允许任意路径 |
| 使用 `$CLAUDE_PROJECT_DIR` 的绝对路径 | 硬编码路径 |
| 跳过敏感文件（`.env`、`.git/`、密钥） | 处理所有文件 |
| 首先在隔离环境中测试钩子 | 部署未经测试的钩子 |
| HTTP 钩子使用显式 `allowedEnvVars` | 向 webhooks 暴露所有环境变量 |

## 调试

### 启用调试模式

使用调试标志运行 Claude 以获取详细的钩子日志：

```bash
claude --debug
```

### 详细模式

在 Claude Code 中使用 `Ctrl+O` 启用详细模式，查看钩子执行进度。

### 独立测试钩子

```bash
# Test with sample JSON input
echo '{"tool_name": "Bash", "tool_input": {"command": "ls -la"}}' | python3 .claude/hooks/validate-bash.py

# Check exit code
echo $?
```

## 完整配置示例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/format-code.sh\"",
            "timeout": 30
          },
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-init.sh\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify all tasks are complete before stopping.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## 钩子执行详情

| 方面 | 行为 |
|--------|----------|
| **超时** | 默认 60 秒，可为每个命令配置 |
| **并行化** | 所有匹配的钩子并行运行 |
| **去重** | 相同的钩子命令去重 |
| **环境** | 在当前目录和 Claude Code 的环境中运行 |

## 故障排除

### 钩子未执行
- 验证 JSON 配置语法正确
- 检查 matcher 模式是否匹配工具名称
- 确保脚本存在且可执行：`chmod +x script.sh`
- 运行 `claude --debug` 查看钩子执行日志
- 验证钩子从 stdin 读取 JSON（而非命令参数）

### 钩子意外阻塞
- 使用示例 JSON 测试钩子：`echo '{"tool_name": "Write", ...}' | ./hook.py`
- 检查退出码：0 表示允许，2 表示阻塞
- 检查 stderr 输出（退出码 2 时显示）

### JSON 解析错误
- 始终从 stdin 读取，而非命令参数
- 使用正确的 JSON 解析（而非字符串操作）
- 优雅地处理缺失字段

## 安装

### 步骤 1：创建钩子目录
```bash
mkdir -p ~/.claude/hooks
```

### 步骤 2：复制示例钩子
```bash
cp 06-hooks/*.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/*.sh
```

### 步骤 3：在设置中配置
在 `~/.claude/settings.json` 或 `.claude/settings.json` 中编辑上面显示的钩子配置。

## 相关概念

- **[检查点和回退](../08-checkpoints/)** - 保存和恢复对话状态
- **[斜杠命令](../01-slash-commands/)** - 创建自定义斜杠命令
- **[技能](../03-skills/)** - 可重用的自主能力
- **[子代理](../04-subagents/)** - 委托任务执行
- **[插件](../07-plugins/)** - 捆绑扩展包
- **[高级功能](../09-advanced-features/)** - 探索 Claude Code 的高级功能

## 其他资源

- **[Official Hooks Documentation](https://code.claude.com/docs/en/hooks)** - 完整的钩子参考
- **[CLI Reference](https://code.claude.com/docs/en/cli-reference)** - 命令行接口文档
- **[Memory Guide](../02-memory/)** - 持久化上下文配置
