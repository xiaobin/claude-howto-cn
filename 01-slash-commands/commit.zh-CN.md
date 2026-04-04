---
description: 创建带有上下文的 git 提交
---

## Context

- 当前 git 状态：!`git status`
- 当前 git diff：!`git diff HEAD`
- 当前分支：!`git branch --show-current`
- 最近提交：!`git log --oneline -10`

## 你的任务

根据以上更改，创建单个 git 提交。

如果通过参数提供了消息，使用它：$ARGUMENTS

否则，分析更改并遵循约定式提交格式创建合适的提交消息：
- `feat:` 新功能
- `fix:` bug 修复
- `docs:` 文档更改
- `refactor:` 代码重构
- `test:` 添加测试
- `chore:` 维护任务
