---
name: Setup CI/CD Pipeline
description: 实现 pre-commit 钩子和 GitHub Actions 进行质量保证
tags: ci-cd, devops, automation
---

# Setup CI/CD Pipeline

根据项目类型实施全面的 DevOps 质量门：

1. **分析项目**：检测语言、框架、构建系统和现有工具
2. **配置 pre-commit 钩子**，使用语言特定工具：
   - 格式化：Prettier/Black/gofmt/rustfmt 等
   - 检查：ESLint/Ruff/golangci-lint/Clippy 等
   - 安全：Bandit/gosec/cargo-audit/npm audit 等
   - 类型检查：TypeScript/mypy/flow（如果适用）
   - 测试：运行相关测试套件
3. **创建 GitHub Actions 工作流**（.github/workflows/）：
   - 在 push/PR 时镜像 pre-commit 检查
   - 多版本/平台矩阵（如果适用）
   - 构建和测试验证
   - 部署步骤（如果需要）
4. **验证流水线**：本地测试、创建测试 PR、确认所有检查通过

使用免费/开源工具。尊重现有配置。保持执行快速。
