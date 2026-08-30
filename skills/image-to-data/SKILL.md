---
name: image-to-data
description: 使用 AI Vision 从建筑图像中提取数据。分析工地照片、扫描文档、图纸。
tags:
- AI
---

# Image To Data

## 概述

基于 DDC 方法论（第 2.4 章），此 skill 使用计算机视觉、OCR 和 AI 模型从建筑图像中提取结构化数据，以分析工地照片、扫描文档和图纸。

**书籍参考：** "Преобразование данных в структурированную форму" / "Data Transformation to Structured Form"

## 快速开始

```python
from dataclasses import dataclass, field
from enum import Enum
from typing import List, Dict, Optional, Any, Tuple
from datetime import datetime
import json
import base64

class ImageType(Enum):
    """建筑图像的类型"""
    SITE_PHOTO = "site_photo"
    SCANNED_DOCUMENT = "scanned_document"
    FLOOR_PLAN = "floor_plan"
    ELEVATION = "elevation"
    DETAIL_DRAWING = "detail_drawing"
    PROGRESS_PHOTO = "progress_photo"
    SAFETY_PHOTO = "safety_photo"
    DEFECT_PHOTO = "defect_photo"
    MATERIAL_PHOTO = "material_photo"
    EQUIPMENT_PHOTO = "equipment_photo"

class ExtractionType(Enum):
    """数据提取的类型"""
    OCR_TEXT = "ocr_text"
    TABLE = "table"
    OBJECT_DETECTION = "object_detection"
    MEASUREMENT = "measurement"
    CLASSIFICATION = "classification"
    PROGRESS = "progress"

@dataclass
class BoundingBox:
    """检测区域的边界框"""
    x: int
    y: int
    width: int
    height: int
    confidence: float = 1.0

@dataclass
class TextRegion:
    """从图像中提取的文本区域"""
    text: str
    bbox: BoundingBox
    confidence: float
    language: str = "en"

@dataclass
class DetectedObject:
    """图像中检测到的对象"""
    label: str
    bbox: BoundingBox
    confidence: float
    attributes: Dict[str, Any] = field(default_factory=dict)

@dataclass
class ExtractedTable:
    """从图像中提取的表格"""
    headers: List[str]
    rows: List[List[str]]
    bbox: BoundingBox
    confidence: float

@dataclass
class ProgressMeasurement:
    """从图像中获取的进度测量"""
    element_type: str
    total_count: int
    completed_count: int
    percent_complete: float
    area_sqft: Optional[float] = None
    volume_cuft: Optional[float] = None

@dataclass
class ImageAnalysisResult:
    """完整的图像分析结果"""
    image_id: str
    image_type: ImageType
    text_regions: List[TextRegion]
    detected_objects: List[DetectedObject]
    tables: List[ExtractedTable]
    progress: Optional[ProgressMeasurement] = None
    metadata: Dict[str, Any] = field(default_factory=dict)
    processing_time: float = 0.0


class OCREngine:
    """用于文本提取的 OCR 引擎"""

    def __init__(self, engine: str = "tesseract"):
        self.engine = engine
        self.supported_languages = ["en", "ru", "de", "fr", "es"]

    def extract_text(
        self,
        image_data: bytes,
        language: str = "en"
    ) -> List[TextRegion]:
        """从图像中提取文本"""
        # 模拟 OCR 提取（在生产环境中使用实际的 OCR 库）
        # 生产环境：pytesseract, EasyOCR, 或云 OCR 服务

        regions = []

        # 模拟检测图纸中的标题栏
        regions.append(TextRegion(
            text="PROJECT: OFFICE BUILDING",
            bbox=BoundingBox(x=100, y=50, width=300, height=30, confidence=0.95),
            confidence=0.95,
            language=language
        ))

        regions.append(TextRegion(
            text="DRAWING: A-101",
            bbox=BoundingBox(x=100, y=90, width=200, height=25, confidence=0.92),
            confidence=0.92,
            language=language
        ))

        regions.append(TextRegion(
            text="SCALE: 1:100",
            bbox=BoundingBox(x=100, y=120, width=150, height=20, confidence=0.88),
            confidence=0.88,
            language=language
        ))

        return regions

    def extract_structured_text(
        self,
        image_data: bytes,
        template: Optional[Dict] = None
    ) -> Dict[str, str]:
        """使用模板匹配提取结构化文本"""
        # 提取文本区域
        regions = self.extract_text(image_data)

        # 匹配到模板字段
        structured = {}

        if template:
            for field_name, field_config in template.items():
                # 查找匹配的区域
                for region in regions:
                    if field_config.get("keyword") in region.text.lower():
                        structured[field_name] = region.text
                        break
        else:
            # 默认提取
            for region in regions:
                if "PROJECT:" in region.text:
                    structured["project_name"] = region.text.split(":")[-1].strip()
                elif "DRAWING:" in region.text:
                    structured["drawing_number"] = region.text.split(":")[-1].strip()
                elif "SCALE:" in region.text:
                    structured["scale"] = region.text.split(":")[-1].strip()

        return structured


class ObjectDetector:
    """建筑图像的对象检测"""

    def __init__(self, model: str = "yolov8"):
        self.model = model
        self.construction_classes = self._load_construction_classes()

    def _load_construction_classes(self) -> Dict[str, Dict]:
        """加载建筑特定的对象类别"""
        return {
            # 设备
            "excavator": {"category": "equipment", "safety_zone": 20},
            "crane": {"category": "equipment", "safety_zone": 30},
            "forklift": {"category": "equipment", "safety_zone": 10},
            "concrete_mixer": {"category": "equipment", "safety_zone": 5},
            "scaffolding": {"category": "equipment", "safety_zone": 5},

            # 安全
            "hard_hat": {"category": "ppe", "required": True},
            "safety_vest": {"category": "ppe", "required": True},
            "safety_glasses": {"category": "ppe", "required": False},
            "harness": {"category": "ppe", "required": False},

            # 材料
            "rebar_bundle": {"category": "material", "unit": "bundle"},
            "concrete_block": {"category": "material", "unit": "pallet"},
            "lumber_stack": {"category": "material", "unit": "bundle"},
            "pipe_stack": {"category": "material", "unit": "bundle"},

            # 工人
            "worker": {"category": "person", "track": True},

            # 建筑元素
            "column": {"category": "structure"},
            "beam": {"category": "structure"},
            "slab": {"category": "structure"},
            "wall": {"category": "structure"},
        }

    def detect(
        self,
        image_data: bytes,
        confidence_threshold: float = 0.5
    ) -> List[DetectedObject]:
        """检测图像中的对象"""
        # 模拟检测（在生产环境中使用实际模型）
        # 生产环境：YOLO, Faster R-CNN, 等

        detected = []

        # 模拟检测到的对象
        sample_detections = [
            ("worker", 0.92, BoundingBox(200, 300, 80, 180, 0.92)),
            ("hard_hat", 0.88, BoundingBox(210, 300, 30, 25, 0.88)),
            ("safety_vest", 0.85, BoundingBox(210, 340, 60, 80, 0.85)),
            ("scaffolding", 0.78, BoundingBox(400, 100, 200, 400, 0.78)),
            ("concrete_block", 0.72, BoundingBox(50, 450, 100, 50, 0.72)),
        ]

        for label, conf, bbox in sample_detections:
            if conf >= confidence_threshold:
                class_info = self.construction_classes.get(label, {})
                detected.append(DetectedObject(
                    label=label,
                    bbox=bbox,
                    confidence=conf,
                    attributes=class_info
                ))

        return detected

    def detect_safety_compliance(
        self,
        image_data: bytes
    ) -> Dict:
        """检测图像中的安全合规性"""
        objects = self.detect(image_data)

        workers = [o for o in objects if o.label == "worker"]
        hard_hats = [o for o in objects if o.label == "hard_hat"]
        vests = [o for o in objects if o.label == "safety_vest"]

        compliance = {
            "workers_detected": len(workers),
            "hard_hats_detected": len(hard_hats),
            "vests_detected": len(vests),
            "hard_hat_compliance": len(hard_hats) / len(workers) if workers else 1.0,
            "vest_compliance": len(vests) / len(workers) if workers else 1.0,
            "overall_compliance": "compliant" if len(hard_hats) >= len(workers) else "non-compliant",
            "violations": []
        }

        if len(hard_hats) < len(workers):
            compliance["violations"].append({
                "type": "missing_hard_hat",
                "count": len(workers) - len(hard_hats)
            })

        return compliance


class TableExtractor:
    """从图像中提取表格"""

    def extract_tables(
        self,
        image_data: bytes,
        detect_headers: bool = True
    ) -> List[ExtractedTable]:
        """从图像中提取表格"""
        # 模拟表格提取
        # 生产环境：Camelot, Tabula, 或自定义 CNN

        tables = []

        # 模拟一个进度表
        tables.append(ExtractedTable(
            headers=["Activity", "Start", "End", "Duration"],
            rows=[
                ["Foundation", "2024-01-01", "2024-01-15", "14 days"],
                ["Framing", "2024-01-16", "2024-02-28", "44 days"],
                ["MEP Rough-in", "2024-03-01", "2024-03-31", "31 days"]
            ],
            bbox=BoundingBox(50, 200, 500, 200, 0.85),
            confidence=0.85
        ))

        return tables

    def table_to_dataframe(self, table: ExtractedTable) -> Dict:
        """将表格转换为字典（类 DataFrame）"""
        return {
            "columns": table.headers,
            "data": table.rows,
            "records": [
                dict(zip(table.headers, row))
                for row in table.rows
            ]
        }


class ProgressAnalyzer:
    """从图像分析建筑进度"""

    def __init__(self):
        self.reference_models = {}

    def analyze_progress(
        self,
        current_image: bytes,
        reference_image: Optional[bytes] = None,
        element_type: str = "general"
    ) -> ProgressMeasurement:
        """通过比较图像分析进度"""
        # 模拟进度分析
        # 生产环境：使用语义分割 + 比较

        # 模拟进度检测
        return ProgressMeasurement(
            element_type=element_type,
            total_count=100,
            completed_count=65,
            percent_complete=65.0,
            area_sqft=15000.0,
            volume_cuft=None
        )

    def compare_with_plan(
        self,
        site_photo: bytes,
        plan_image: bytes
    ) -> Dict:
        """将工地照片与图纸进行比较"""
        return {
            "match_score": 0.78,
            "deviations": [],
            "completion_estimate": 65.0,
            "areas_of_concern": []
        }


class ConstructionImageAnalyzer:
    """
    建筑图像分析的主类。
    基于 DDC 方法论第 2.4 章。
    """

    def __init__(self):
        self.ocr = OCREngine()
        self.detector = ObjectDetector()
        self.table_extractor = TableExtractor()
        self.progress_analyzer = ProgressAnalyzer()

    def analyze_image(
        self,
        image_data: bytes,
        image_type: ImageType,
        image_id: str = "img_001",
        extract_types: Optional[List[ExtractionType]] = None
    ) -> ImageAnalysisResult:
        """
        分析建筑图像。

        Args:
            image_data: 图像数据作为 bytes
            image_type: 图像类型
            image_id: 唯一图像标识符
            extract_types: 要执行的提取类型

        Returns:
            完整的分析结果
        """
        start_time = datetime.now()

        if extract_types is None:
            extract_types = [ExtractionType.OCR_TEXT, ExtractionType.OBJECT_DETECTION]

        text_regions = []
        detected_objects = []
        tables = []
        progress = None

        # OCR 提取
        if ExtractionType.OCR_TEXT in extract_types:
            text_regions = self.ocr.extract_text(image_data)

        # 对象检测
        if ExtractionType.OBJECT_DETECTION in extract_types:
            detected_objects = self.detector.detect(image_data)

        # 表格提取
        if ExtractionType.TABLE in extract_types:
            tables = self.table_extractor.extract_tables(image_data)

        # 进度分析
        if ExtractionType.PROGRESS in extract_types:
            progress = self.progress_analyzer.analyze_progress(image_data)

        processing_time = (datetime.now() - start_time).total_seconds()

        return ImageAnalysisResult(
            image_id=image_id,
            image_type=image_type,
            text_regions=text_regions,
            detected_objects=detected_objects,
            tables=tables,
            progress=progress,
            metadata={"extraction_types": [e.value for e in extract_types]},
            processing_time=processing_time
        )

    def analyze_site_photo(
        self,
        image_data: bytes,
        image_id: str = "site_001"
    ) -> Dict:
        """分析工地照片以获取进度和安全信息"""
        result = self.analyze_image(
            image_data,
            ImageType.SITE_PHOTO,
            image_id,
            [ExtractionType.OBJECT_DETECTION, ExtractionType.PROGRESS]
        )

        safety = self.detector.detect_safety_compliance(image_data)

        return {
            "image_id": result.image_id,
            "objects_detected": len(result.detected_objects),
            "progress": result.progress,
            "safety_compliance": safety,
            "equipment": [o.label for o in result.detected_objects if o.attributes.get("category") == "equipment"],
            "materials": [o.label for o in result.detected_objects if o.attributes.get("category") == "material"]
        }

    def extract_drawing_data(
        self,
        image_data: bytes,
        image_id: str = "dwg_001"
    ) -> Dict:
        """从扫描图纸中提取数据"""
        result = self.analyze_image(
            image_data,
            ImageType.FLOOR_PLAN,
            image_id,
            [ExtractionType.OCR_TEXT, ExtractionType.TABLE]
        )

        # 提取标题栏信息
        title_block = self.ocr.extract_structured_text(image_data)

        return {
            "image_id": result.image_id,
            "title_block": title_block,
            "text_regions": len(result.text_regions),
            "tables": [
                self.table_extractor.table_to_dataframe(t)
                for t in result.tables
            ],
            "all_text": [r.text for r in result.text_regions]
        }

    def batch_analyze(
        self,
        images: List[Tuple[bytes, ImageType, str]]
    ) -> List[ImageAnalysisResult]:
        """分析多张图像"""
        results = []
        for image_data, image_type, image_id in images:
            result = self.analyze_image(image_data, image_type, image_id)
            results.append(result)
        return results

    def export_results(
        self,
        result: ImageAnalysisResult,
        format: str = "json"
    ) -> str:
        """导出分析结果"""
        data = {
            "image_id": result.image_id,
            "image_type": result.image_type.value,
            "text_count": len(result.text_regions),
            "object_count": len(result.detected_objects),
            "table_count": len(result.tables),
            "texts": [
                {"text": r.text, "confidence": r.confidence}
                for r in result.text_regions
            ],
            "objects": [
                {"label": o.label, "confidence": o.confidence}
                for o in result.detected_objects
            ],
            "processing_time": result.processing_time
        }

        if format == "json":
            return json.dumps(data, indent=2)
        else:
            raise ValueError(f"Unsupported format: {format}")
```

