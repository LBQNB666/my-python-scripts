# ⚠️ Word 自动化避坑指南

> 真正踩过坑才能总结出的经验

## 🔥 常见问题与解决方案

---

## 1. 中文乱码问题

### 问题描述
Word 文档中的中文显示为乱码，或字体无法正确渲染。

### 解决方案

```python
from docx import Document
from docx.oxml.ns import qn

def set_chinese_font(run, font_name='微软雅黑', font_size=12):
    """正确设置中文字体"""
    run.font.name = font_name
    run._element.rPr.rFonts.set(qn('w:eastAsia'), font_name)  # 必须设置东亚字体
    run.font.size = Pt(font_size)

# 使用
p = doc.add_paragraph()
run = p.add_run('中文内容')
set_chinese_font(run)
```

### 关键点
- **必须设置 `w:eastAsia`** 才会对中文生效
- 字体名要正确：`微软雅黑`、`宋体`、`黑体`
- Linux 系统可能需要安装对应字体

---

## 2. 表格跨页断开

### 问题描述
Word 表格在跨页时会被断开，影响阅读。

### 解决方案

```python
from docx import Document
from docx.oxml import OxmlElement

def set_row_no_split(table_row):
    """防止表格行跨页断开"""
    row_element = table_row._element
    trPr = row_element.get_or_add_trPr()
    
    # 添加 <w:cantSplit/> 元素
    cantSplit = OxmlElement('w:cantSplit')
    cantSplit.set(qn('w:val'), '1')
    trPr.append(cantSplit)

def set_table_no_split(table):
    """防止整个表格跨页断开"""
    tbl = table._element
    tblPr = tbl.tblPr
    if tblPr is None:
        tblPr = OxmlElement('w:tblPr')
        tbl.insert(0, tblPr)
    
    tblCellMar = OxmlElement('w:tblCellMar')
    # 设置单元格边距
    tblPr.append(tblCellMar)

# 使用
table = doc.add_table(rows=10, cols=4)
for row in table.rows:
    set_row_no_split(row)
```

### 另一种方法：段落格式

```python
def set_paragraph_keep_with_next(paragraph):
    """保持段落与下一段同页"""
    pPr = paragraph._element.get_or_add_pPr()
    keepNext = OxmlElement('w:keepNext')
    keepNext.set(qn('w:val'), '1')
    pPr.append(keepNext)

def set_paragraph_keep_lines(paragraph):
    """保持段落不分开"""
    pPr = paragraph._element.get_or_add_pPr()
    keepLines = OxmlElement('w:keepLines')
    keepLines.set(qn('w:val'), '1')
    pPr.append(keepLines)
```

---

## 3. 合并单元格报错

### 问题描述
合并单元格时出现 `InvalidSpanError` 或位置错乱。

### 解决方案

```python
def merge_cells_safe(table, start_row, start_col, end_row, end_col):
    """安全合并单元格"""
    # 必须是矩形区域
    cell_start = table.cell(start_row, start_col)
    cell_end = table.cell(end_row, end_col)
    
    try:
        merged_cell = cell_start.merge(cell_end)
        return merged_cell
    except Exception as e:
        print(f'合并失败: {e}')
        return None

# 合并整行
cells = table.rows[0].cells
cells[0].merge(cells[-1])  # 合并第一行所有单元格

# 合并整列
for row in table.rows:
    row.cells[0].merge(row.cells[-1])
```

### 合并后访问单元格

```python
# 合并后的单元格只能通过左上角访问
merged = table.cell(0, 0)
print(merged.text)  # 获取合并单元格内容

# 设置内容
merged.text = '合并后的内容'
```

---

## 4. 样式继承问题

### 问题描述
修改样式时影响了其他段落，或样式不生效。

### 解决方案

