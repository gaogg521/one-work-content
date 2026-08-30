---
name: sheet-cog
description: CellCog 由其自己的 Coding Agent 构建。同一个代理构建你的电子表格。用于复杂数据操作、公式、数据透视表、财务模型、预算模板、数据跟踪器和 Excel/XLSX 生成的完整 Python 访问权限 —— 由每天开发整个 AI 平台的工程大脑提供支持。
metadata: None
openclaw: None
emoji: 📈
author: CellCog
dependencies:
- cellcog
tags:
- AI
- Python
---

# Sheet Cog - 由构建 CellCog 的代理构建

**CellCog 由其自己的 Coding Agent 构建。同一个代理构建你的电子表格。**

完整的 Python 访问权限、复杂的数据操作、公式、数据透视表和财务模型 —— 由每天开发整个 AI 平台的工程大脑提供支持。不是模板填充器。一个理解你的数据并准确构建你所需内容的程序员。

---

## 先决条件

此技能需要 `cellcog` 技能进行 SDK 设置和 API 调用。

```bash
clawhub install cellcog
```

**首先阅读 cellcog 技能**以进行 SDK 设置。此技能向你展示可能性。

**快速模式 (v1.0+)：**
```python
# 即发即弃 - 立即返回
result = client.create_chat(
    prompt="[your spreadsheet request]",
    notify_session_key="agent:main:main",
    task_label="spreadsheet-task",
    chat_mode="agent"  # Agent 模式处理大多数电子表格效果很好
)
# Daemon 在完成时通知你 - 不要轮询
```

---

## 你可以创建什么电子表格

### 财务模型

专业的财务分析和预测：

- **Startup Financial Model**: "Create a 3-year financial model for a SaaS startup including revenue projections, expenses, and cash flow"
- **DCF Model**: "Build a discounted cash flow model for valuing a company"
- **Investment Analysis**: "Create a real estate investment analysis spreadsheet with ROI calculations"
- **Revenue Model**: "Build a revenue forecasting model with multiple scenarios (base, optimistic, pessimistic)"
- **Unit Economics**: "Create a unit economics spreadsheet showing CAC, LTV, payback period"

### 预算模板

个人和企业预算：

- **Personal Budget**: "Create a monthly personal budget tracker with income, fixed expenses, variable expenses, and savings goals"
- **Household Budget**: "Build a family budget spreadsheet with categories for housing, food, transportation, etc."
- **Project Budget**: "Create a project budget template with phases, resources, and variance tracking"
- **Marketing Budget**: "Build a marketing budget spreadsheet with channels, planned vs actual, and ROI tracking"
- **Event Budget**: "Create a wedding budget spreadsheet with vendor categories and payment tracking"

### 数据跟踪器

任何数据的有组织跟踪：

- **Fitness Tracker**: "Create a workout log spreadsheet with exercises, sets, reps, weights, and progress charts"
- **Habit Tracker**: "Build a daily habit tracking spreadsheet with monthly overview"
- **Inventory Tracker**: "Create an inventory management spreadsheet with stock levels, reorder points, and valuation"
- **Sales Tracker**: "Build a sales pipeline tracker with stages, probabilities, and forecasting"
- **Time Tracker**: "Create a timesheet template with projects, hours, and billing calculations"

### 业务工具

运营电子表格：

- **Invoice Template**: "Create a professional invoice template with automatic calculations"
- **Employee Directory**: "Build an employee directory spreadsheet with contact info, departments, and start dates"
- **Vendor Comparison**: "Create a vendor comparison spreadsheet for evaluating suppliers"
- **OKR Tracker**: "Build an OKR tracking spreadsheet for quarterly goals"
- **Meeting Agenda**: "Create a meeting agenda template with action items tracking"

### 分析模板

数据分析和计算：

- **Break-Even Analysis**: "Create a break-even analysis spreadsheet with charts"
- **Scenario Analysis**: "Build a scenario planning spreadsheet with what-if analysis"
- **Pricing Calculator**: "Create a pricing model spreadsheet with cost-plus and value-based options"
- **Loan Calculator**: "Build a loan amortization schedule with payment breakdown"
- **Commission Calculator**: "Create a sales commission calculator with tiered rates"

---

## 电子表格功能

CellCog 电子表格可以包括：

