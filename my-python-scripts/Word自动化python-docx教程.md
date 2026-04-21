# 📝 Word 自动化 python-docx 完整教程

> 用 Python 操作 Word 文档的全面指南

## 📂 目录

1. [安装与基础](#安装与基础)
2. [创建文档](#创建文档)
3. [文本操作](#文本操作)
4. [表格操作](#表格操作)
5. [格式设置](#格式设置)
6. [实用案例](#实用案例)

---

## 📦 安装与基础

### 安装库

```bash
pip install python-docx
```

### 导入

```python
from docx import Document
from docx.shared import Inches, Pt, RGBColor
from docx.enum.text import WD_PARAGRAPH_ALIGNMENT
```

---

## 📄 创建文档

### 1. 创建新文档

```python
doc = Document()
doc.save('demo.docx')
```

### 2. 打开现有文档

```python
doc = Document('existing.docx')
# 编辑后保存
doc.save('modified.docx')
```

### 3. 添加标题

```python
# 0-4 级标题
doc.add_heading('一级标题', level=0)
doc.add_heading('二级标题', level=1)
doc.add_heading('三级标题', level=2)
```

### 4. 添加段落

```python
# 普通段落
doc.add_paragraph('这是普通段落')

# 带样式的段落
p = doc.add_paragraph()
run = p.add_run('部分加粗')
run.bold = True
p.add_run('，部分正常')

# 添加分隔线
doc.add_paragraph('─' * 50)
```

### 5. 添加分页

```python
doc.add_page_break()
```

---

## ✍️ 文本操作

### 1. 添加文本

```python
paragraph = doc.add_paragraph()
paragraph.add_run('这是文本')
```

### 2. 设置字体样式

```python
from docx.shared import Font

# 设置整个段落字体
for run in paragraph.runs:
    run.font.name = '微软雅黑'
    run.font.size = Pt(12)
    run.font.bold = True
    run.font.italic = True
    run.font.color.rgb = RGBColor(255, 0, 0)  # 红色
```

### 3. 对齐方式

```python
from docx.enum.text import WD_PARAGRAPH_ALIGNMENT

p.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER      # 居中
p.alignment = WD_PARAGRAPH_ALIGNMENT.LEFT        # 左对齐
p.alignment = WD_PARAGRAPH_ALIGNMENT.RIGHT       # 右对齐
p.alignment = WD_PARAGRAPH_ALIGNMENT.JUSTIFY      # 两端对齐
```

### 4. 行间距

```python
from docx.shared import Pt
from docx.oxml.ns import qn

paragraph.paragraph_format.line_spacing = Pt(18)  # 固定行距
paragraph.paragraph_format.space_before = Pt(12)   # 段前间距
paragraph.paragraph_format.space_after = Pt(12)   # 段后间距
```

---

## 📊 表格操作

### 1. 创建表格

```python
# 3行4列的表格
table = doc.add_table(rows=3, cols=4, style='Table Grid')
```

### 2. 填充表格数据

```python
# 方式1：直接赋值
table.rows[0].cells[0].text = '姓名'
table.rows[0].cells[1].text = '年龄'

# 方式2：循环填充
data = [
    ['姓名', '年龄', '城市'],
    ['张三', '25', '北京'],
    ['李四', '30', '上海'],
    ['王五', '28', '广州']
]

for i, row_data in enumerate(data):
    row = table.rows[i]
    for j, cell_text in enumerate(row_data):
        row.cells[j].text = cell_text
```

### 3. 表格样式

```python
# 预设样式
table.style = 'Light Grid Accent 1'
table.style = 'Table Grid'
table.style = 'Colorful List'

# 设置表格边框
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

def set_cell_border(cell, **kwargs):
    """设置单元格边框"""
    tc = cell._tc
    tcPr = tc.get_or_add_tcPr()
    tcBorders = OxmlElement('w:tcBorders')
    for edge in ['top', 'left', 'bottom', 'right']:
        if edge in kwargs:
            element = OxmlElement(f'w:{edge}')
            element.set(qn('w:val'), kwargs[edge]['val'])
            element.set(qn('w:sz'), kwargs[edge]['sz'])
            tcBorders.append(element)
    tcPr.append(tcBorders)
```

### 4. 合并单元格

```python
# 合并行
cell = table.cell(0, 0)
cell.merge(table.cell(0, 1))  # 合并第一行前两列

# 合并列
cell.merge(table.cell(0, 2))  # 合并第一行前三列
```

### 5. 表格自动调整

```python
from docx.shared import Inches

# 设置列宽
for row in table.rows:
    row.cells[0].width = Inches(2)
    row.cells[1].width = Inches(1.5)
    row.cells[2].width = Inches(2)
```

---

## 🎨 格式设置

### 1. 设置图片

```python
from docx.shared import Inches

# 添加图片
doc.add_picture('image.jpg', width=Inches(4))

# 设置图片大小
doc.add_picture('image.jpg', width=Inches(3), height=Inches(2))
```

### 2. 设置项目符号列表

```python
# 添加项目符号
doc.add_paragraph('第一项', style='List Bullet')
doc.add_paragraph('第二项', style='List Bullet')
doc.add_paragraph('第三项', style='List Bullet')

# 添加编号
doc.add_paragraph('第一项', style='List Number')
doc.add_paragraph('第二项', style='List Number')
```

### 3. 设置超链接

```python
from docx.oxml.ns import qn

def add_hyperlink(paragraph, url, text):
    """添加超链接"""
    part = paragraph.part
    r_id = part.relate_to(url, 'http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink', is_external=True)
    
    hyperlink = OxmlElement('w:hyperlink')
    hyperlink.set(qn('r:id'), r_id)
    
    new_run = OxmlElement('w:r')
    rPr = OxmlElement('w:rPr')
    
    c = OxmlElement('w:color')
    c.set(qn('w:val'), '0000FF')
    rPr.append(c)
    
    u = OxmlElement('w:u')
    u.set(qn('w:val'), 'single')
    rPr.append(u)
    
    new_run.append(rPr)
    new_run.text = text
    hyperlink.append(new_run)
    
    paragraph._p.append(hyperlink)
    return hyperlink
```

---

## 🎯 实用案例

### 案例 1: 生成员工信息表

```python
from docx import Document
from docx.shared import Inches, Pt, RGBColor
from docx.enum.text import WD_PARAGRAPH_ALIGNMENT

def create_employee_report(employees, output_file):
    """生成员工信息报告"""
    doc = Document()
    
    # 添加标题
    title = doc.add_heading('员工信息表', level=0)
    title.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    
    # 添加时间
    from datetime import datetime
    doc.add_paragraph(f'生成时间: {datetime.now().strftime("%Y-%m-%d %H:%M:%S")}')
    
    # 创建表格
    table = doc.add_table(rows=len(employees)+1, cols=4, style='Table Grid')
    table.autofit = True
    
    # 表头
    headers = ['工号', '姓名', '部门', '职位']
    for i, header in enumerate(headers):
        cell = table.rows[0].cells[i]
        cell.text = header
        # 设置表头样式
        for paragraph in cell.paragraphs:
            paragraph.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    
    # 填充数据
    for row_idx, emp in enumerate(employees, start=1):
        table.rows[row_idx].cells[0].text = emp['id']
        table.rows[row_idx].cells[1].text = emp['name']
        table.rows[row_idx].cells[2].text = emp['department']
        table.rows[row_idx].cells[3].text = emp['position']
    
    doc.save(output_file)
    print(f'报告已生成: {output_file}')

# 使用
employees = [
    {'id': '001', 'name': '张三