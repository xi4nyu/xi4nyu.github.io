# AGENTS.md

## 项目概述

这是一个基于 Hexo 的个人技术博客站点，主要记录编程、AI、工具使用等技术内容。

- **作者**: cc
- **语言**: 中文 (zh-CN)
- **博客地址**: http://xi4nyu.github.io
- **主题**: meilidu
- **文章数量**: 约 30+ 篇

## 项目结构

```
xi4nyu.github.io/
├── _config.yml           # Hexo 主配置文件
├── package.json          # 项目依赖
├── source/               # 源文件目录
│   └── _posts/          # 博客文章（Markdown 格式）
├── themes/              # 主题目录
│   └── meilidu/         # 当前使用的主题
└── scaffolds/           # 文章模板
```

## 内容主题

博客主要涵盖以下技术领域：

- **AI/机器学习**: LLM、Claude、GPT 等 AI 工具和技术
- **编程语言**: Python、Go 等
- **开发工具**: Claude Code、MCP、py-spy 等
- **其他技术**: 各类开发实践和工具使用经验

## 工作流程

### 创建新文章

```bash
# 使用 Hexo 创建新文章
hexo new post "文章标题"

# 或直接在 source/_posts/ 创建 Markdown 文件
# 文件名格式: YYYY-MM-DD-title.md
```

### 文章格式

每篇文章使用 YAML front matter：

```markdown
---
title: 文章标题
tags: 标签名
categories: 分类名
date: YYYY-MM-DD HH:mm:ss
---

文章内容...
```

### 本地预览

```bash
# 启动本地服务器
npm run server
# 或
hexo server

# 访问 http://localhost:4000
```

### 构建和部署

```bash
# 清理缓存
npm run clean

# 生成静态文件
npm run build

# 部署到 GitHub Pages
npm run deploy
```

部署配置已设置为推送到 `gh-pages` 分支。

## 技术栈

- **静态站点生成器**: Hexo 7.3.0
- **主题**: meilidu
- **插件**:
  - hexo-filter-mathjax: 数学公式支持
  - hexo-filter-mermaid-diagrams: 流程图支持
  - hexo-renderer-marked: Markdown 渲染
  - prismjs: 代码高亮

## 开发注意事项

1. **文章命名**: 使用日期前缀 (YYYY-MM-DD-title.md) 便于排序和管理
2. **分类体系**: 主要分类包括 AI、编程语言、工具等
3. **代码高亮**: 使用 highlight.js 和 Prism.js 双重支持
4. **数学公式**: 通过 MathJax 支持 LaTeX 公式
5. **时区**: 设置为 Asia/Shanghai

## 常见任务

### 添加新博客文章

在 `source/_posts/` 目录下创建新的 Markdown 文件，按照现有文章格式编写内容。

### 修改主题配置

主题配置文件位于 `themes/meilidu/` 目录下。

### 更新依赖

```bash
npm update
```

### 调试构建问题

```bash
# 清理并重新生成
hexo clean && hexo generate
```

## 写作规范

本博客的文章遵循阮一峰技术博客写作规范，具体规则见 `docs/ruanyifeng-writing/` 目录：

- **主规范文档**: `docs/ruanyifeng-writing/阮一峰技术博客写作规范.md`
- **语言修辞分析**: `docs/ruanyifeng-writing/ruanyifeng-language-rhetoric-report.md`

### 核心原则

**一句话总纲**：一篇文章只讲一件事，用最短的路径把它讲清楚。

三条不可动摇的原则：

| 原则 | 含义 | 违反的后果 |
|---|---|---|
| 单线 | 一篇一个论点，一条线走到底 | 读者迷路，不知道你在讲哪件事 |
| 具体 | 抽象概念必须落到可感、可算、可复现的东西上 | 读者看懂了每个字，却没懂这件事 |
| 先想清楚 | 想不清楚就不要动笔，先拆问题 | 写成思维导图，不是文章 |

### 文章评审流程

**所有新文章在发布前必须经过写作规范评审**：

1. **创建文章草稿** - 在 `source/_posts/` 创建 Markdown 文件
2. **内容评审** - 使用 AI 工具对照规范进行评审
3. **改进优化** - 根据评审建议修改文章
4. **最终检查** - 使用规范中的"发布前检查清单"自查
5. **发布** - 通过评审后方可发布

### 评审要点

文章评审应重点关注以下方面：

- **选题与标题**：标题是否等于内容，是否 5-15 字
- **开篇**：是否在 1-3 句内完成点题、说价值、交代动因
- **结构**：是否单线结构，是否使用编号推进
- **段落与句子**：段落是否 1-3 句，句长是否合理
- **语言**：人称使用、连接词、术语处理是否规范
- **类比**：抽象概念是否有具体类比，映射是否显式
- **举例**：例子是否可复现，数字是否精确
- **代码与图**：是否有引导句和解释句，图是否紧跟段落
- **引用**：是否有参考链接，出处是否明确
- **结尾**：是否显式收束，是否与开头闭环

### 常见问题

- ❌ 一篇文章讲两件事
- ❌ 标题与内容不符
- ❌ 抽象概念没有落地
- ❌ 裸贴代码，没有引导句
- ❌ 营销腔、排比抒情、空洞形容词
- ❌ 长难句、超长段落
- ❌ 术语不解释

## AI 辅助建议

当使用 AI 工具（如 Claude Code）协助此项目时：

### 内容创作

- **写作辅助**: 协助撰写符合规范的技术文章
- **规范评审**: 对照 `docs/ruanyifeng-writing/` 规范评审文章
- **改进建议**: 根据评审结果给出具体的改进建议
- **代码示例**: 生成文章中的代码示例和说明

### 技术支持

- **格式检查**: 检查 Markdown 格式和 YAML front matter
- **SEO 优化**: 优化文章标题、标签和描述
- **主题定制**: 辅助修改主题样式和布局

### 评审命令示例

使用 AI 工具评审文章时，可以这样提问：

```
请根据 docs/ruanyifeng-writing/阮一峰技术博客写作规范.md 
对文章 source/_posts/YYYY-MM-DD-title.md 进行评审，
给出改进建议。
```

## 最近更新

- 2025-10-11: 发布 GLM-4-6 相关文章
- 2025-08-25: 发布 Claude Code 使用教程
- 2025-03-23: 发布 MCP 相关内容
- 2025-03: 发布多篇 AI 相关文章系列
