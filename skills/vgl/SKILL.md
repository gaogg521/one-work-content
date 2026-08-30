---
name: vgl
description: 为 Bria 的 FIBO 图像生成模型编写结构化的 VGL (Visual Generation Language) JSON prompts。当为文本到图像生成、图像编辑、inpainting、outpainting、背景生成或 captioning 创建 JSON 格式的详细图像描述时使用此 skill。触发词包括编写结构化 prompts、创建 VGL JSON、为 AI 生成描述图像，或与 Bria/FIBO 的 structured_prompt 格式协作。也在将自然语言图像请求转换为 FIBO 模型所需的确定性 JSON schema 时使用。
tags:
- AI
- Schema
---

# Bria VGL Prompt 编写

使用 Visual Generation Language (VGL) 为 Bria 的 FIBO 模型生成结构化 JSON prompts。

> **相关 Skill**: 使用 **[bria-ai](../bria-ai/SKILL.md)** 通过 Bria API 执行这些 VGL prompts。VGL 定义结构化 prompt 格式；bria-ai 处理生成、编辑和背景移除。

## 核心概念

VGL 用确定性 JSON 替代模糊的自然语言 prompts，显式声明每个视觉属性：objects、lighting、camera settings、composition 和 style。这确保了可复现、可控的图像生成。

## 操作模式

| 模式 | 输入 | 输出 | 使用场景 |
|------|-------|--------|----------|
| **Generate** | 文本 prompt | VGL JSON | 从描述创建新图像 |
| **Edit** | 图像 + 指令 | VGL JSON | 修改参考图像 |
| **Edit_with_Mask** | 遮罩图像 + 指令 | VGL JSON | 填充灰色遮罩区域 |
| **Caption** | 仅图像 | VGL JSON | 描述现有图像 |
| **Refine** | 现有 JSON + 编辑 | 更新后的 VGL JSON | 修改现有 prompt |

## JSON Schema

输出一个包含这些必需键的单个有效 JSON 对象：

### 1. `short_description` (String)
图像内容的简洁摘要，最多 200 词。包含关键 subjects、actions、setting 和 mood。

### 2. `objects` (Array, max 5 items)
每个 object 需要：

```json
{
  "description": "Detailed description, max 100 words",
  "location": "center | top-left | bottom-right foreground | etc.",
  "relative_size": "small | medium | large within frame",
  "shape_and_color": "Basic shape and dominant color",
  "texture": "smooth | rough | metallic | furry | fabric | etc.",
  "appearance_details": "Notable visual details",
  "relationship": "Relationship to other objects",
  "orientation": "upright | tilted 45 degrees | facing left | horizontal | etc."
}
```

**Human subjects** 额外添加:
```json
{
  "pose": "Body position description",
  "expression": "winking | joyful | serious | surprised | calm",
  "clothing": "Attire description",
  "action": "What the person is doing",
  "gender": "Gender description",
  "skin_tone_and_texture": "Skin appearance"
}
```

**Object clusters** 额外添加:
```json
{
  "number_of_objects": 3
}
```

**Size guidance**: 如果人物是主体，使用 `"medium-to-large"` 或 `"large within frame"`。

### 3. `background_setting` (String)
Overall environment、setting 和不在 `objects` 中的 background elements。

### 4. `lighting` (Object)
```json
{
  "conditions": "bright daylight | dim indoor | studio lighting | golden hour | blue hour | overcast",
  "direction": "front-lit | backlit | side-lit from left | top-down",
  "shadows": "long, soft shadows | sharp, defined shadows | minimal shadows"
}
```

### 5. `aesthetics` (Object)
```json
{
  "composition": "rule of thirds | symmetrical | centered | leading lines | medium shot | close-up",
  "color_scheme": "monochromatic blue | warm complementary | high contrast | pastel",
  "mood_atmosphere": "serene | energetic | mysterious | joyful | dramatic | peaceful"
}
```
人物作为主体时，在 composition 中指定镜头类型：`"medium shot"`, `"close-up"`, `"portrait composition"`。