## 常见用例

### 分析工地照片

```python
analyzer = ConstructionImageAnalyzer()

# 加载图像（在生产环境中，从文件读取）
with open("site_photo.jpg", "rb") as f:
    image_data = f.read()

result = analyzer.analyze_site_photo(image_data)

print(f"Objects detected: {result['objects_detected']}")
print(f"Safety compliance: {result['safety_compliance']['overall_compliance']}")
print(f"Progress: {result['progress'].percent_complete}%")
```

### 提取图纸数据

```python
with open("floor_plan.png", "rb") as f:
    drawing_data = f.read()

data = analyzer.extract_drawing_data(drawing_data)

print(f"Drawing: {data['title_block'].get('drawing_number')}")
print(f"Project: {data['title_block'].get('project_name')}")
for table in data['tables']:
    print(f"Table with {len(table['records'])} rows")
```

### 检测安全违规

```python
detector = ObjectDetector()

with open("site_photo.jpg", "rb") as f:
    image_data = f.read()

safety = detector.detect_safety_compliance(image_data)

if safety['overall_compliance'] == 'non-compliant':
    for violation in safety['violations']:
        print(f"Violation: {violation['type']} - Count: {violation['count']}")
```

## 快速参考

| 组件 | 用途 |
|-----------|---------|
| `ConstructionImageAnalyzer` | 主分析引擎 |
| `OCREngine` | 文本提取 |
| `ObjectDetector` | 对象检测 |
| `TableExtractor` | 表格提取 |
| `ProgressAnalyzer` | 进度分析 |
| `ImageAnalysisResult` | 完整分析结果 |

## 资源

- **书籍**: "Data-Driven Construction" by Artem Boiko, Chapter 2.4
- **网站**: https://datadrivenconstruction.io

## 下一步

- 使用 [cad-to-data](../cad-to-data/SKILL.md) 进行 CAD/BIM 提取
- 使用 [defect-detection-ai](../../../DDC_Innovative/defect-detection-ai/SKILL.md) 进行缺陷检测
- 使用 [safety-compliance-checker](../../../DDC_Innovative/safety-compliance-checker/SKILL.md) 进行安全检查
