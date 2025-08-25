---
title: 使用 Claude Code 进行编码
tags: ClaudeCode
categories: AI
date: 2025-08-25 10:44:00
---


### 介绍

什么是[Claude Code](https://www.anthropic.com/claude-code)，Claude Code 是一个命令行的编程肋手，你可以通过自然语言和它进行对话，让它帮助你完成代码编写。而且借肋MCP，它可以完成更多其它的功能，不只是编程。它已经开源：https://github.com/anthropics/claude-code。

### 安装
由于Claude Code 使用TypeScript编写，所以使用npm安装，首先得安装nodejs.

```bash
npm install -g @anthropic-ai/claude-code
```

### 首次配置
我们可以使用Anthropics官方的模型，也可以指定其它的模型，比如：QWen，Kimi，GLM等。

```bash
# 设置Anthorpics Key
export ANTHROPIC_API_KEY="your-api-key-here"

# 设置QWen
export ANTHROPIC_BASE_URL=https://dashscope.aliyuncs.com/api/v2/apps/claude-code-proxy
export ANTHROPIC_AUTH_TOKEN="your-api-key-here"

# 设置Kimi
export ANTHROPIC_BASE_URL=https://api.moonshot.cn/anthropic
export ANTHROPIC_AUTH_TOKEN="your-api-key-here"

```

也可以使用claude-code-router，它是一个第三方的路由工具，用于为 Claude Code 灵活地切换不同的后端 API。dashScope平台提供了一个简单的扩展包 claude-code-config，可为 claude-code-router 生成包含 dashScope 支持的默认配置。

```bash
npm install -g @musistudio/claude-code-router
npm install -g @dashscope-js/claude-code-config

# 生成配置
ccr-dashscope

# 使用UI进行设置
ccr ui

# 使用 Claude Codoe
ccr code
```
![ccr](/images/ai/claudecode.png)


### 使用Claude Code 进行编码

生成一个坦克大战的游戏

```bash
mkdir tank && cd tank
ccr code
```

![tank](/images/ai/tank.png)