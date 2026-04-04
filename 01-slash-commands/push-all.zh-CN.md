---
description: 暂存所有更改、创建提交并推送到远程（谨慎使用）
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*), Bash(git push:*), Bash(git diff:*), Bash(git log:*), Bash(git pull:*)
---

# Commit and Push Everything

⚠️ **警告**：暂存所有更改、提交并推送到远程。仅在确信所有更改属于同一批次时使用。

## 工作流

### 1. 分析更改
并行运行：
- `git status` — 显示已修改/已添加/已删除/未跟踪文件
- `git diff --stat` — 显示更改统计
- `git log -1 --oneline` — 显示最近提交以了解消息风格

### 2. 安全检查

**❌ 如果检测到以下内容则停止并警告：**
- 密钥：`.env*`、`*.key`、`*.pem`、`credentials.json`、`secrets.yaml`、`id_rsa`、`*.p12`、`*.pfx`、`*.cer`
- API 密钥：任何含有真实值的 `*_API_KEY`、`*_SECRET`、`*_TOKEN` 变量（非占位符如 `your-api-key`、`xxx`、`placeholder`）
- 大文件：大于 10MB 且无 Git LFS
- 构建产物：`node_modules/`、`dist/`、`build/`、`__pycache__/`、`*.pyc`、`.venv/`
- 临时文件：`.DS_Store`、`thumbs.db`、`*.swp`、`*.tmp`

**API 密钥验证：**
检查已修改文件中的模式：
```bash
OPENAI_API_KEY=sk-proj-xxxxx  # ❌ 检测到真实密钥！
AWS_SECRET_KEY=AKIA...         # ❌ 检测到真实密钥！
STRIPE_API_KEY=sk_live_...    # ❌ 检测到真实密钥！

# ✅ 可接受的占位符：
API_KEY=your-api-key-here
SECRET_KEY=placeholder
TOKEN=xxx
API_KEY=<your-key>
SECRET=${YOUR_SECRET}
```

**✅ 验证：**
- `.gitignore` 配置正确
- 无合并冲突
- 分支正确（如果是 main/master 则警告）
- API 密钥仅为占位符

### 3. 请求确认

展示摘要：
```
📊 Changes Summary:
- X files modified, Y added, Z deleted
- Total: +AAA insertions, -BBB deletions

🔒 Safety: ✅ No secrets | ✅ No large files | ⚠️ [warnings]
🌿 Branch: [name] → origin/[name]

I will: git add . → commit → push

Type 'yes' to proceed or 'no' to cancel.
```

**等待明确的"yes"后再继续。**

### 4. 执行（确认后）

按顺序运行：
```bash
git add .
git status  # Verify staging
```

### 5. 生成提交消息

分析更改并创建约定式提交：

**格式：**
```
[type]: Brief summary (max 72 characters)

- Key change 1
- Key change 2
- Key change 3
```

**类型：** `feat`、`fix`、`docs`、`style`、`refactor`、`test`、`chore`、`perf`、`build`、`ci`

**示例：**
```
docs: Update concept README files with comprehensive documentation

- Add architecture diagrams and tables
- Include practical examples
- Expand best practices sections
```

### 6. 提交并推送

```bash
git commit -m "$(cat <<'EOF'
[Generated commit message]
EOF
)"
git push  # If fails: git pull --rebase && git push
git log -1 --oneline --decorate  # Verify
```

### 7. 确认成功

```
✅ Successfully pushed to remote!

Commit: [hash] [message]
Branch: [branch] → origin/[branch]
Files changed: X (+insertions, -deletions)
```

## 错误处理

- **git add 失败**：检查权限、锁定文件、验证仓库已初始化
- **git commit 失败**：修复 pre-commit 钩子、检查 git 配置（user.name/email）
- **git push 失败**：
  - 非快进：`git pull --rebase && git push`
  - 无远程分支：`git push -u origin [branch]`
  - 受保护分支：改用 PR 工作流

## 使用时机

✅ **适合：**
- 多文件文档更新
- 带测试和文档的功能
- 跨文件的 bug 修复
- 项目级格式化/重构
- 配置更改

❌ **避免：**
- 不确定正在提交什么
- 包含密钥/敏感数据
- 受保护分支且无审查
- 存在合并冲突
- 想要细粒度的提交历史
- pre-commit 钩子失败

## 替代方案

如果用户想要控制，建议：
1. **选择性暂存**：审查/暂存特定文件
2. **交互式暂存**：`git add -p` 选择补丁
3. **PR 工作流**：创建分支 → 推送 → PR（使用 `/pr` 命令）

**⚠️ 记住**：推送前始终审查更改。不确定时，使用单独的 git 命令以获得更多控制。
