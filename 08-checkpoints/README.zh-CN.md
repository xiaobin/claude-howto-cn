<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 检查点和回退

检查点允许你保存对话状态并回退到 Claude Code 会话中的先前点。这对于探索不同方法、从错误中恢复或比较替代解决方案非常宝贵。

## 概述

检查点允许你保存对话状态并回退到先前点，启用安全的实验和对多种方法的探索。它们是你对话状态的快照，包括：
- 所有交换的消息
- 进行的文件修改
- 工具使用历史
- 会话上下文

检查点在探索不同方法、从错误中恢复或比较替代解决方案时非常宝贵。

## 关键概念

| 概念 | 说明 |
|---------|-------------|
| **检查点** | 对话状态的快照，包括消息、文件和上下文 |
| **回退** | 返回先前的检查点，丢弃后续更改 |
| **分支点** | 从中可以探索多种方法的检查点 |

## 访问检查点

你可以通过两种主要方式访问和管理检查点：

### 使用键盘快捷键
按两次 `Esc`（`Esc` + `Esc`）打开检查点界面并浏览已保存的检查点。

### 使用斜杠命令
使用 `/rewind` 命令（别名：`/checkpoint`）快速访问：

```bash
# Open rewind interface
/rewind

# Or use the alias
/checkpoint
```

## 回退选项

当你回退时，会看到五个选项的菜单：

1. **恢复代码和对话** —— 将文件和消息都恢复到该检查点
2. **恢复对话** —— 仅回退消息，保持当前代码不变
3. **恢复代码** —— 仅恢复文件更改，保持完整的对话历史
4. **从这里总结** —— 将从该点往后的对话压缩成 AI 生成的摘要，而不是丢弃。原始消息保留在记录中。你可以选择性地提供指令，聚焦于特定主题的总结。
5. **算了** —— 取消并返回当前状态

## 自动检查点

Claude Code 自动为你创建检查点：

- **每个用户提示词** —— 每次用户输入时创建一个新检查点
- **持久化** —— 检查点跨会话持久化
- **自动清理** —— 检查点在 30 天后自动清理

这意味着你随时可以回退到对话中的任何先前点，从几分钟前到几天前。

## 使用场景

| 场景 | 工作流 |
|----------|----------|
| **探索方法** | 保存 → 尝试 A → 保存 → 回退 → 尝试 B → 比较 |
| **安全重构** | 保存 → 重构 → 测试 → 如果失败：回退 |
| **A/B 测试** | 保存 → 设计 A → 保存 → 回退 → 设计 B → 比较 |
| **错误恢复** | 发现问题 → 回退到最后的好状态 |

## 使用检查点

### 查看和回退

按两次 `Esc` 或使用 `/rewind` 打开检查点浏览器。你将看到所有可用检查点的列表及时间戳。选择任何检查点以回退到该状态。

### 检查点详情

每个检查点显示：
- 创建时间戳
- 修改的文件
- 对话中的消息数量
- 使用的工具

## 实用示例

### 示例 1：探索不同方法

```
User: Let's add a caching layer to the API

Claude: I'll add Redis caching to your API endpoints...
[Makes changes at checkpoint A]

User: Actually, let's try in-memory caching instead

Claude: I'll rewind to explore a different approach...
[User presses Esc+Esc and rewinds to checkpoint A]
[Implements in-memory caching at checkpoint B]

User: Now I can compare both approaches
```

### 示例 2：从错误中恢复

```
User: Refactor the authentication module to use JWT

Claude: I'll refactor the authentication module...
[Makes extensive changes]

User: Wait, that broke the OAuth integration. Let's go back.

Claude: I'll help you rewind to before the refactoring...
[User presses Esc+Esc and selects the checkpoint before the refactor]

User: Let's try a more conservative approach this time
```

### 示例 3：安全实验

```
User: Let's try rewriting this in a functional style
[Creates checkpoint before experiment]

Claude: [Makes experimental changes]

User: The tests are failing. Let's rewind.
[User presses Esc+Esc and rewinds to the checkpoint]

Claude: I've rewound the changes. Let's try a different approach.
```

### 示例 4：分支方法

```
User: I want to compare two database designs
[Takes note of checkpoint - call it "Start"]

Claude: I'll create the first design...
[Implements Schema A]

User: Now let me go back and try the second approach
[User presses Esc+Esc and rewinds to "Start"]

Claude: Now I'll implement Schema B...
[Implements Schema B]

User: Great! Now I have both schemas to choose from
```

## 检查点保留

Claude Code 自动管理你的检查点：

- 检查点随每个用户提示词自动创建
- 旧检查点保留最多 30 天
- 检查点自动清理以防止无限存储增长

## 工作流模式

### 探索分支策略

探索多种方法时：

