---
description: 清理代码、暂存更改并准备 Pull Request
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git diff:*), Bash(npm test:*), Bash(npm run lint:*)
---

# Pull Request Preparation Checklist

创建 PR 前，执行以下步骤：

1. 运行格式化：`prettier --write .`
2. 运行测试：`npm test`
3. 审查 git diff：`git diff HEAD`
4. 暂存更改：`git add .`
5. 遵循约定式提交创建提交消息：
   - `fix:` bug 修复
   - `feat:` 新功能
   - `docs:` 文档
   - `refactor:` 代码重构
   - `test:` 测试添加
   - `chore:` 维护

6. 生成 PR 摘要，包括：
   - 更改了什么
   - 为什么更改
   - 执行的测试
   - 潜在影响
