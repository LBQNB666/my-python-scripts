# 🤖 AI 项目收藏

> 收藏和研究的 AI 相关项目和资源

## 📂 目录

### 图像生成
- Stable Diffusion 模型
- Flux 工作流
- ComfyUI 整合包

### AI 工具
- QwenPaw - 本地 AI 助手
- MiniMax M2.7 - 全模态模型
- Cursor - AI 编程工具

### 自动化
- Python 脚本
- 工作流自动化

---

## 🎨 图像生成项目

### ComfyUI 相关

| 项目 | 说明 | 链接 |
|------|------|------|
| ComfyUI-aki-v3 | 我正在用的整合包 | 本地 |
| ComfyUI-Manager | 插件管理器 | [GitHub](https://github.com/mattmusk/ComfyUI-Manager) |
| ComfyUI-GGUF | GGUF 模型支持 | [GitHub](https://github.com/comfyanonymous/ComfyUI) |
| ComfyUI-Impact-Pack | 增强节点包 | [GitHub](https://github.com/Dr.Lt.Data/ComfyUI-Impact-Pack) |

### 热门 Flux 模型

| 模型 | 说明 | VRAM |
|------|------|------|
| flux1-dev-fp8 | 开发版 FP8 量化 | ~16GB |
| flux- realism | 写实风格 | ~16GB |
| flux-schnell | 快速生成 | ~16GB |

### 热门 LoRA

| LoRA | 风格 | 大小 |
|------|------|------|
| majicFlus | 写实人像 | ~146MB |
| YFilter | 人像增强 | ~288MB |
| 极致人物修复 | 皮肤细节 | ~475MB |

---

## 🛠️ AI 工具配置

### MiniMax M2.7 配置

```bash
# 环境变量配置
export ANTHROPIC_BASE_URL=https://api.minimaxi.com/anthropic
export ANTHROPIC_API_KEY=你的Token Plan Key
```

### QwenPaw 配置

```json
{
  "provider_id": "minimax-cn",
  "model": "MiniMax-M2.7",
  "api_key": "你的API Key"
}
```

### MCP 配置

#### MiniMax MCP (搜索和图片理解)
```json
{
  "name": "MiniMax",
  "command": "uvx",
  "args": ["minimax-coding-plan-mcp", "-y"],
  "env": {
    "MINIMAX_API_KEY": "你的API Key",
    "MINIMAX_API_HOST": "https://api.minimaxi.com"
  }
}
```

#### GitHub MCP (代码管理)
```json
{
  "name": "GitHub",
  "command": "uvx",
  "args": ["github-mcp-server", "-y"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "你的Token"
  }
}
```

---

## 📊 我的 AI 工具链

```
日常使用:
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  MiniMax    │────▶│   QwenPaw    │────▶│  ComfyUI    │
│   M2.7      │     │   (AI助手)   │     │  (生图)     │
└─────────────┘     └──────────────┘     └─────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  GitHub      │
                    │  (代码仓库)  │
                    └──────────────┘
```

---

## 🔗 优质 AI 资源

### 模型下载
- [Civitai](https://civitai.com/) - SD 模型和 LoRA
- [Hugging Face](https://huggingface.co