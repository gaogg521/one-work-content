---
name: image-cog
description: 由 CellCog 提供支持的 AI 图像生成。创建图像、编辑照片、一致的角色、产品摄影、基于参考的图像、图像集、风格迁移。使用 AI 进行专业图像创建。
metadata: None
openclaw: None
emoji: 🎨
author: CellCog
dependencies:
- cellcog
tags:
- AI
---

# Image Cog - 由 CellCog 提供支持的 AI 图像生成

使用 AI 创建专业图像 - 从单张图像到一致的角色集再到产品摄影。

---

## 先决条件

此 skill 需要 `cellcog` skill 进行 SDK 设置和 API 调用。

```bash
clawhub install cellcog
```

**首先阅读 cellcog skill**以进行 SDK 设置。此 skill 向您展示可能的功能。

**快速模式 (v1.0+)：**
```python
# Fire-and-forget - 立即返回
result = client.create_chat(
    prompt="[your image request]",
    notify_session_key="agent:main:main",
    task_label="image-task",
    chat_mode="agent"  # 简单图像使用 "agent"，复杂图像使用 "agent team"
)
# Daemon 在完成时通知您 - 不要轮询
```

---

## 您可以创建什么图像

### 单张图像创建

从文本描述生成任何图像：

- **场景**: "A cozy coffee shop interior with morning light streaming through windows"
- **肖像**: "Professional headshot of a confident woman in business attire"
- **产品**: "Minimalist product shot of a white sneaker on a marble surface"
- **抽象**: "Geometric abstract art in navy and gold"
- **自然**: "Misty mountain landscape at sunrise with a lone hiker"

### 图像编辑

转换现有图像：

- **风格迁移**: "Transform this photo into a watercolor painting"
- **背景移除**: "Remove the background and place on a clean white backdrop"
- **增强**: "Enhance the colors and add dramatic lighting"
- **修改**: "Change the person's outfit to a red dress"

### 一致的角色

在不同场景中创建同一角色的多张图像：

- **角色系列**: "Create a tech entrepreneur character, then show them: 1) At their desk coding, 2) Presenting to investors, 3) Celebrating a product launch"
- **吉祥物变体**: "Design a friendly robot mascot, then create versions for: welcome page, error page, success message, loading screen"
- **故事序列**: "Create a main character, then illustrate them in 5 scenes of a journey"

这对于以下方面非常强大：
- 漫画条和故事板
- 具有一致角色的营销活动
- 视频帧生成
- 跨语境的品牌吉祥物

### 产品摄影风格

专业产品视觉效果：

- **Hero Shots**: "Product hero shot of a smartwatch on a gradient background"
- **生活方式照片**: "Smartphone being used by a person in a modern living room"
- **平铺照片**: "Flat lay of skincare products with botanical elements"
- **360 度视图**: "Multiple angles of a leather handbag - front, side, back, detail"

### 相关图像集

用于活动或系列的多个连贯图像：

- **社交媒体集**: "5 Instagram post images for a fitness brand - consistent style, varied content"
- **网站 Hero 图**: "3 hero images for a SaaS landing page - professional, modern, tech-focused"
- **广告变体**: "4 versions of a product ad with different backgrounds and moods"
- **博客插图**: "Set of 6 illustrations for a blog post about productivity tips"

### 基于参考的生成

使用现有图像作为风格、角色或构图的参考：

- **风格匹配**: "Create a new image in the same artistic style as this reference"
- **角色一致性**: "Using this person as reference, create a new scene with them hiking"
- **品牌对齐**: "Create product images matching this brand's visual style"
- **构图参考**: "Create a similar composition but with different subjects"

---

## 图像规格

| 方面 | 选项 |
|--------|---------|
| **宽高比** | 1:1 (square), 16:9, 9:16, 4:3, 3:4, 3:2, 2:3, 21:9 |
| **尺寸** | 1K (~1024px), 2K (~2048px), 4K (~4096px) |
| **风格** | Photorealistic, illustration, watercolor, oil painting, anime, digital art, vector |
| **格式** | PNG (default) |

**尺寸建议：**
- **1K**: 快速迭代、缩略图、社交媒体帖子、草稿
- **2K**: 标准网页内容、演示文稿、营销材料
- **4K**: Hero 图像、印刷材料、细节重要的最终交付物

---

## 何时使用 Agent Team 模式

对于图像生成，建议在以下情况下使用 `chat_mode="agent team"`：
- 需要多个元素的复杂场景
- 一致的角色系列
- 需要分析的基于参考的生成
- 相关图像集

对于简单的单张图像，`chat_mode="agent"` 可以更快地工作。

---

## 示例图像提示

**专业头像：**
> "Create a professional headshot of a friendly Asian woman in her 30s, wearing a navy blazer, soft studio lighting, neutral gray background, confident but approachable expression. 1:1 square, 2K quality, photorealistic."

**产品摄影：**
> "Product shot of a premium wireless earbuds case, matte black finish, on a reflective dark surface with subtle blue accent lighting. Minimalist, high-end tech aesthetic. 4:3 landscape, 4K for hero image."

**一致角色集：**
> "Create a character: young Black male software developer, casual style with glasses, friendly demeanor. Then create 4 images:
> 1. Working at a standing desk with multiple monitors
> 2. In a video call meeting, explaining something
> 3. At a coffee shop with laptop, thinking
> 4. Celebrating with team, high-fiving
> Keep the character exactly consistent across all images."

**社交媒体集：**
> "Create 5 Instagram posts for a plant-based meal delivery service:
> 1. Colorful Buddha bowl from above
> 2. Happy person unpacking delivery
> 3. Meal prep containers arranged neatly
> 4. Close-up of fresh ingredients
> 5. Before/after showing ingredients to finished dish
> Style: bright, fresh, appetizing, consistent warm color grading. 1:1 square format."

**风格迁移：**
> "Transform this uploaded photo of a city street into a Studio Ghibli anime style illustration. Keep the composition and elements but apply the characteristic Ghibli warmth, soft clouds, and whimsical details."

---

## 获取更好图像的提示

1. **描述性要强**: "Woman in office" 太模糊。"Confident woman in her 40s, silver blazer, modern glass-walled office, warm afternoon light" 更好。

2. **指定风格**: "Photorealistic", "digital illustration", "watercolor", "minimalist vector".

3. **描述光线**: "Soft natural light", "dramatic side lighting", "golden hour glow", "studio lighting".

4. **包含氛围**: "Professional and confident", "warm and inviting", "energetic and vibrant".

5. **提及构图**: "Rule of thirds", "centered symmetry", "close-up", "wide establishing shot".

6. **为了一致性**: 创建角色系列时，首先详细描述角色，然后在后续提示中引用 "the same character"。
