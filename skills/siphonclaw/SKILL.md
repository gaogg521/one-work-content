---
name: siphonclaw
description: 具有视觉搜索、OCR 和现场捕获的文档智能管道
version: 1.0.0
metadata:
  siphonclaw:
    emoji: 🔍
    requires:
      plugins: []
tags:
- 图像处理
- 文档
---

# SiphonClaw

域无关的文档智能管道。将 PDF、图像和电子表格摄取到可搜索的知识库中，具有双轨检索（文本 + 视觉）、OCR、置信度评分和现场捕获。

为现场服务工程师、研究人员、机械师和任何需要从大型文档集合中获取快速答案的人构建。

## SiphonClaw 的功能

- **Ingest** 文档（PDF、Excel、图像、屏幕截图）到具有文本和视觉嵌入的本地向量数据库中
- **Search** 使用三重混合检索：BM25 关键词匹配 + 语义文本向量 + 视觉页面嵌入，使用 RRF 融合并使用交叉编码器重新排序
- **Identify** 使用视觉模型从照片中识别设备、零件或组件，然后搜索本地知识库
- **Capture** 现场修复和维修记录作为未来检索的一等知识库条目
- **Score** 每个响应都具有复合置信度（检索 + 忠实度 + 相关性 + 覆盖率）和脚注样式来源引用

## MCP 工具

SiphonClaw 通过 MCP 暴露五个工具，用于与代理和其他 MCP 兼容客户端集成。

---

### siphonclaw_search

使用三重混合检索（文本 + 视觉 + 关键词）搜索知识库。

**参数：**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `query` | string | yes | 自然语言搜索查询或确切的零件号 / 错误代码 |
| `top_k` | integer | no | 返回的结果数量（默认：5，最大：20） |
| `filters` | object | no | 元数据过滤器（例如，`{"source_type": "service_manual", "model": "ModelA"}`） |
| `mode` | string | no | 搜索模式：`"hybrid"`（默认）、`"text"`、 `"visual"`、 `"keyword"` |

**返回：**

```json
{
  "results": [
    {
      "content": "Extracted text from the matching chunk or page",
      "source": "ServiceManual_ModelA.pdf",
      "page": 42,
      "section": "4.3 Transformer Replacement",
      "score": 0.92,
      "match_type": "hybrid"
    }
  ],
  "confidence": 0.87,
  "confidence_tier": "Confident - verify part number",
  "keywords_used": ["low voltage supply", "assembly mount", "ModelA"],
  "citations": ["[1] ServiceManual_ModelA, page 42", "[2] Parts Catalog PC-1102, page 15"]
}
```

---

### siphonclaw_ingest

将文档或照片添加到知识库。支持 PDF、Excel、图像（JPG/PNG）和屏幕截图。

**参数：**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `file_path` | string | yes | 要摄取的文件的绝对路径 |
| `source_type` | string | no | 文档类型提示：`"manual"`、 `"parts_catalog"`、 `"field_note"`、 `"photo"`、 `"other"`（默认：自动检测） |
| `metadata` | object | no | 要附加的其他元数据（例如，`{"model": "ModelA", "domain": "industrial"}`） |

**返回：**

```json
{
  "status": "ingested",
  "file": "ServiceManual_ModelA.pdf",
  "pages_processed": 127,
  "chunks_created": 843,
  "visual_pages_indexed": 127,
  "ocr_pages": 12,
  "duration_seconds": 45.2
}
```

---

### siphonclaw_field_note

将现场修复或维修记录保存为一等知识库条目。这些被索引并可在未来的搜索中检索，形成一个学习循环。

**参数：**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `note` | string | yes | 修复、程序或观察的自由文本描述 |
| `model` | string | no | 设备型号或标识符（例如，`"ModelA"`） |
| `parts` | array[string] | no | 维修中使用的零件号（例如，`["12345", "67890"]`） |
| `procedure_ref` | string | no | 对手册程序的引用（例如，`"ServiceManual_ModelA section 4.3"`） |
| `tags` | array[string] | no | 用于分类的自由格式标签（例如，`["hv_transformer", "calibration"]`） |

**返回：**

```json
{
  "status": "saved",
  "field_note_id": "fn-2026-02-09-001",
  "indexed": true,
  "model": "ModelA",
  "parts_cross_referenced": ["12345"],
  "retrievable": true
}
```

---

### siphonclaw_identify

发送设备、零件、标签或错误屏幕的照片。SiphonClaw 使用视觉模型来识别它所看到的内容，然后搜索本地知识库以获取相关文档。如果本地置信度较低，则回退到网络搜索。

**参数：**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `image_path` | string | yes | 图像文件的绝对路径（JPG、PNG、HEIC） |
| `context` | string | no | 关于图像的其他上下文（例如，`"circuit board inside equipment housing"`） |
| `search_after` | boolean | no | 识别后自动搜索 KB（默认：`true`） |

**返回：**

```json
{
  "identification": "Industrial power supply board, Model PSU-200",
  "visual_features": ["green PCB", "3 large capacitors", "manufacturer logo visible", "part label partially obscured"],
  "ocr_text": "PSU-200 REV C  SN: 4829103",
  "search_results": [
    {
      "content": "PSU-200 replacement procedure...",
      "source": "ServiceManual_ModelA.pdf",
      "page": 67,
      "score": 0.94
    }
  ],
  "confidence": 0.91,
  "web_search_used": false
}
```

