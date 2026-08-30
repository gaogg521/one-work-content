---
name: csv-pipeline
description: 处理、转换、分析和报告 CSV 与 JSON 数据文件。当用户需要过滤行、连接数据集、计算聚合、转换格式、去重或从表格数据生成汇总报告时使用。支持任何 CSV、TSV 或 JSON Lines 文件。
metadata: None
clawdbot: None
emoji: 📊
os:
- linux
- darwin
- win32
requires: None
anyBins:
- python3
- python
- uv
tags:
- CI/CD
---

# CSV 数据处理流水线

使用标准命令行工具和 Python 处理表格数据（CSV、TSV、JSON、JSON Lines）。除 Python 3 外，无需额外依赖。

## 何时使用

- 用户提供了 CSV/TSV/JSON 文件并要求分析、转换或生成报告
- 需要对表格数据进行连接、过滤、分组或聚合
- 格式转换（CSV 转 JSON、JSON 转 CSV 等）
- 去重、排序或清理脏数据
- 生成汇总统计或报告
- ETL 工作流：从一种格式提取、转换、加载到另一种格式

## 使用标准工具快速操作

### 查看

```bash
# 预览前 5 行
head -5 data.csv

# 统计行数（不含表头）
tail -n +2 data.csv | wc -l

# 显示列名
head -1 data.csv

# 统计某列的唯一值数量（第 3 列）
tail -n +2 data.csv | cut -d',' -f3 | sort -u | wc -l
```

### 使用 `awk` 过滤

```bash
# 过滤第 3 列大于 100 的行
awk -F',' 'NR==1 || $3 > 100' data.csv > filtered.csv

# 过滤第 2 列匹配模式的行
awk -F',' 'NR==1 || $2 ~ /pattern/' data.csv > matched.csv

# 对第 4 列求和
awk -F',' 'NR>1 {sum += $4} END {print sum}' data.csv
```

### 排序与去重

```bash
# 按第 2 列（数值）排序
head -1 data.csv > sorted.csv && tail -n +2 data.csv | sort -t',' -k2 -n >> sorted.csv

# 按所有列去重
head -1 data.csv > deduped.csv && tail -n +2 data.csv | sort -u >> deduped.csv

# 按指定列去重（保留首次出现）
awk -F',' '!seen[$2]++' data.csv > deduped.csv
```

## Python 操作（用于复杂转换）

### 读取与查看

```python
import csv, json, sys
from collections import Counter

def read_csv(path, delimiter=','):
    """Read CSV/TSV into list of dicts."""
    with open(path, newline='', encoding='utf-8') as f:
        return list(csv.DictReader(f, delimiter=delimiter))

def write_csv(rows, path, delimiter=','):
    """Write list of dicts to CSV."""
    if not rows:
        return
    with open(path, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=rows[0].keys(), delimiter=delimiter)
        writer.writeheader()
        writer.writerows(rows)

# Quick stats
data = read_csv('data.csv')
print(f"Rows: {len(data)}")
print(f"Columns: {list(data[0].keys())}")
for col in data[0]:
    non_empty = sum(1 for r in data if r[col].strip())
    print(f"  {col}: {non_empty}/{len(data)} non-empty")
```

### 过滤与转换

```python
# Filter rows
filtered = [r for r in data if float(r['amount']) > 100]

# Add computed column
for r in data:
    r['total'] = str(float(r['price']) * int(r['quantity']))

# Rename columns
renamed = [{('new_name' if k == 'old_name' else k): v for k, v in r.items()} for r in data]

# Type conversion
for r in data:
    r['amount'] = float(r['amount'])
    r['date'] = r['date'].strip()
```

### 分组与聚合

```python
from collections import defaultdict

def group_by(rows, key):
    """Group rows by a column value."""
    groups = defaultdict(list)
    for r in rows:
        groups[r[key]].append(r)
    return dict(groups)

def aggregate(rows, group_col, agg_col, func='sum'):
    """Aggregate a column by groups."""
    groups = group_by(rows, group_col)
    results = []
    for name, group in sorted(groups.items()):
        values = [float(r[agg_col]) for r in group if r[agg_col].strip()]
        if func == 'sum':
            agg = sum(values)
        elif func == 'avg':
            agg = sum(values) / len(values) if values else 0
        elif func == 'count':
            agg = len(values)
        elif func == 'min':
            agg = min(values) if values else 0
        elif func == 'max':
            agg = max(values) if values else 0
        results.append({group_col: name, f'{func}_{agg_col}': str(agg), 'count': str(len(group))})
    return results

# Example: sum revenue by category
summary = aggregate(data, 'category', 'revenue', 'sum')
write_csv(summary, 'summary.csv')
```