```python
def apply_style_to_paragraph(paragraph, style_name=None, **kwargs):
    """独立应用样式，不影响其他段落"""
    # 如果指定了样式名称
    if style_name:
        paragraph.style = doc.styles[style_name]
    
    # 或者手动设置每个 run
    for run in paragraph.runs:
        if 'bold' in kwargs:
            run.font.bold = kwargs['bold']
        if 'size' in kwargs:
            run.font.size = Pt(kwargs['size'])
        if 'color' in kwargs:
            run.font.color.rgb = RGBColor(*kwargs['color'])

# 不要这样做 ❌
# for para in doc.paragraphs:
#     para.runs[0].font.bold = True  # 可能影响全局

# 应该这样做 ✅
# 新建段落时创建独立的 run
new_para = doc.add_paragraph()
new_run = new_para.add_run('新内容')
new_run.font.bold = True
```

---

## 5. 读取时丢失格式

### 问题描述
读取现有 Word 文档时，部分格式丢失。

### 解决方案

```python
def read_with_formatting(file_path):
    """尽量保留格式读取 Word"""
    doc = Document(file_path)
    
    for para in doc.paragraphs:
        # 获取样式名称
        style_name = para.style.name
        
        # 获取所有 run 的格式
        for run in para.runs:
            font_name = run.font.name
            font_size = run.font.size
            font_bold = run.font.bold
            font_color = run.font.color.rgb if run.font.color else None
    
    return doc

def copy_element_with_format(source_element, target_doc):
    """复制元素时保留格式"""
    # 导入需要的模块
    from docx.oxml import OxmlElement
    from copy import deepcopy
    
    # 深拷贝元素
    new_element = deepcopy(source_element)
    target_doc.element.body.append(new_element)
```

---

## 6. 页眉页脚设置

### 问题描述
添加页眉页脚后格式错乱，或不显示。

### 解决方案

```python
from docx.oxml.ns import qn

def add_header_with_format(doc, header_text, page_num=True):
    """添加格式正确的页眉"""
    section = doc.sections[0]
    header = section.header
    header.is_linked_to_previous = False
    
    # 清除默认内容
    for para in header.paragraphs:
        para.clear()
    
    # 添加页眉内容
    if header.paragraphs:
        para = header.paragraphs[0]
    else:
        para = header.add_paragraph()
    
    para.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    
    # 添加文本
    run = para.add_run(header_text)
    run.font.size = Pt(10)
    run.font.name = '微软雅黑'
    run._element.rPr.rFonts.set(qn('w:eastAsia'), '微软雅黑')
    
    # 添加页码
    if page_num:
        run = para.add_run('  第 ')
        run.font.size = Pt(10)
        
        # 添加页码域
        fldChar1 = OxmlElement('w:fldChar')
        fldChar1.set(qn('w:fldCharType'), 'begin')
        
        instrText = OxmlElement('w:instrText')
        instrText.text = 'PAGE'
        
        fldChar2 = OxmlElement('w:fldChar')
        fldChar2.set(qn('w:fldCharType'), 'end')
        
        run._r.append(fldChar1)
        run._r.append(instrText)
        run._r.append(fldChar2)
        
        run = para.add_run(' 页')
        run.font.size = Pt(10)

def add_footer_with_page_numbers(doc):
    """添加带页码的页脚"""
    section = doc.sections[0]
    footer = section.footer
    footer.is_linked_to_previous = False
    
    para = footer.paragraphs[0] if footer.paragraphs else footer.add_paragraph()
    para.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    
    run = para.add_run('© 2026 公司名称 - 第 ')
    run.font.size = Pt(9)
    
    # 页码
    run = para.add_run()
    fldChar1 = OxmlElement('w:fldChar')
    fldChar1.set(qn('w:fldCharType'), 'begin')
    run._r.append(fldChar1)
    
    instrText = OxmlElement('w:instrText')
    instrText.text = 'PAGE'
    run._r.append(instrText)
    
    fldChar2 = OxmlElement('w:fldChar')
    fldChar2.set(qn('w:fldCharType'), 'end')
    run._