---

### siphonclaw_status

获取管道健康、摄取统计、模型可用性和成本跟踪。

**参数：**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `detail` | string | no | 详细程度：`"summary"`（默认）、`"full"`、 `"costs"`、 `"models"` |

**返回：**

```json
{
  "status": "healthy",
  "knowledge_base": {
    "total_documents": 3164,
    "total_chunks": 656000,
    "visual_pages_indexed": 31200,
    "last_ingestion": "2026-02-09T14:30:00Z"
  },
  "models": {
    "ocr": {"model": "qwen3-vl:latest", "provider": "ollama", "available": true},
    "text_embedding": {"model": "bge-m3:latest", "provider": "ollama", "available": true},
    "visual_embedding": {"model": "qwen3-vl-embed:2b", "provider": "ollama", "available": true},
    "generation": {"model": "MiniMax-M2.5", "provider": "openrouter", "available": true},
    "reasoning": {"model": "kimi-k2.5", "provider": "openrouter", "available": true},
    "fallback": {"model": "glm-4.7-flash:latest", "provider": "ollama", "available": true}
  },
  "costs": {
    "today": "$0.12",
    "this_month": "$2.45",
    "daily_budget": "$5.00",
    "budget_remaining": "$4.88"
  },
  "dead_letter_queue": {
    "pending_retry": 2,
    "permanently_failed": 1
  }
}
```

## 设置

### 模式 A：混合本地 + 云（推荐）

本地模型免费处理摄取（OCR + 嵌入）。云 API 以每次查询几美分的价格处理智能（生成 + 推理）。

**每月费用：典型使用约 $0.50-5/月。**

```bash
# 1. 安装 SiphonClaw
pip install siphonclaw
# 或：openclaw skill install siphonclaw
# 或：git clone https://github.com/openclaw/siphonclaw && pip install -r requirements.txt

# 2. 安装 Ollama 并拉取本地模型（总共约 10 GB）
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen3-vl:latest          # 6.1 GB - OCR
ollama pull bge-m3:latest             # ~1.5 GB - 文本嵌入
ollama pull qwen3-vl-embed:2b        # ~2 GB - 视觉嵌入

# 3. 获取 OpenRouter API 密钥（一个密钥用于所有智能模型）
#    访问：https://openrouter.ai -> 注册 -> 复制 API 密钥
siphonclaw config set openrouter_key sk-or-v1-xxxxx

# 4. （可选）获取 Brave Search API 密钥以进行网络搜索回退
#    访问：https://brave.com/search/api -> 注册 -> 免费层：每月 2,000 次查询
siphonclaw config set brave_key BSA-xxxxx

# 5. 指向你的文档并摄取
siphonclaw config set docs_path /path/to/my/docs
siphonclaw ingest

# 6. 搜索
siphonclaw search "part number for compressor valve"
```

### 模式 B：全云

所有内容都通过 OpenRouter 运行。设置更简单（无需 Ollama），但大量文档集的摄取成本为 $50-100+ 的 API 令牌。

**第一个月：~$50-105。之后：~$0.50/月。**

```bash
# 1. 安装 SiphonClaw
pip install siphonclaw

# 2. 获取 OpenRouter API 密钥
siphonclaw config set openrouter_key sk-or-v1-xxxxx

# 3. 将摄取模式设置为云
siphonclaw config set ingestion_mode cloud

# 4. （可选）获取 Brave Search API 密钥
siphonclaw config set brave_key BSA-xxxxx

# 5. 指向你的文档并摄取
siphonclaw config set docs_path /path/to/my/docs
siphonclaw ingest

# 6. 搜索
siphonclaw search "part number for compressor valve"
```

### 成本比较

| Operation | Mode A (Hybrid) | Mode B (Full Cloud) |
|-----------|-----------------|---------------------|
| Ingest 3,000 PDFs | $0 (local) | ~$50-100 (OCR + embeddings) |
| 100 searches/month | ~$0.50 (API generation) | ~$0.50 (same) |
| Monthly total | **~$0.50-5/mo** | **~$50-105 first month, $0.50/mo after** |

## 配置参考

SiphonClaw 从 `config/models.yaml` 和环境变量读取配置。

**环境变量（通过 `.env` 或 shell）：**

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENROUTER_API_KEY` | Mode A/B | 智能模型的 OpenRouter API 密钥 |
| `BRAVE_SEARCH_API_KEY` | no | 网络搜索回退的 Brave Search API 密钥 |
| `OLLAMA_BASE_URL` | no | Ollama 服务器 URL（默认：`http://127.0.0.1:11434`） |
| `SIPHONCLAW_BUDGET_DAILY` | no | 每日 API 支出上限（美元）（默认：`5.00`） |
| `SIPHONCLAW_DOCS_PATH` | no | 用于摄取的文档目录路径 |

**代理配置示例 (`config.json`)：**

```json
{
  "skills": {
    "entries": {
      "siphonclaw": {
        "openrouter_key": "sk-or-v1-xxxxx",
        "brave_key": "BSA-xxxxx",
        "docs_path": "/path/to/docs",
        "ingestion_mode": "local",
        "ollama_url": "http://127.0.0.1:11434"
      }
    }
  }
}
```

**模型配置：** 有关具有摄取和智能设置的完整模型层配置，请参见 `config/models.yaml`。
