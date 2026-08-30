---
name: bim-qto
slug: bim-qto
display_name: BIM QTO
description: 从 BIM/CAD 数据中提取数量以进行成本估算。按 type、level、zone 分组。生成 QTO 报告。
tags:
- 建筑
- 数据
---

# BIM Quantity Takeoff

## 概述
Quantity Takeoff (QTO) 从 BIM 模型中提取可测量的数量。本技能处理 BIM 导出文件，生成按组分类的数量报告，用于成本估算。

## Python 实现

```python
import pandas as pd
import numpy as np
from typing import Dict, Any, List, Optional, Tuple
from dataclasses import dataclass, field
from enum import Enum


class QTOUnit(Enum):
    """Quantity takeoff 测量单位。"""
    COUNT = "ea"
    LENGTH = "m"
    AREA = "m2"
    VOLUME = "m3"
    WEIGHT = "kg"
    LINEAR_FOOT = "lf"
    SQUARE_FOOT = "sf"
    CUBIC_YARD = "cy"


@dataclass
class QTOItem:
    """单个 QTO line item。"""
    category: str
    type_name: str
    description: str
    quantity: float
    unit: str
    level: Optional[str] = None
    material: Optional[str] = None
    element_count: int = 0


@dataclass
class QTOReport:
    """完整的 QTO 报告。"""
    project_name: str
    items: List[QTOItem]
    total_elements: int
    categories: int
    generated_date: str


class BIMQuantityTakeoff:
    """从 BIM 数据中提取数量。"""

    # 不同 BIM 导出的列映射
    COLUMN_MAPPINGS = {
        'type': ['Type Name', 'TypeName', 'type_name', 'Family and Type', 'IfcType'],
        'category': ['Category', 'category', 'IfcClass', 'Element Category'],
        'level': ['Level', 'level', 'Building Storey', 'BuildingStorey', 'Floor'],
        'volume': ['Volume', 'volume', 'Volume (m³)', 'Qty_Volume'],
        'area': ['Area', 'area', 'Surface Area', 'Area (m²)', 'Qty_Area'],
        'length': ['Length', 'length', 'Length (m)', 'Qty_Length'],
        'count': ['Count', 'count', 'Quantity', 'ElementCount'],
        'material': ['Material', 'material', 'Structural Material', 'MaterialName']
    }

    def __init__(self, df: pd.DataFrame):
        """使用 BIM 数据 DataFrame 初始化。"""
        self.df = df
        self.column_map = self._detect_columns()

    def _detect_columns(self) -> Dict[str, str]:
        """检测数据中存在的列。"""
        mapping = {}

        for standard, variants in self.COLUMN_MAPPINGS.items():
            for variant in variants:
                if variant in self.df.columns:
                    mapping[standard] = variant
                    break

        return mapping

    def get_column(self, standard_name: str) -> Optional[str]:
        """从标准名称获取实际列名。"""
        return self.column_map.get(standard_name)

    def group_by_type(self, sum_column: str = 'volume') -> pd.DataFrame:
        """按 type name 分组数量。"""

        type_col = self.get_column('type')
        qty_col = self.get_column(sum_column)

        if type_col is None:
            raise ValueError("Type column not found")

        if qty_col is None:
            # 回退到 count
            result = self.df.groupby(type_col).size().reset_index(name='count')
        else:
            result = self.df.groupby(type_col).agg({
                qty_col: 'sum'
            }).reset_index()
            result['count'] = self.df.groupby(type_col).size().values

        result.columns = ['Type', 'Quantity', 'Count'] if len(result.columns) == 3 else ['Type', 'Count']
        return result.sort_values('Count', ascending=False)

    def group_by_category(self, sum_column: str = 'volume') -> pd.DataFrame:
        """按 category 分组数量。"""

        cat_col = self.get_column('category')
        qty_col = self.get_column(sum_column)

        if cat_col is None:
            raise ValueError("Category column not found")

        agg_dict = {}
        if qty_col:
            agg_dict[qty_col] = 'sum'

        if agg_dict:
            result = self.df.groupby(cat_col).agg(agg_dict).reset_index()
            result['count'] = self.df.groupby(cat_col).size().values
        else:
            result = self.df.groupby(cat_col).size().reset_index(name='count')

        return result.sort_values('count', ascending=False)

    def group_by_level(self, sum_column: str = 'volume') -> pd.DataFrame:
        """按 building level 分组数量。"""

        level_col = self.get_column('level')
        qty_col = self.get_column(sum_column)

        if level_col is None:
            raise ValueError("Level column not found")

        agg_dict = {}
        if qty_col:
            agg_dict[qty_col] = 'sum'

        if agg_dict:
            result = self.df.groupby(level_col).agg(agg_dict).reset_index()
            result['count'] = self.df.groupby(level_col).size().values
        else:
            result = self.df.groupby(level_col).size().reset_index(name='count')

        return result

    def pivot_by_level_and_type(self) -> pd.DataFrame:
        """创建 pivot table：levels 作为行，types 作为列。"""

        level_col = self.get_column('level')
        type_col = self.get_column('type')

        if level_col is None or type_col is None:
            raise ValueError("Level or Type column not found")

        pivot = pd.crosstab(
            self.df[level_col],
            self.df[type_col],
            margins=True
        )

        return pivot

    def filter_by_category(self, categories: List[str]) -> 'BIMQuantityTakeoff':
        """过滤到特定 categories。"""

        cat_col = self.get_column('category')
        if cat_col is None:
            raise ValueError("Category column not found")

        filtered_df = self.df[self.df[cat_col].isin(categories)]
        return BIMQuantityTakeoff(filtered_df)

    def filter_by_level(self, levels: List[str]) -> 'BIMQuantityTakeoff':
        """过滤到特定 levels。"""

        level_col = self.get_column('level')
        if level_col is None:
            raise ValueError("Level column not found")

        filtered_df = self.df[self.df[level_col].isin(levels)]
        return BIMQuantityTakeoff(filtered_df)

    def get_walls(self) -> pd.DataFrame:
        """获取 wall 数量。"""
        cat_col = self.get_column('category')
        if cat_col:
            walls = self.df[self.df[cat_col].str.contains('Wall', case=False, na=False)]
            return BIMQuantityTakeoff(walls).group_by_type()
        return pd.DataFrame()

    def get_floors(self) -> pd.DataFrame:
        """获取 floor/slab 数量。"""
        cat_col = self.get_column('category')
        if cat_col:
            floors = self.df[self.df[cat_col].str.contains('Floor|Slab', case=False, na=False)]
            return BIMQuantityTakeoff(floors).group_by_type()
        return pd.DataFrame()

    def get_doors(self) -> pd.DataFrame:
        """获取 door 数量。"""
        cat_col = self.get_column('category')
        if cat_col:
            doors = self.df[self.df[cat_col].str.contains('Door', case=False, na=False)]
            return BIMQuantityTakeoff(doors).group_by_type()
        return pd.DataFrame()

    def get_windows(self) -> pd.DataFrame:
        """获取 window 数量。"""
        cat_col = self.get_column('category')
        if cat_col:
            windows = self.df[self.df[cat_col].str.contains('Window', case=False, na=False)]
            return BIMQuantityTakeoff(windows).group_by_type()
        return pd.DataFrame()

    def generate_report(self, project_name: str = "Project") -> QTOReport:
        """生成完整的 QTO 报告。"""

        from datetime import datetime

        items = []
        type_col = self.get_column('type')
        cat_col = self.get_column('category')
        level_col = self.get_column('level')
        vol_col = self.get_column('volume')
        area_col = self.get_column('area')
        mat_col = self.get_column('material')

        # 按 type 分组
        grouped = self.df.groupby(type_col if type_col else self.df.columns[0])

        for type_name, group in grouped:
            # 确定主要数量
            qty = 0
            unit = QTOUnit.COUNT.value

            if vol_col and vol_col in group.columns:
                qty = group[vol_col].sum()
                unit = QTOUnit.VOLUME.value
            elif area_col and area_col in group.columns:
                qty = group[area_col].sum()
                unit = QTOUnit.AREA.value
            else:
                qty = len(group)
                unit = QTOUnit.COUNT.value

            # 获取 category 和 material
            category = group[cat_col].iloc[0] if cat_col and cat_col in group.columns else ""
            material = group[mat_col].iloc[0] if mat_col and mat_col in group.columns else ""
            level = group[level_col].iloc[0] if level_col and level_col in group.columns else ""

            items.append(QTOItem(
                category=str(category),
                type_name=str(type_name),
                description=str(type_name),
                quantity=round(qty, 2),
                unit=unit,
                level=str(level) if level else None,
                material=str(material) if material else None,
                element_count=len(group)
            ))

        return QTOReport(
            project_name=project_name,
            items=items,
            total_elements=len(self.df),
            categories=self.df[cat_col].nunique() if cat_col else 0,
            generated_date=datetime.now().isoformat()
        )

    def to_excel(self, output_path: str, project_name: str = "Project"):
        """导出 QTO 到 Excel，包含多个 sheets。"""

        with pd.ExcelWriter(output_path, engine='openpyxl') as writer:
            # 按 category 汇总
            self.group_by_category().to_excel(
                writer, sheet_name='By Category', index=False)

            # 按 type 汇总
            self.group_by_type().to_excel(
                writer, sheet_name='By Type', index=False)

            # Level 细分
            try:
                self.pivot_by_level_and_type().to_excel(
                    writer, sheet_name='Level-Type Matrix')
            except:
                pass

            # Walls
            walls = self.get_walls()
            if not walls.empty:
                walls.to_excel(writer, sheet_name='Walls', index=False)

            # Doors and Windows
            doors = self.get_doors()
            if not doors.empty:
                doors.to_excel(writer, sheet_name='Doors', index=False)

            windows = self.get_windows()
            if not windows.empty:
                windows.to_excel(writer, sheet_name='Windows', index=False)

        return output_path
```

## 快速开始

```python
# 加载 BIM 导出
df = pd.read_excel("revit_export.xlsx")

# 初始化 QTO
qto = BIMQuantityTakeoff(df)

# 按 type 获取数量
by_type = qto.group_by_type()
print(by_type.head(10))

# 获取 wall schedule
walls = qto.get_walls()
print(walls)
```

## 常见使用场景

### 1. 完整 QTO 报告
```python
qto = BIMQuantityTakeoff(df)
report = qto.generate_report("Office Building")
print(f"Elements: {report.total_elements}")
for item in report.items[:5]:
    print(f"{item.type_name}: {item.quantity} {item.unit}")
```

### 2. 逐层分析
```python
pivot = qto.pivot_by_level_and_type()
print(pivot)
```

### 3. 导出到 Excel
```python
qto.to_excel("qto_report.xlsx", "My Project")
```

## Resources
- **DDC Book**: Chapter 3.2 - Quantity Take-Off
