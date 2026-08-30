---
name: invoice-template
description: 本技能从结构化数据和模板生成专业的 PDF 发票。创建带有公司品牌、itemized lists、tax calculations 和 payment details 的发票。
---

# 发票模板技能

## 概述

本技能从结构化数据和模板生成专业的 PDF 发票。创建带有公司品牌、itemized lists、tax calculations 和 payment details 的发票。

## 如何使用

1. 描述你想要完成的内容
2. 提供任何必需的输入数据或文件
3. 我将执行适当的操作

**示例提示：**
- "Generate invoices from order data"
- "Create recurring invoices"
- "Batch generate monthly invoices"
- "Customize invoice templates per client"

## 领域知识


### 发票数据结构

```python
invoice_data = {
    "invoice_number": "INV-2026-001",
    "date": "2026-01-30",
    "due_date": "2026-02-28",
    
    "from": {
        "name": "Your Company",
        "address": "123 Business St",
        "email": "billing@company.com"
    },
    
    "to": {
        "name": "Client Name",
        "address": "456 Client Ave",
        "email": "client@example.com"
    },
    
    "items": [
        {"description": "Consulting", "quantity": 10, "rate": 150.00},
        {"description": "Development", "quantity": 20, "rate": 100.00}
    ],
    
    "tax_rate": 0.08,
    "notes": "Payment due within 30 days"
}
```

### 使用 ReportLab 生成 PDF

```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
from reportlab.lib.units import inch

def create_invoice(data: dict, output_path: str):
    c = canvas.Canvas(output_path, pagesize=letter)
    width, height = letter
    
    # Header
    c.setFont("Helvetica-Bold", 24)
    c.drawString(1*inch, height - 1*inch, "INVOICE")
    
    # Invoice details
    c.setFont("Helvetica", 12)
    c.drawString(1*inch, height - 1.5*inch, f"Invoice #: {data['invoice_number']}")
    c.drawString(1*inch, height - 1.75*inch, f"Date: {data['date']}")
    
    # From/To
    y = height - 2.5*inch
    c.drawString(1*inch, y, f"From: {data['from']['name']}")
    c.drawString(4*inch, y, f"To: {data['to']['name']}")
    
    # Items table
    y = height - 4*inch
    c.setFont("Helvetica-Bold", 10)
    c.drawString(1*inch, y, "Description")
    c.drawString(4*inch, y, "Qty")
    c.drawString(5*inch, y, "Rate")
    c.drawString(6*inch, y, "Amount")
    
    c.setFont("Helvetica", 10)
    subtotal = 0
    for item in data['items']:
        y -= 0.3*inch
        amount = item['quantity'] * item['rate']
        subtotal += amount
        c.drawString(1*inch, y, item['description'])
        c.drawString(4*inch, y, str(item['quantity']))
        c.drawString(5*inch, y, f"${item['rate']:.2f}")
        c.drawString(6*inch, y, f"${amount:.2f}")
    
    # Totals
    tax = subtotal * data['tax_rate']
    total = subtotal + tax
    
    y -= 0.5*inch
    c.drawString(5*inch, y, f"Subtotal: ${subtotal:.2f}")
    y -= 0.25*inch
    c.drawString(5*inch, y, f"Tax ({data['tax_rate']*100}%): ${tax:.2f}")
    y -= 0.25*inch
    c.setFont("Helvetica-Bold", 12)
    c.drawString(5*inch, y, f"Total: ${total:.2f}")
    
    c.save()
    return output_path
```

### HTML 模板方案

```python
from weasyprint import HTML
from jinja2 import Template

invoice_template = """
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: Arial; margin: 40px; }
        .header { display: flex; justify-content: space-between; }
        table { width: 100%; border-collapse: collapse; margin: 20px 0; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: left; }
        .total { font-weight: bold; font-size: 18px; }
    </style>
</head>
<body>
    <div class="header">
        <h1>INVOICE</h1>
        <div>
            <p>Invoice #: {{ invoice_number }}</p>
            <p>Date: {{ date }}</p>
        </div>
    </div>
    <table>
        <tr><th>Description</th><th>Qty</th><th>Rate</th><th>Amount</th></tr>
        {% for item in items %}
        <tr>
            <td>{{ item.description }}</td>
            <td>{{ item.quantity }}</td>
            <td>${{ "%.2f"|format(item.rate) }}</td>
            <td>${{ "%.2f"|format(item.quantity * item.rate) }}</td>
        </tr>
        {% endfor %}
    </table>
    <p class="total">Total: ${{ "%.2f"|format(total) }}</p>
</body>
</html>
"""

def create_invoice_html(data: dict, output_path: str):
    template = Template(invoice_template)
    
    # Calculate total
    total = sum(i['quantity'] * i['rate'] for i in data['items'])
    total *= (1 + data.get('tax_rate', 0))
    data['total'] = total
    
    html = template.render(**data)
    HTML(string=html).write_pdf(output_path)
    return output_path
```


## 最佳实践

1. **生成前验证必填字段**
2. **使用模板保持品牌一致性**
3. **自动计算 totals（不要信任输入）**
4. **包含 payment instructions 和 terms**

## 安装

```bash
# 安装必需的依赖
pip install python-docx openpyxl python-pptx reportlab jinja2
```

## 资源

- [easy-invoice-pdf Repository](https://github.com/nickmitchko/easy-invoice-pdf)
- [Claude Office Skills Hub](https://github.com/claude-office-skills/skills)