### 连接数据集

```python
def inner_join(left, right, on):
    """Inner join two datasets on a key column."""
    right_index = {}
    for r in right:
        key = r[on]
        if key not in right_index:
            right_index[key] = []
        right_index[key].append(r)

    results = []
    for lr in left:
        key = lr[on]
        if key in right_index:
            for rr in right_index[key]:
                merged = {**lr}
                for k, v in rr.items():
                    if k != on:
                        merged[k] = v
                results.append(merged)
    return results

def left_join(left, right, on):
    """Left join: keep all left rows, fill missing right with empty."""
    right_index = {}
    right_cols = set()
    for r in right:
        key = r[on]
        right_cols.update(r.keys())
        if key not in right_index:
            right_index[key] = []
        right_index[key].append(r)
    right_cols.discard(on)

    results = []
    for lr in left:
        key = lr[on]
        if key in right_index:
            for rr in right_index[key]:
                merged = {**lr}
                for k, v in rr.items():
                    if k != on:
                        merged[k] = v
                results.append(merged)
        else:
            merged = {**lr}
            for col in right_cols:
                merged[col] = ''
            results.append(merged)
    return results

# Example
orders = read_csv('orders.csv')
customers = read_csv('customers.csv')
joined = left_join(orders, customers, on='customer_id')
write_csv(joined, 'orders_with_customers.csv')
```

### 去重

```python
def deduplicate(rows, key_cols=None):
    """Remove duplicate rows. If key_cols specified, dedupe by those columns only."""
    seen = set()
    unique = []
    for r in rows:
        if key_cols:
            key = tuple(r[c] for c in key_cols)
        else:
            key = tuple(sorted(r.items()))
        if key not in seen:
            seen.add(key)
            unique.append(r)
    return unique

# Deduplicate by email column
clean = deduplicate(data, key_cols=['email'])
```

## 格式转换

### CSV 转 JSON

```python
import json, csv

with open('data.csv', newline='', encoding='utf-8') as f:
    rows = list(csv.DictReader(f))

# Array of objects
with open('data.json', 'w') as f:
    json.dump(rows, f, indent=2)

# JSON Lines (one object per line, streamable)
with open('data.jsonl', 'w') as f:
    for row in rows:
        f.write(json.dumps(row) + '\n')
```

### JSON 转 CSV

```python
import json, csv

with open('data.json') as f:
    rows = json.load(f)

with open('data.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.DictWriter(f, fieldnames=rows[0].keys())
    writer.writeheader()
    writer.writerows(rows)
```

### JSON Lines 转 CSV

```python
import json, csv

rows = []
with open('data.jsonl') as f:
    for line in f:
        if line.strip():
            rows.append(json.loads(line))

with open('data.csv', 'w', newline='', encoding='utf-8') as f:
    all_keys = set()
    for r in rows:
        all_keys.update(r.keys())
    writer = csv.DictWriter(f, fieldnames=sorted(all_keys))
    writer.writeheader()
    writer.writerows(rows)
```

### TSV 转 CSV

```bash
tr '\t' ',' < data.tsv > data.csv
```

## 数据清理模式

### 修复常见 CSV 问题

```python
def clean_csv(rows):
    """Clean common CSV data quality issues."""
    cleaned = []
    for r in rows:
        clean_row = {}
        for k, v in r.items():
            # Strip whitespace from keys and values
            k = k.strip()
            v = v.strip() if isinstance(v, str) else v
            # Normalize empty values
            if v in ('', 'N/A', 'n/a', 'NA', 'null', 'NULL', 'None', '-'):
                v = ''
            # Normalize boolean values
            if v.lower() in ('true', 'yes', '1', 'y'):
                v = 'true'
            elif v.lower() in ('false', 'no', '0', 'n'):
                v = 'false'
            clean_row[k] = v
        cleaned.append(clean_row)
    return cleaned
```