### 6. `photographic_characteristics` (Object)
```json
{
  "depth_of_field": "shallow | deep | bokeh background",
  "focus": "sharp focus on subject | soft focus | motion blur",
  "camera_angle": "eye-level | low angle | high angle | dutch angle | bird's-eye",
  "lens_focal_length": "wide-angle | 50mm standard | 85mm portrait | telephoto | macro"
}
```
**人物照片**: 优先使用 `"standard lens (35mm-50mm)"` 或 `"portrait lens (50mm-85mm)"`。除非特别指定，避免使用 wide-angle。

### 7. `style_medium` (String)
`"photograph"` | `"oil painting"` | `"watercolor"` | `"3D render"` | `"digital illustration"` | `"pencil sketch"`

除非明确要求，默认使用 `"photograph"`。

### 8. `artistic_style` (String)
如果不是 photograph，用最多3个词描述特征：`"impressionistic, vibrant, textured"`

如果是 photograph，使用 `"realistic"` 或类似词汇。

### 9. `context` (String)
描述图像类型/用途：
- `"High-fashion editorial photograph for magazine spread"`
- `"Concept art for fantasy video game"`
- `"Commercial product photography for e-commerce"`

### 10. `text_render` (Array)
**默认: 空数组 `[]`**

仅在用户明确提供精确文本内容时填充：
```json
{
  "text": "Exact text from user (never placeholder)",
  "location": "center | top-left | bottom",
  "size": "small | medium | large",
  "color": "white | red | blue",
  "font": "serif typeface | sans-serif | handwritten | bold impact",
  "appearance_details": "Metallic finish | 3D effect | etc."
}
```
例外：物体固有的通用文本（例如，停车标志上的 "STOP"）。

### 11. `edit_instruction` (String)
描述编辑/生成的单一祈使命令。

## 编辑指令格式

### 标准编辑（无遮罩）
以动作动词开头，描述变更，绝不引用 "original image"：

| 类别 | 重写后的指令 |
|----------|----------------------|
| 风格变更 | `Turn the image into the cartoon style.` |
| 物体属性 | `Change the dog's color to black and white.` |
| 添加元素 | `Add a wide-brimmed felt hat to the subject.` |
| 移除物体 | `Remove the book from the subject's hands.` |
| 替换物体 | `Change the rose to a bright yellow sunflower.` |
| 光照 | `Change the lighting from dark and moody to bright and vibrant.` |
| 构图 | `Change the perspective to a wider shot.` |
| 文本变更 | `Change the text "Happy Anniversary" to "Hello".` |
| 质量 | `Refine the image to obtain increased clarity and sharpness.` |

### 遮罩区域编辑
将 "masked regions" 或 "masked area" 作为目标引用：

| 意图 | 重写后的指令 |
|--------|----------------------|
| 物体生成 | `Generate a white rose with a blue center in the masked region.` |
| 扩展 | `Extend the image into the masked region to create a scene featuring...` |
| 背景填充 | `Create the following background in the masked region: A vast ocean extending to horizon.` |
| 氛围填充 | `Fill the background masked area with a clear, bright blue sky with wispy clouds.` |
| 主体修复 | `Restore the area in the mask with a young woman.` |
| 环境填充 | `Create inside the masked area: a greenhouse with rows of plants under glass ceiling.` |

## 保真度规则

### 标准编辑模式
保留所有视觉属性，除非指令明确要求更改：
- Subject identity, pose, appearance
- Object existence, location, size, orientation
- Composition, camera angle, lens characteristics
- Style/medium

只更改编辑严格需要的内容。

### 遮罩编辑模式
- 精确保留所有可见（非遮罩）部分
- 填充灰色遮罩区域以与未遮罩区域无缝融合
- 匹配现有的 style、lighting 和 subject matter
- 绝不描述灰色遮罩——描述填充它们的内容

## 示例输出

