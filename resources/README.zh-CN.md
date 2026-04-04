<picture>
  <source media="(prefers-color-scheme: dark)" srcset="logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="logos/claude-howto-logo.svg">
</picture>

# Claude How To - 品牌资产

Claude How To 项目的完整 Logo、图标和网站图标集合。所有资产均采用 V3.0 设计：指南针加代码括号（`>`）符号，代表在代码中的引导式导航 — 使用黑/白/灰配色方案，搭配亮绿（#22C55E）作为强调色。

## 目录结构

```
resources/
├── logos/
│   ├── claude-howto-logo.svg       # 主 Logo - 浅色模式 (520×120px)
│   └── claude-howto-logo-dark.svg  # 主 Logo - 深色模式 (520×120px)
├── icons/
│   ├── claude-howto-icon.svg       # 应用图标 - 浅色模式 (256×256px)
│   └── claude-howto-icon-dark.svg  # 应用图标 - 深色模式 (256×256px)
└── favicons/
    ├── favicon-16.svg              # 网站图标 - 16×16px
    ├── favicon-32.svg              # 网站图标 - 32×32px（主要）
    ├── favicon-64.svg              # 网站图标 - 64×64px
    ├── favicon-128.svg             # 网站图标 - 128×128px
    └── favicon-256.svg             # 网站图标 - 256×256px
```

`assets/logo/` 中的附加资产：
```
assets/logo/
├── logo-full.svg       # 标识 + 文字组合（水平）
├── logo-mark.svg       # 仅指南针符号（120×120px）
├── logo-wordmark.svg   # 仅文字
├── logo-icon.svg       # 应用图标（512×512，圆角）
├── favicon.svg         # 16×16 优化版
├── logo-white.svg      # 深色背景白色版本
└── logo-black.svg      # 黑色单色版本
```

## 资产概览

### 设计概念（V3.0）

**指南针加代码括号** — 引导与代码相遇：
- **指南针环** = 导航、寻找方向
- **北指针（绿色）** = 方向、学习路上的进步
- **南指针（黑色）** = 根基、坚实基础
- **`>` 括号** = 终端提示符、代码、CLI 上下文
- **刻度线** = 精度、结构化学习

### Logo

**文件：**
- `logos/claude-howto-logo.svg`（浅色模式）
- `logos/claude-howto-logo-dark.svg`（深色模式）

**规格：**
- **尺寸**：520×120 px
- **用途**：主头部/品牌 Logo 配文字
- **使用场景：**
  - 网站头部
  - README 徽章
  - 营销材料
  - 印刷材料
- **格式**：SVG（完全可缩放）
- **模式**：浅色（白色背景）和深色（#0A0A0A 背景）

### 图标

**文件：**
- `icons/claude-howto-icon.svg`（浅色模式）
- `icons/claude-howto-icon-dark.svg`（深色模式）

**规格：**
- **尺寸**：256×256 px
- **用途**：应用图标、头像缩略图
- **使用场景：**
  - 应用图标
  - 头像
  - 社交媒体缩略图
  - 文档头部
- **格式**：SVG（完全可缩放）
- **模式**：浅色（白色背景）和深色（#0A0A0A 背景）

**设计元素：**
- 带方位和中间方位刻度线的指南针环
- 绿色北指针（方向/引导）
- 黑色南指针（根基）
- 中央 `>` 代码括号（终端/CLI）
- 绿色圆点强调

### 网站图标（Favicon）

针对网页使用优化的多尺寸版本：

| 文件 | 尺寸 | DPI | 用途 |
|------|------|-----|------|
| `favicon-16.svg` | 16×16 px | 1x | 浏览器标签（老版本浏览器） |
| `favicon-32.svg` | 32×32 px | 1x | 标准浏览器网站图标 |
| `favicon-64.svg` | 64×64 px | 1x-2x | 高 DPI 显示屏 |
| `favicon-128.svg` | 128×128 px | 2x | Apple 触摸图标、书签 |
| `favicon-256.svg` | 256×256 px | 4x | 现代浏览器、PWA 图标 |

**优化说明：**
- 16px：最小几何图形 — 仅环、指针、箭头
- 32px：添加方位刻度线
- 64px+：完整细节，包含中间方位刻度
- 所有版本保持与主图标视觉一致
- SVG 格式确保任意尺寸清晰显示

## HTML 集成

### 基础网站图标设置

