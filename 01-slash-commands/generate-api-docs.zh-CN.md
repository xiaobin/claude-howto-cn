---
description: 从源代码创建全面的 API 文档
---

# API Documentation Generator

按以下步骤生成 API 文档：

1. 扫描 `/src/api/` 中的所有文件
2. 提取函数签名和 JSDoc 注释
3. 按端点/模块组织
4. 创建带示例的 Markdown 文档
5. 包含请求/响应模式
6. 添加错误文档

输出格式：
- 在 `/docs/api.md` 生成 Markdown 文件
- 为所有端点包含 curl 示例
- 添加 TypeScript 类型