| Feature | Description |
|---------|-------------|
| **Formulas** | SUM、AVERAGE、IF、VLOOKUP 和复杂计算 |
| **Formatting** | 标题、颜色、边框、数字格式、条件格式 |
| **Charts** | 嵌入工作表中的条形图、折线图、饼图 |
| **Multiple Sheets** | 带有链接工作表的有组织工作簿 |
| **Data Validation** | 下拉菜单、输入限制 |
| **Named Ranges** | 用于更清晰的公式 |
| **Print Layout** | 准备好打印/PDF |

---

## 输出格式

| Format | Best For |
|--------|----------|
| **XLSX** | 可在 Excel、Google Sheets、Numbers 中编辑 |
| **Interactive HTML** | 基于 Web 的计算器和工具 |

---

## 用于电子表格的聊天模式

| Scenario | Recommended Mode |
|----------|------------------|
| 预算模板、跟踪器、数据表、基本计算 | `"agent"` |
| 具有多场景分析、复杂公式的复杂财务模型 | `"agent team"` |

**大多数电子表格请求默认使用 `"agent"`。** CellCog 的代理模式高效地处理公式、格式、图表和数据组织。

为需要深度准确性验证的复杂财务建模保留 `"agent team"` —— 如 DCF 模型、多场景预测或公式正确性至关重要的互连工作簿。

---

## 示例电子表格提示

**SaaS 财务模型：**
> "Create a 3-year SaaS financial model with:
> 
> **Assumptions Sheet:**
> - Starting MRR: $10,000
> - Monthly growth rate: 15%
> - Churn rate: 3%
> - Average revenue per customer: $99
> - CAC: $500
> - Gross margin: 80%
> 
> **Monthly P&L:** Revenue, COGS, Gross Profit, Operating Expenses (broken down), Net Income
> 
> **Key Metrics:** MRR, ARR, Customers, Churn, LTV, CAC, LTV:CAC ratio
> 
> **Charts:** MRR growth, customer growth, profitability timeline
> 
> Include scenario toggles for growth rate (10%, 15%, 20%)."

**个人预算：**
> "Create a monthly personal budget spreadsheet:
> 
> **Income Section:** Salary, side income, other
> 
> **Fixed Expenses:** Rent, utilities, insurance, subscriptions, loan payments
> 
> **Variable Expenses:** Groceries, dining out, transportation, entertainment, shopping, health
> 
> **Savings:** Emergency fund, retirement, vacation fund
> 
> Include:
> - Monthly summary with % of income per category
> - Year-at-a-glance sheet with monthly totals
> - Pie chart showing expense breakdown
> - Conditional formatting (red if over budget)
> 
> Assume $5,000/month income."

**销售跟踪器：**
> "Build a sales pipeline tracker spreadsheet with:
> 
> **Columns:** Company, Contact, Deal Value, Stage (dropdown: Lead, Qualified, Proposal, Negotiation, Closed Won, Closed Lost), Probability, Expected Close Date, Notes, Last Contact
> 
> **Calculations:** Weighted pipeline value, deals by stage, win rate
> 
> **Dashboard Sheet:** Pipeline by stage (funnel chart), monthly forecast, top 10 deals, activity metrics
> 
> Include sample data for 20 deals."

**盈亏平衡分析：**
> "Create a break-even analysis spreadsheet:
> 
> **Inputs:**
> - Fixed costs (rent, salaries, etc.)
> - Variable cost per unit
> - Selling price per unit
> 
> **Calculations:**
> - Break-even units
> - Break-even revenue
> - Margin of safety
> 
> **Sensitivity table:** Show break-even at different price points
> 
> **Chart:** Cost-volume-profit graph showing break-even point
> 
> Default values: Fixed costs $50,000/month, variable cost $15/unit, price $25/unit."

---

## 更好电子表格的提示

1. **指定结构**：列出你需要的表、列和计算。

2. **提供假设**：对于财务模型，给出起始数字和增长率。

3. **提及所需公式**："Include VLOOKUP for...", "Calculate running totals", "Show variance vs plan."

4. **请求示例数据**："Include realistic sample data for testing" 有助于查看它的实际效果。

5. **描述格式**："Conditional formatting for negative values", "Currency format", "Freeze header row."

6. **图表偏好**："Include a line chart showing trend", "Pie chart for breakdown."