```html
<!-- 浏览器网站图标 -->
<link rel="icon" type="image/svg+xml" href="/resources/favicons/favicon-32.svg">
<link rel="icon" type="image/svg+xml" href="/resources/favicons/favicon-16.svg" sizes="16x16">

<!-- Apple 触摸图标（移动设备主屏幕） -->
<link rel="apple-touch-icon" href="/resources/favicons/favicon-128.svg">

<!-- PWA 和现代浏览器 -->
<link rel="icon" type="image/svg+xml" href="/resources/favicons/favicon-256.svg" sizes="256x256">
```

### 完整设置

```html
<head>
  <!-- 主要网站图标 -->
  <link rel="icon" type="image/svg+xml" href="/resources/favicons/favicon-32.svg" sizes="32x32">
  <link rel="icon" type="image/svg+xml" href="/resources/favicons/favicon-16.svg" sizes="16x16">

  <!-- Apple 触摸图标 -->
  <link rel="apple-touch-icon" href="/resources/favicons/favicon-128.svg">

  <!-- PWA 图标 -->
  <link rel="icon" type="image/svg+xml" href="/resources/favicons/favicon-256.svg" sizes="256x256">

  <!-- Android -->
  <link rel="shortcut icon" href="/resources/favicons/favicon-256.svg">

  <!-- PWA manifest 引用（如果使用 manifest.json）-->
  <meta name="theme-color" content="#000000">
</head>
```

## 配色方案

### 主色
- **黑色**：`#000000`（主文本、描边、南指针）
- **白色**：`#FFFFFF`（浅色背景）
- **灰色**：`#6B7280`（次要文本、次要刻度线）

### 强调色
- **亮绿**：`#22C55E`（北指针、圆点、强调线 — 仅用于高亮，禁止作为背景色）

### 深色模式
- **背景**：`#0A0A0A`（近黑）

### CSS 变量
```css
--color-primary: #000000;
--color-secondary: #6B7280;
--color-accent: #22C55E;
--color-bg-light: #FFFFFF;
--color-bg-dark: #0A0A0A;
```

### Tailwind 配置
```js
colors: {
  brand: {
    primary: '#000000',
    secondary: '#6B7280',
    accent: '#22C55E',
  }
}
```

### 使用指南
- 使用黑色作为主文本和结构元素
- 使用灰色作为次要/支持元素
- **仅将绿色用于高亮** — 指针、圆点、强调线
- 永远不要将绿色作为背景色
- 保持 WCAG AA 对比度（4.5:1 最低）

## 设计指南

### Logo 使用
- 用于白色或深色（#0A0A0A）背景
- 按比例缩放
- Logo 周围包含留白（最小值：Logo 高度 / 2）
- 根据背景使用提供的浅色/深色变体

### 图标使用
- 使用标准尺寸：16、32、64、128、256px
- 保持指南针比例
- 按比例缩放

### 网站图标使用
- 根据场景使用适当尺寸
- 16-32px：浏览器标签、书签
- 64px：网站图标
- 128px+：Apple/Android 主屏幕

## SVG 优化

所有 SVG 文件采用扁平设计，无渐变或滤镜：
- 干净的基于描边的几何图形
- 无嵌入式光栅
- 优化路径
- 响应式 viewBox

网页优化方式：
```bash
# 在保持质量的同时压缩 SVG
svgo --config='{
  "js2svg": {
    "indent": 2
  },
  "plugins": [
    "convertStyleToAttrs",
    "removeRasterImages"
  ]
}' input.svg -o output.svg
```

## PNG 转换

将 SVG 转换为 PNG 以支持旧版浏览器：

```bash
# 使用 ImageMagick
convert -density 300 -background none favicon-256.svg favicon-256.png

# 使用 Inkscape
inkscape -D -z --file=favicon-256.svg --export-png=favicon-256.png
```

## 无障碍

- 高对比度颜色比率（WCAG AA 合规 — 最低 4.5:1）
- 任意尺寸都可识别的干净几何形状
- 可缩放矢量格式
- 图标中无文字（文字在字标中单独添加）
- 无红绿颜色依赖来传达含义

## 署名

这些资产是 Claude How To 项目的一部分。

**许可证**：MIT（参见项目 LICENSE 文件）

## 版本历史

- **v3.0**（2026 年 2 月）：指南针括号设计，黑/白/灰 + 绿色强调色配色
- **v2.0**（2026 年 1 月）：Claude 风格的 12 射线星爆设计，绿宝石配色
- **v1.0**（2026 年 1 月）：原始六边形递进图标设计

---

**最后更新**：2026 年 2 月
**当前版本**：3.0（指南针括号）
**所有资产**：生产就绪 SVG、完全可缩放、WCAG AA 无障碍
