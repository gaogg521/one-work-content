---
name: image2prompt
description: 分析图像并为图像生成生成详细的提示。支持肖像、风景、产品、动物、插图类别，可输出结构化或自然语言格式。
homepage: https://docs.openclaw.ai/tools/image2prompt
user-invocable: True
metadata:
  openclaw:
    emoji: 🖼️
    primaryEnv: OPENAI_API_KEY
    requires:
      anyBins:
      - openclaw
tags:
- 图像生成
- 管理
---

# Image to Prompt

分析图像并为 AI 图像生成生成详细的、可复现质量的提示。

## 工作流

**步骤 1：类别检测**
首先，将图像分类到以下类别之一：
- `portrait` — 人物为主角（照片、艺术品、数字艺术）
- `landscape` — 自然风景、城市景观、建筑、户外环境
- `product` — 商业产品照片、商品
- `animal` — 动物为主角
- `illustration` — 图表、信息图、UI 模型、技术图纸
- `other` — 不符合上述类别的图像

**步骤 2：类别特定分析**
根据检测到的类别生成详细提示。

## 用法

### 基本分析

```bash
# 分析图像（自动检测类别）
openclaw message send --image /path/to/image.jpg "Analyze this image and generate a detailed prompt for reproduction"
```

### 指定输出格式

**自然语言（默认）：**
```
Analyze this image and write a detailed, flowing prompt description (600-1000 words for portraits, 400-600 for others).
```

**结构化 JSON：**
```
Analyze this image and output a structured JSON description with all visual elements categorized.
```

### 带维度提取

请求维度高亮以获取每个视觉方面的标记短语：
```
Analyze this image with dimension extraction. Tag phrases for: backgrounds, objects, characters, styles, actions, colors, moods, lighting, compositions, themes.
```

## 类别特定元素

### 肖像分析涵盖：
- **Model/Style**: 摄影类型、质量级别、视觉风格
- **Subject**: 性别、年龄、种族、肤色、体型
- **Facial Features**: 眼睛、嘴唇、脸型、表情
- **Hair**: 颜色、长度、风格、分缝
- **Pose**: 身体姿势、朝向、腿/手位置、视线
- **Clothing**: 类型、颜色、图案、合身度、材质、风格
- **Accessories**: 珠宝、包、帽子等
- **Environment**: 位置、地面、背景、氛围
- **Lighting**: 类型、一天中的时间、阴影、对比度、色温
- **Camera**: 角度、高度、镜头类型、镜头、景深、透视
- **Technical**: 真实感、后期处理、分辨率

### 风景分析涵盖：
- 地形和水体特征
- 天空和大气元素
- 前景/背景构图
- 自然光线和氛围
- 调色板和摄影风格

### 产品分析涵盖：
- 产品特征和材质
- 设计元素和形状
- 舞台布置和背景
- 摄影棚灯光设置
- 商业摄影风格

### 动物分析涵盖：
- 物种识别和斑纹
- 姿势和行为
- 表情和特征
- 栖息地和环境
- 野生动物/宠物摄影风格

### 插图分析涵盖：
- 图表类型（流程图、信息图、UI 等）
- 视觉元素（图标、形状、连接线）
- 布局和层次结构
- 设计风格（扁平、等距等）
- 配色方案和含义

## 输出示例

### 自然语言输出（肖像）
```json
{
  "prompt": "A stunning photorealistic portrait of a young woman in her mid-20s with fair porcelain skin and warm pink undertones. She has striking emerald green almond-shaped eyes with long dark lashes, full rose-colored lips curved in a subtle confident smile, and an oval face with high cheekbones..."
}
```

### 结构化输出（肖像）
```json
{
  "structured": {
    "model": "photorealistic",
    "quality": "ultra high",
    "style": "cinematic natural light photography",
    "subject": {
      "identity": "young beautiful woman",
      "gender": "female",
      "age": "mid 20s",
      "ethnicity": "European",
      "skin_tone": "fair porcelain with pink undertones",
      "body_type": "slim athletic",
      "facial_features": {
        "eyes": "emerald green, almond-shaped, intense gaze",
        "lips": "full, rose pink, subtle smile",
        "face_shape": "oval with high cheekbones",
        "expression": "confident and serene"
      },
      "hair": {
        "color": "warm honey blonde",
        "length": "long",
        "style": "soft waves",
        "part": "center"
      }
    },
    "pose": {
      "position": "standing",
      "body_orientation": "three-quarter turn to camera",
      "legs": "weight on right leg, relaxed stance",
      "hands": {
        "right_hand": "resting on hip",
        "left_hand": "hanging naturally at side"
      },
      "gaze": "direct eye contact with camera"
    },
    "clothing": {
      "type": "flowing maxi dress",
      "color": "dusty rose",
      "pattern": "solid",
      "details": "V-neckline, cinched waist, silk material",
      "style": "romantic feminine"
    },
    "accessories": ["delicate gold necklace", "small hoop earrings"],
    "environment": {
      "location": "outdoor garden",
      "ground": "cobblestone path",
      "background": "blooming roses, soft bokeh",
      "atmosphere": "dreamy and romantic"
    },
    "lighting": {
      "type": "natural sunlight",
      "time": "golden hour",
      "shadow_quality": "soft diffused shadows",
      "contrast": "medium",
      "color_temperature": "warm"
    },
    "camera": {
      "angle": "slightly below eye level",
      "camera_height": "chest height",
      "shot_type": "medium shot",
      "lens": "85mm",
      "depth_of_field": "shallow",
      "perspective": "slight compression, flattering"
    },
    "mood": "romantic, confident, ethereal",
    "realism": "highly photorealistic",
    "post_processing": "soft color grading, subtle glow",
    "resolution": "8k"
  }
}
```

### 带维度
```json
{
  "prompt": "...",
  "dimensions": {
    "backgrounds": ["outdoor garden", "blooming roses", "soft bokeh"],
    "objects": ["delicate gold necklace", "small hoop earrings"],
    "characters": ["young beautiful woman", "mid 20s", "European"],
    "styles": ["photorealistic", "cinematic natural light photography"],
    "actions": ["standing", "three-quarter turn", "direct eye contact"],
    "colors": ["dusty rose", "honey blonde", "emerald green"],
    "moods": ["romantic", "confident", "ethereal", "dreamy"],
    "lighting": ["golden hour", "natural sunlight", "soft diffused shadows"],
    "compositions": ["medium shot", "85mm", "shallow depth of field"],
    "themes": ["romantic feminine", "portrait photography"]
  }
}
```

## 获取最佳结果的提示

1. **高分辨率图像**产生更详细的提示
2. **清晰、光线充足的图像**产生更好的类别检测
3. **请求结构化输出**当您需要以编程方式访问单个元素时
4. **使用维度提取**当构建提示数据库或训练数据时
5. **如有需要，请指定字数期望**以用于自然语言输出

## 集成

此 skill 适用于任何具有视觉能力的模型。为获得最佳效果，请使用：
- GPT-4 Vision
- Claude 3 (Opus/Sonnet)
- Gemini Pro Vision
