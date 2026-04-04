---
name: Documentation Refactor
description: 重构项目文档以提高清晰度和可访问性
tags: documentation, refactoring, organization
---

# Documentation Refactor

重构项目文档结构，适应项目类型：

1. **分析项目**：识别类型（库/API/Web 应用/CLI/微服务）、架构和用户画像
2. **集中文档**：将技术文档移至 `docs/`，并做好交叉引用
3. **根目录 README.md**：精简为入口点，包含概述、快速开始、模块/组件摘要、许可证、联系人
4. **组件文档**：为每个模块/包/服务添加 README 文件，包含设置和测试说明
5. **组织 `docs/`**：按相关类别分类：
   - Architecture、API Reference、Database、Design、Troubleshooting、Deployment、Contributing（根据项目需要调整）
6. **创建指南**（选择适用的）：
   - User Guide：面向最终用户的应用文档
   - API Documentation：面向 API 的端点、认证、示例文档
   - Development Guide：设置、测试、贡献工作流
   - Deployment Guide：服务/应用的生产部署
7. **使用 Mermaid** 绘制所有图表（架构、流程、模式）

保持文档简洁、可扫描，并契合项目类型。
