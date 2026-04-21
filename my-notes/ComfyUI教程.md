# 🎨 ComfyUI 完全指南

> 我学习 ComfyUI 的笔记和心得

## 📖 什么是 ComfyUI？

ComfyUI 是一个功能强大、高度模块化的 Stable Diffusion 图形用户界面和后端系统。

### 与 WebUI 的区别

| 特性 | ComfyUI | WebUI (AUTOMATIC1111) |
|------|---------|----------------------|
| 工作流 | 可视化节点流程 | 一键生成 |
| 灵活性 | 极高 | 一般 |
| 学习曲线 | 较陡 | 较平缓 |
| 内存占用 | 较低 | 较高 |
| 速度 | 较快 | 一般 |

## 🛠️ 安装与配置

### 系统要求
- **操作系统**: Windows 10/11, macOS, Linux
- **显卡**: NVIDIA GPU (6GB+ VRAM)
- **内存**: 16GB+
- **硬盘**: 至少 30GB 可用空间

### 安装步骤

1. **下载 ComfyUI**
   - 官方地址: https://github.com/comfyanonymous/ComfyUI
   - 或使用整合包（如 ComfyUI-aki-v3）

2. **安装模型**
   ```
   模型放置目录:
   models/checkpoints/    # 大模型 (如 Flux, SDXL)
   models/vae/           # VAE 模型
   models/loras/          # LoRA 模型
   models/clip/           # CLIP 模型
   models/controlnet/     # ControlNet 模型
   ```

3. **安装插件 (custom nodes)**
   - 安装 ComfyUI-Manager 管理器
   - 通过管理器安装其他插件

### 推荐插件

| 插件名 | 功能 |
|--------|------|
| ComfyUI-Manager | 管理器，简化安装 |
| ComfyUI-Impact-Pack | 增强节点 |
| ComfyUI-GGUF | 支持 GGUF 格式模型 |
| ComfyUI-nunchaku | 优化性能 |
| ComfyUI-KJNodes | 实用工具节点 |

## 📝 基础工作流

### 1. 简单文生图

```
Load Checkpoint → CLIP Text Encode (正向) → CLIP Text Encode (负向) 
       ↓
KSampler → VAE Decode → Save Image
```

### 2. 图生图

```
Load Checkpoint → Load Image → CLIP Text Encode → KSampler → Save Image
                                    ↑
                         VAE Encode (from Load Image)
```

### 3. ControlNet 控制

```
Load Checkpoint → Load Image → ControlNet Apply → CLIP Text Encode 
       ↓
KSampler → VAE Decode → Save Image
```

## 🎯 进阶技巧

### 1. 工作流优化

- **使用 GGUF 量化模型** - 减少显存占用
- **调整批次数量** - 一次生成多张
- **使用高分辨率修复** - 提升图片质量

### 2. Flux 模型使用

Flux 是目前最强的开源图像生成模型之一。

```javascript
// Flux 工作流关键节点
LoadFluxModel        // 加载 Flux 模型
CLIPTextEncodeFlux   // 文本编码 (Flux 专用)
BasicScheduler       // 调度器
FluxGuidance         // 引导参数
KSamplerAdvanced     // 采样器
VAEDecode            // 解码
SaveImage            // 保存
```

### 3. LoRA 使用技巧

- **权重调整**: 0.1-0.8 之间效果较好
- **叠加使用**: 可以叠加多个 LoRA
- **选择性应用**: 只对特定层应用

## 🔧 常见问题

### Q: 显存不足怎么办？
A: 
1. 使用 GGUF 量化版本模型
2. 减少批次大小 (batch size)
3. 使用 CPU offload
4. 降低图片分辨率

### Q: 生成速度慢？
A:
1. 升级显卡驱动
2. 使用较短的 step 数 (20-30)
3. 开启图片加速

### Q: 模型放在哪？
A:
```
ComfyUI/
└── models/
    ├── checkpoints/     # 大模型
    ├── vae/            # VAE
    ├── loras/          # LoRA
    ├── controlnet/     # ControlNet
    └── embeddings/     # 嵌入
```

## 📚 学习资源

### 官方资源
- [ComfyUI 官方文档](https://docs.comfyui.com/)
- [ComfyUI GitHub](https://github.com/comfyanonymous/ComfyUI)

### 工作流网站
- [Civitai](https://civitai.com/) - 模型和工作流分享
- [OpenArt](https://openart.ai/) - 工作流教程
- [ComfyUI Workflows](https://github.com/CY-CHENYUE/ComfyUI_Workflows) - 工作流集合

### 社区
- Reddit: r/comfyui
- Discord: ComfyUI Official

## 🔄 更新日志

- **2026-04-21**: 创建 ComfyUI 学习笔记
- 添加基础教程和进阶技巧
- 整理常见问题解决方案