```
1. Start with initial implementation → Checkpoint A
2. Try Approach 1 → Checkpoint B
3. Rewind to Checkpoint A
4. Try Approach 2 → Checkpoint C
5. Compare results from B and C
6. Choose best approach and continue
```

### 安全重构模式

进行重大更改时：

```
1. Current state → Checkpoint (auto)
2. Start refactoring
3. Run tests
4. If tests pass → Continue working
5. If tests fail → Rewind and try different approach
```

## 最佳实践

由于检查点是自动创建的，你可以专注于工作，而不用担心手动保存状态。但请记住以下实践：

### 有效使用检查点

✅ **做：**
- 回退前查看可用的检查点
- 当你想探索不同方向时使用回退
- 保留检查点以比较不同方法
- 了解每个回退选项的作用（恢复代码和对话、恢复对话、恢复代码或总结）

❌ **不要：**
- 仅依赖检查点来保存代码
- 期望检查点跟踪外部文件系统更改
- 使用检查点作为版本控制的替代品

## 配置

你可以在设置中切换自动检查点：

```json
{
  "autoCheckpoint": true
}
```

- `autoCheckpoint`：启用或禁用每个用户提示词时自动创建检查点（默认：`true`）

## 限制

检查点有以下限制：

- **Bash 命令更改不跟踪** —— 文件系统上的 `rm`、`mv`、`cp` 等操作不会在检查点中捕获
- **外部更改不跟踪** —— 在 Claude Code 外部（编辑器、终端等）进行的更改不会被捕获
- **不能替代版本控制** —— 使用 git 进行代码库的永久、可审计更改

## 故障排除

### 缺少检查点

**问题**：找不到预期的检查点

**解决方案**：
- 检查检查点是否被清除
- 验证 `autoCheckpoint` 在设置中已启用
- 检查磁盘空间

### 回退失败

**问题**：无法回退到检查点

**解决方案**：
- 确保没有未提交的更改冲突
- 检查检查点是否损坏
- 尝试回退到另一个检查点

## 与 Git 集成

检查点补充（但不替代）git：

| 功能 | Git | 检查点 |
|---------|-----|-------------|
| 范围 | 文件系统 | 对话 + 文件 |
| 持久性 | 永久 | 基于会话 |
| 粒度 | 提交 | 任意点 |
| 速度 | 较慢 | 即时 |
| 共享 | 是 | 有限 |

两者一起使用：
1. 使用检查点进行快速实验
2. 使用 git 提交进行最终确定的更改
3. git 操作前创建检查点
4. 将成功的检查点状态提交到 git

## 快速入门指南

### 基本工作流

1. **正常工作** —— Claude Code 自动创建检查点
2. **想回去？** —— 按两次 `Esc` 或使用 `/rewind`
3. **选择检查点** —— 选择要回退的列表
4. **选择恢复什么** —— 选择恢复代码和对话、恢复对话、恢复代码、从这里总结，或取消
5. **继续工作** —— 你已回到该点

### 键盘快捷键

- **`Esc` + `Esc`** —— 打开检查点浏览器
- **`/rewind`** —— 访问检查点的另一种方式
- **`/checkpoint`** —— `/rewind` 的别名

## 何时回退：上下文监控

检查点可以让你回去——但你怎么知道什么时候应该？随着对话增长，Claude 的上下文窗口会填满，模型质量会悄然下降。你可能在不知不觉中从一个半盲模型发送代码。

**[cc-context-stats](https://github.com/luongnv89/cc-context-stats)** 通过向 Claude Code 状态栏添加实时**上下文区域**来解决这个问题。它追踪你在上下文窗口中的位置——从**Plan**（绿色，安全进行规划和编码）到**Code**（黄色，避免开始新计划）再到**Dump**（橙色，完成并回退）。当你看到区域变化时，就知道是时候创建检查点并重新开始了，而不是用降级的输出继续。

## 相关概念

- **[高级功能](../09-advanced-features/)** —— 规划模式和其他高级能力
- **[内存管理](../02-memory/)** —— 管理对话历史和上下文
- **[斜杠命令](../01-slash-commands/)** —— 用户调用的快捷方式
- **[钩子](../06-hooks/)** —— 事件驱动的自动化
- **[插件](../07-plugins/)** —— 捆绑扩展包

## 其他资源

- [Official Checkpointing Documentation](https://code.claude.com/docs/en/checkpointing)
- [Advanced Features Guide](../09-advanced-features/) - Extended thinking and other capabilities

## 总结

检查点是 Claude Code 中的自动功能，让你安全地探索不同方法而不必担心丢失工作。每个用户提示词都会自动创建新检查点，因此你可以回退到会话中的任何先前点。

主要优势：
- 无顾虑地尝试多种方法进行实验
- 快速从错误中恢复
- 并排比较不同的解决方案
- 与版本控制系统安全集成

记住：检查点不能替代 git。使用检查点进行快速实验，使用 git 进行永久代码更改。
