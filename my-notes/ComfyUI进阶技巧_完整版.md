# 🚀 ComfyUI 进阶技巧：ControlNet 与 IPAdapter

> 让 AI 图像生成更精准控制的进阶教程

## 📂 目录

1. [ControlNet 是什么](#controlnet-是什么)
2. [ControlNet 类型详解](#controlnet-类型详解)
3. [ControlNet 工作流](#controlnet-工作流)
4. [IPAdapter 是什么](#ipadapter-是什么)
5. [IPAdapter 模型选择](#ipadapter-模型选择)
6. [IPAdapter 工作流](#ipadapter-工作流)
7. [实战案例](#实战案例)

---

## 🎯 ControlNet 是什么？

### 定义

ControlNet 是一种神经网络结构，可以将**额外的条件输入**添加到预训练的扩散模型中，从而控制图像生成过程。

### 核心作用

```
文字描述 ─────┐
              ├──→ 扩散模型 → 生成图像
              │
ControlNet ───┘  ← 添加精确控制（姿势、轮廓、深度等）
```

### 你已安装的插件

你的叶秋版 ComfyUI 已安装：`comfyui_controlnet_aux`

---

## 📋 ControlNet 类型详解

### 1. Canny (边缘检测)

**用途**: 提取图像边缘线条，精确控制构图

**预处理**: `CannyEdgePreprocessor`

**参数**:
- Low Threshold: 100-200（低阈值）
- High Threshold: 200-400（高阈值）

**使用场景**: 建筑、产品设计、线条稿上色

```
原始图 → Canny → 控制线稿 → 生成新图
```

### 2. Depth (深度图)

**用途**: 保持图像的空间深度结构

**预处理**: `DepthMapPreprocessor`

**参数**:
- 分辨率: 512-1024

**使用场景**: 室内设计、场景重建、保持物体前后关系

### 3. Pose (姿态检测)

**用途**: 控制人物姿势

**预处理**: `DWPose` 或 `OpenPose`

**参数**:
- detect_hand: 是否检测手部
- detect_face: 是否检测面部

**使用场景**: 角色设计、动画制作、保持角色姿势

### 4. Scribble (涂鸦)

**用途**: 根据简单涂鸦生成图像

**预处理**: `ScribblePreprocessor`

**使用场景**: 快速草图转精细画作

### 5. Softedge (软边缘)

**用途**: 柔和的边缘控制，生成更自然的图像

**预处理**: `SoftEdgePreprocessor`

**优势**: 比 Canny 更自由，生成结果更丰富

### 6. Tile (分块)

**用途**: 增加细节、修复局部

**预处理**: `TilePreprocessor`

**使用场景**: 高清修复、细节增强

---

## 🔧 ControlNet 工作流

### 基础连接

```
                    ┌──────────────────┐
                    │  Load Image      │
                    │  (控制图)        │
                    └────────┬─────────┘
                             │
                             ▼
┌──────────────┐    ┌────────┴─────────┐    ┌─────────────────┐
│ LoadCheckpoint│───▶│  CLIPTextEncode │    │ ControlNet      │
│ (大模型)      │    │  (正向提示词)   │    │ Preprocessor    │
└──────────────┘    └─────────────────┘    │ (AUX集成)       │
                             │             └────────┬────────┘
                             │                      │
                             ▼                      ▼
                    ┌────────┴──────────────────────┐
                    │       KSampler                  │
                    │  (CFG 7-9, Steps 28-35)        │
                    └───────────────┬────────────────┘
                                    │
                                    ▼
                            ┌──────────────┐
                            │  VAEDecode   │
                            └──────────────┘
```

### 完整节点连接

```
LoadCheckpoint (大模型)
    │
    ├──────────────────────────┐
    │                          │
    ▼                          ▼
CLIPTextEncode (正)      ControlNetLoader
    │                          │
    ▼                          ▼
CLIPTextEncode (负)      AUXPreprocessor
    │                          │
    │                          ▼
    │                   ControlNetApply
    │                          │
    └──────────┬───────────────┘
               │
               ▼
          KSampler
               │
               ▼
          VAEDecode → SaveImage
```

### ControlNet 参数

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| Start Percent | 开始影响时间 | 0-0.3 |
| End Percent | 结束影响时间 | 0.7-1.0 |
| Strength | 控制强度 | 0.5-1.0 |

**强度建议**:
- 0.3-0.5: 轻微影响，保持自由度
- 0.5-0.8: 中等影响，平衡控制
- 0.8-1.0: 强控制，严格遵循

---

## 🎨 IPAdapter 是什么？

### 定义

IPAdapter 是**图像提示器**，可以将参考图的风格、构图、色调等特征迁移到生成图像中。

### 与 img2img 的区别

| 方面 | IPAdapter | img2img |
|------|-----------|---------|
| **原理** | 将图像特征融入提示词 | 直接在原图上修改 |
| **自由度** | 高，可自由发挥 | 低，保留原图痕迹 |
| **风格迁移** | 强 | 一般 |

**形象比喻**:
- IPAdapter = 记住风格后**重新创作**
- img2img = 拿着原图**慢慢擦除重画**

### 你已安装的插件

你的叶秋版 ComfyUI 已安装：`ComfyUI_IPAdapter_plus`

---

## 📦 IPAdapter 模型选择

### 模型存放位置

```
ComfyUI/models/
├── clip_vision/          # 必须安装
│   ├── CLIP-ViT-H-14-laion2B-s32B-b79K.safetensors
│   └── CLIP-ViT-bigG-14-laion2B-39B-b160k.safetensors
└── ipadapter/            # 需要创建
```

### 模型对比

| 模型 | 用途 | 强度 |
|------|------|------|
| `ip-adapter_sd15.safetensors` | 基础模型 | 平均 |
| `ip-adapter_sd15_light_v11.bin` | 轻量版 | 较弱 |
| `ip-adapter-plus_sd15.safetensors` | Plus 版 | 强 |
| `ip-adapter-plus-face_sd15.safetensors` | 脸部专用 | 强 |
| `ip-adapter-full-face_sd15.safetensors` | 完整脸部 | 最强 |
| `ip-adapter_sdxl_vit-h.safetensors` | SDXL 用 | 强 |
| `ip-adapter-plus_sdxl_vit-h.safetensors` | SDXL Plus | 最强 |

### 模型选择建议

| 场景 | 推荐模型 |
|------|----------|
| 风格迁移 | ip-adapter-plus |
| 人像参考 | ip-adapter-plus-face |
| 产品图 | ip-adapter (基础) |
| 动漫风格 | ip-adapter (基础) + 对应 checkpoint |

---

## 🔧 IPAdapter 工作流

### 基础连接

```
                    ┌──────────────────┐
                    │  Load Image     │
                    │  (参考图)       │
                    └────────┬─────────┘
                             │
                             ▼
┌──────────────┐    ┌────────┴─────────┐    ┌─────────────────┐
│ LoadCheckpoint│───▶│  CLIPTextEncode │    │ IPAdapter       │
│ (大模型)      │    │  (正向提示词)   │    │ Apply           │
└──────────────┘    └─────────────────┘    │ (IPAdapter Plus)│
                             │             └────────┬────────┘
                             │                      │
                             ▼                      ▼
                    ┌────────┴──────────────────────┐
                    │       KSampler                  │
                    └───────────────────────────────┘
```

### IPAdapter 参数

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| Weight | 影响强度 | 0.5-1.0 |
| Noise | 噪声比例 | 0.0-1.0 |
| Start At | 开始影响 | 0.0-1.0 |
| End At | 结束影响 | 0.0-1.0 |

**Noise 解读**:
- 0.0 = 完全保持参考图风格
- 0.5 = 平衡
- 1.0 = 几乎不受参考影响

---

## 🎯 实战案例

### 案例 1: 姿势控制 (ControlNet Pose)

**目标**: 让生成的女孩保持特定姿势

**工作流**:
```
1.**案例 1: 姿势控制 (ControlNet Pose)**

**目标**: 让生成的女孩保持特定姿势

**步骤**:
1. 加载一张姿势参考图（或者用空白背景的人体图）
2. 使用 DWPose Preprocessor 检测姿势
3. 连接 ControlNet Apply 到 KSampler
4. 设置 Strength: 0.8

**提示词**:
```
1girl, solo, elegant pose, beautiful dress, indoor, soft lighting, best quality
```

**负面提示词**:
```
bad hands, bad anatomy, missing fingers, extra limbs, worst quality
```

---

### 案例 2: 边缘线稿上色 (Canny)

**目标**: 给线稿精确上色

**工作流**:
1. 准备一张线稿图
2. 用 Canny Preprocessor 提取边缘
3. 文生图时应用 ControlNet

**提示词**:
```
1girl, anime style, colorful clothing, detailed background, masterpiece
```

**参数建议**:
- Preprocessor Low/High: 100/200
- ControlNet Strength: 0.7
- Steps: 28-35

---

### 案例 3: 风格迁移 (IPAdapter)

**目标**: 用参考图的风格生成新图像

**工作流**:
1. 加载一张风格参考图（如油画）
2. IPAdapter Apply 连接模型
3. 使用目标提示词生成

**提示词**:
```
1girl, portrait, elegant, indoor, soft lighting, masterpiece
```

**参数建议**:
- Weight: 0.7
- Noise: 0.3
- 选择 Plus 版模型效果更好

---

### 案例 4: 人像风格参考 (IPAdapter Face)

**目标**: 生成类似参考图面部特征的肖像

**工作流**:
1. 加载一张人像照片
2. 使用 ip-adapter-plus-face 模型
3. 生成新人像

**提示词**:
```
1girl, beautiful Asian woman, portrait photography, natural lighting, best quality
```

**参数建议**:
- Weight: 0.6-0.8
- Noise: 0.2-0.4
- 模型: ip-adapter-plus-face_sd15.safetensors

---

## 🎨 高级技巧

### 1. ControlNet + IPAdapter 组合

```
参考图1 (姿势) ──▶ ControlNet
                          │
                          ▼
参考图2 (风格) ──▶ IPAdapter ──▶ KSampler ──▶ 生成图
                          ▲
                          │
                    Checkpoint
```

**使用场景**: 既要保持姿势，又要参考风格的复杂需求

### 2. 多个 ControlNet

可以同时使用多个 ControlNet：

```
参考图1 (边缘) ──▶ ControlNet1 (Weight 0.5)
参考图2 (深度) ──▶ ControlNet2 (Weight 0.3)
                              │
                              ▼
                     KSampler (综合控制)
```

### 3. 分阶段控制

使用 Start/End Percent 分阶段影响：

| 阶段 | Start | End | 用途 |
|------|-------|-----|------|
| 构图期 | 0.0 | 0.3 | 建立结构 |
| 细化期 | 0.3 | 0.7 | 添加细节 |
| 收尾期 | 0.7 | 1.0 | 微调完成 |

---

## 📊 参数调试指南

### ControlNet 调试

| 问题 | 解决方案 |
|------|----------|
| 生成图和参考差异大 | 提高 Strength |
| 生成图太死板 | 降低 Strength 或调整 Start Percent |
| 边缘不清晰 | 调整 Canny 阈值 |
| 姿势不对 | 使用 DWPose 检测 |

### IPAdapter 调试

| 问题 | 解决方案 |
|------|----------|
| 风格不明显 | 提高 Weight 或选择 Plus 模型 |
| 图像太像原图 | 提高 Noise |
| 图像太自由发挥 | 降低 Noise |
| 脸部不像 | 使用 Face 版模型，提高 Weight |

---

## 🔗 模型下载

### ControlNet 模型

| 模型 | 链接 |
|------|------|
| ControlNet 1.1 | [Hugging Face](https://huggingface.co/lllyasvely/ControlNet-v1-1) |
| ControlNet 1.0 | [Hugging Face](https://huggingface.co/lllyasvely/ControlNet-v1-1/tree/main) |

### IPAdapter 模型

| 模型 | 链接 |
|------|------|
| 官方 GitHub | [ComfyUI_IPAdapter_plus](https://github.com/cubiq/ComfyUI_IPAdapter_plus) |

### 模型安装路径

```
ComfyUI/
├── models/
│   ├── controlnet/           # ControlNet 模型
│   ├── clip_vision/          # CLIP Vision (必须)
│   ├── ipadapter/            # IPAdapter 模型 (需手动创建)
│   └── ipadapter/
│       ├── ip-adapter_sd15.safetensors
│       ├── ip-adapter-plus_sd15.safetensors
│       └── ip-adapter-plus-face_sd15.safetensors
```

---

## 🔄 更新日志

- **2026-04-21**: 创建 ControlNet 和 IPAdapter 进阶教程
- 详细讲解 6 种 ControlNet 类型
- 分析 IPAdapter 模型选择
- 提供 4 个实战案例
- 分享高级组合技巧