```json
{
  "short_description": "A professional businesswoman in a navy blazer stands confidently in a modern glass office, holding a tablet. Natural daylight streams through floor-to-ceiling windows, creating a warm, productive atmosphere.",
  "objects": [
    {
      "description": "A confident businesswoman in her 30s with shoulder-length dark hair, wearing a tailored navy blazer over a white blouse. She holds a tablet in her left hand while gesturing naturally with her right.",
      "location": "center-right",
      "relative_size": "large within frame",
      "shape_and_color": "Human figure, navy and white clothing",
      "texture": "smooth fabric, professional attire",
      "appearance_details": "Minimal jewelry, well-groomed professional appearance",
      "relationship": "Main subject, interacting with tablet",
      "orientation": "facing slightly left, three-quarter view",
      "pose": "Standing upright, relaxed professional stance",
      "expression": "confident, approachable smile",
      "clothing": "Tailored navy blazer, white silk blouse, dark trousers",
      "action": "Presenting or reviewing information on tablet",
      "gender": "female",
      "skin_tone_and_texture": "Medium warm skin tone, healthy smooth complexion"
    },
    {
      "description": "A modern tablet device with a bright display showing charts and graphs",
      "location": "center, held by subject",
      "relative_size": "small",
      "shape_and_color": "Rectangular, silver frame with illuminated screen",
      "texture": "smooth glass and metal",
      "appearance_details": "Thin profile, business application visible on screen",
      "relationship": "Held by businesswoman, focus of her attention",
      "orientation": "vertical, screen facing viewer at slight angle",
      "pose": null,
      "expression": null,
      "clothing": null,
      "action": null,
      "gender": null,
      "skin_tone_and_texture": null,
      "number_of_objects": null
    }
  ],
  "background_setting": "Modern corporate office interior with floor-to-ceiling windows overlooking a city skyline. Minimalist furniture in neutral tones, potted plants adding touches of green.",
  "lighting": {
    "conditions": "bright natural daylight",
    "direction": "side-lit from left through windows",
    "shadows": "soft, natural shadows"
  },
  "aesthetics": {
    "composition": "rule of thirds, medium shot",
    "color_scheme": "professional blues and neutral whites with warm accents",
    "mood_atmosphere": "confident, professional, welcoming"
  },
  "photographic_characteristics": {
    "depth_of_field": "shallow, background slightly soft",
    "focus": "sharp focus on subject's face and upper body",
    "camera_angle": "eye-level",
    "lens_focal_length": "portrait lens (85mm)"
  },
  "style_medium": "photograph",
  "artistic_style": "realistic",
  "context": "Corporate portrait photography for company website or LinkedIn professional profile.",
  "text_render": [],
  "edit_instruction": "Generate a professional businesswoman in a modern office environment holding a tablet."
}
```

## 常见陷阱

1. **不要编造文本** - 除非用户提供精确文本，否则保持 `text_render` 为空
2. **不要过度描述** - 最多5个 objects，优先处理最重要的
3. **匹配模式** - 对遮罩编辑和标准编辑使用正确的 `edit_instruction` 格式
4. **保持保真度** - 只更改明确要求的内容
5. **要具体** - 使用具体值（"85mm portrait lens"）而不是模糊术语（"nice camera"）
6. **无关字段填 null** - 非人类对象的 human-specific 字段应为 `null`

---

## 将 VGL 与 Bria API 一起使用

### 使用结构化 Prompt 生成图像

将 VGL JSON 传递给 `structured_prompt` 参数：

```python
from bria_client import BriaClient

client = BriaClient()

vgl_prompt = {
    "short_description": "Professional businesswoman in modern office...",
    "objects": [...],
    # ... 完整 VGL JSON
}

# 使用 structured_prompt 进行确定性生成
result = client.refine(
    structured_prompt=json.dumps(vgl_prompt),
    instruction="Generate this scene",
    aspect_ratio="16:9"
)
print(result['result']['image_url'])
```

### 优化现有生成

生成后，Bria 返回一个 `structured_prompt`，你可以修改并重新生成：

```python
# 初始生成
result = client.generate("A cozy coffee shop interior")
structured = result['result']['structured_prompt']

# 修改并重新生成
result = client.refine(
    structured_prompt=structured,
    instruction="Change the lighting to golden hour"
)
```

### curl 示例

```bash
curl -X POST "https://engine.prod.bria-api.com/v2/image/generate" \
  -H "api_token: $BRIA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "structured_prompt": "{\"short_description\": \"...\", ...}",
    "prompt": "Generate this scene",
    "aspect_ratio": "16:9"
  }'
```

---

## 参考

- **[Schema Reference](references/schema-reference.md)** - 包含所有参数值的完整 JSON schema
- **[bria-ai](../bria-ai/SKILL.md)** - 用于执行 VGL prompts 的 API 客户端和端点文档
