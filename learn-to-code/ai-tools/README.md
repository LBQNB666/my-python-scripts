# 🤖 AI 编程工具学习指南

> 2026年最火的 AI 编程助手对比与使用教程

## 📊 工具对比

| 工具 | 开发公司 | 特点 | 价格 |
|------|----------|------|------|
| **Cursor** | Anthropic 投资 | 最好用的 AI IDE | 免费/付费 |
| **Claude Code** | Anthropic | 命令行编程工具 | Claude API |
| **GitHub Copilot** | Microsoft | 代码补全老牌 | $10/月 |
| **Trae** | 字节跳动 | 免费中文友好 | 免费 |
| **VS Code + AI** | Microsoft | 轻量级方案 | 免费 |

## 🚀 Cursor 详细教程

### 为什么推荐 Cursor？

1. ✅ 基于 VS Code，上手快
2. ✅ 内置多款 AI 模型（GPT-4、Claude 等）
3. ✅ 支持自定义模型（如 MiniMax M2.7）
4. ✅ 强大的代码补全
5. ✅ 多光标编辑

### 安装 Cursor

1. 访问 https://cursor.com
2. 下载 Windows 版本
3. 安装并打开

### 配置 MiniMax M2.7

1. 按 `Ctrl + ,` 打开设置
2. 选择 **Models** 或 **API Keys**
3. 配置：
   ```
   Provider: Anthropic / Custom
   Base URL: https://api.minimaxi.com/anthropic
   API Key: 你的 MiniMax Token Plan Key
   Model: MiniMax-M2.7
   ```

### 常用快捷键

| 功能 | 快捷键 |
|------|--------|
| AI 聊天 | `Ctrl + L` |
| 代码补全 | `Tab` |
| 接受补全 | `Tab` |
| 拒绝补全 | `Esc` |
| 多行编辑 | `Ctrl + D` |
| 全局搜索 | `Ctrl + Shift + F` |

### Cursor 的核心功能

1. **Composer** - 同时处理多个文件
2. **Agent Mode** - AI 自动完成复杂任务
3. **Codebase Index** - 理解整个项目
4. **Inline Chat** - 代码内即时提问

---

## 💻 Claude Code 教程

### 什么是 Claude Code？

Claude Code 是 Anthropic 官方推出的命令行编程工具，可以在终端中直接与 Claude 对话和编程。

### 安装 Claude Code

```bash
# macOS / Linux
npm install -g @anthropic-ai/cl