### 验证数据类型

```python
def validate_rows(rows, schema):
    """
    Validate rows against a schema.
    schema: dict of column_name -> 'int'|'float'|'date'|'email'|'str'
    Returns (valid_rows, error_rows)
    """
    import re
    valid, errors = [], []
    for i, r in enumerate(rows):
        errs = []
        for col, dtype in schema.items():
            val = r.get(col, '').strip()
            if not val:
                continue
            if dtype == 'int':
                try:
                    int(val)
                except ValueError:
                    errs.append(f"{col}: '{val}' not int")
            elif dtype == 'float':
                try:
                    float(val)
                except ValueError:
                    errs.append(f"{col}: '{val}' not float")
            elif dtype == 'email':
                if not re.match(r'^[^@]+@[^@]+\.[^@]+$', val):
                    errs.append(f"{col}: '{val}' not email")
            elif dtype == 'date':
                if not re.match(r'^\d{4}-\d{2}-\d{2}', val):
                    errs.append(f"{col}: '{val}' not YYYY-MM-DD")
        if errs:
            errors.append({'row': i + 2, 'errors': errs, 'data': r})
        else:
            valid.append(r)
    return valid, errors

# Usage
valid, bad = validate_rows(data, {'amount': 'float', 'email': 'email', 'date': 'date'})
print(f"Valid: {len(valid)}, Errors: {len(bad)}")
for e in bad[:5]:
    print(f"  Row {e['row']}: {e['errors']}")
```

## 生成报告

### Markdown 汇总报告

```python
def generate_report(data, title, group_col, value_col):
    """Generate a Markdown summary report."""
    lines = [f"# {title}", f"", f"**Total rows**: {len(data)}", ""]

    # Group summary
    groups = group_by(data, group_col)
    lines.append(f"## By {group_col}")
    lines.append("")
    lines.append(f"| {group_col} | Count | Sum | Avg | Min | Max |")
    lines.append("|---|---|---|---|---|---|")

    for name in sorted(groups):
        vals = [float(r[value_col]) for r in groups[name] if r[value_col].strip()]
        if vals:
            lines.append(f"| {name} | {len(vals)} | {sum(vals):.2f} | {sum(vals)/len(vals):.2f} | {min(vals):.2f} | {max(vals):.2f} |")

    lines.append("")
    lines.append(f"*Generated from {len(data)} rows*")
    return '\n'.join(lines)

report = generate_report(data, "Sales Summary", "category", "revenue")
with open('report.md', 'w') as f:
    f.write(report)
```

## 大文件处理

对于无法一次性加载到内存的文件：

```python
def stream_process(input_path, output_path, transform_fn, delimiter=','):
    """Process a CSV row-by-row without loading entire file."""
    with open(input_path, newline='', encoding='utf-8') as fin, \
         open(output_path, 'w', newline='', encoding='utf-8') as fout:
        reader = csv.DictReader(fin, delimiter=delimiter)
        writer = None
        for row in reader:
            result = transform_fn(row)
            if result is None:
                continue  # Skip row
            if writer is None:
                writer = csv.DictWriter(fout, fieldnames=result.keys(), delimiter=delimiter)
                writer.writeheader()
            writer.writerow(result)

# Example: filter and transform in streaming fashion
def process_row(row):
    if float(row.get('amount', 0) or 0) < 10:
        return None  # Skip small amounts
    row['amount_usd'] = str(float(row['amount']) * 1.0)  # Add computed field
    return row

stream_process('big_file.csv', 'output.csv', process_row)
```

## 提示

- 始终检查编码：`file -i data.csv` 或使用 `encoding='utf-8-sig'` 打开带 BOM 的文件
- 对于包含逗号的 Excel 导出，CSV 模块会自动处理引号
- 使用 `json.dumps(ensure_ascii=False)` 处理国际字符
- 管道分隔文件：在 csv.reader/writer 中使用 `delimiter='|'`
- 对于非常大的聚合，考虑使用 Python 内置的 `sqlite3`：
  ```bash
  sqlite3 :memory: ".mode csv" ".import data.csv t" "SELECT category, SUM(amount) FROM t GROUP BY category;"
  ```
