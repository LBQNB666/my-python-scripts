def create_blank_slide_with_shapes(prs):
    """创建空白幻灯片并手动添加形状"""
    
    slide = prs.slides.add_slide(prs.slide_layouts[6])  # 空白布局
    
    # 手动添加标题
    title_box = slide.shapes.add_textbox(Inches(0.5), Inches(0.3), Inches(9), Inches(1))
    tf = title_box.text_frame
    p = tf.paragraphs[0]
    p.text = '幻灯片标题'
    p.font.size = Pt(32)
    p.font.bold = True
    
    # 手动添加内容
    content_box = slide.shapes.add_textbox(Inches(0.5), Inches(1.5), Inches(9), Inches(5))
    tf = content_box.text_frame
    tf.word_wrap = True
    
    for i, text in enumerate(['内容1', '内容2', '内容3']):
        if i == 0:
            p = tf.paragraphs[0]
        else:
            p = tf.add_paragraph()
        p.text = text
```

---

## 5. 动画效果丢失

### 问题描述
使用 python-pptx 无法添加动画效果。

### 解决方案

```python
# python-pptx 本身不支持动画效果！
# 动画效果需要使用以下方案：

# 方案1：使用 python-pptx-macro-enabled (非官方)
# 不推荐，功能不完整

# 方案2：手动编辑 PPTX 文件（XML）
from pptx import Presentation
from pptx.oxml.ns import qn
from lxml import etree

def add_simple_animation(slide, shape_id):
    """尝试添加简单动画（有限支持）"""
    
    # 这是一个实验性功能，可能不完整
    # python-pptx 对动画的支持非常有限
    
    # 获取形状的 XML
    sp = slide.shapes[shape_id]._element
    
    # 添加动画（取决于 PowerPoint 版本）
    # 大部分动画需要使用 COM 接口或 Office JS API
    
    print('警告: python-pptx 对动画支持有限')
    print('建议使用 PowerPoint 本身录制动画')

# 方案3：使用 COM 接口 (Windows only)
try:
    from win32com.client import Dispatch
    
    def add_animation_with_com(pptx_path, output_path):
        """使用 PowerPoint COM 接口添加动画"""
        ppt = Dispatch('PowerPoint.Application')
        ppt.Visible = True
        
        presentation = ppt.Presentations.Open(pptx_path)
        
        # 获取第一张幻灯片
        slide = presentation.Slides(1)
        
        # 添加进入动画
        shape = slide.Shapes(1)
        effect = slide.TimeLine.MainSequence.AddEffect(
            shape,  # 目标形状
            10,  # 效果类型 (msoAnimEffectFade)
            0,  # 触发方式
            0   # 持续时间
        )
        
        presentation.SaveAs(output_path)
        presentation.Close()
        ppt.Quit()

except ImportError:
    print('需要安装 pywin32 才能使用 COM 接口')
```

---

## 6. 表格格式问题

### 问题描述
创建的表格格式不美观或显示不正确。

### 解决方案

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RgbColor
from pptx.enum.text import PP_ALIGN

def create_table_correctly(slide, data, left, top, width, height):
    """正确创建表格"""
    
    rows = len(data)
    cols = len(data[0]) if data else 0
    
    # 添加表格
    table = slide.shapes.add_table(rows, cols, left, top, width, height).table
    
    # 设置列宽（平均分配）
    col_width = width // cols
    for i in range(cols):
        table.columns[i].width = col_width
    
    # 填充数据
    for row_idx, row_data in enumerate(data):
        for col_idx, cell_value in enumerate(row_data):
            cell = table.cell(row_idx, col_idx)
            cell.text = str(cell_value)
            
            # 设置对齐
            cell.text_frame.paragraphs[0].alignment = PP_ALIGN.CENTER
            
            # 设置字体（第一行作为表头）
            for para in cell.text_frame.paragraphs:
                for run in para.runs:
                    run.font.size = Pt(11)
                    if row_idx == 0:  # 表头
                        run.font.bold = True
                        run.font.color.rgb = RgbColor(255, 255, 255)

def style_table_professional(table):
    """专业表格样式"""
    
    # 设置表格边框
    for row in table.rows:
        for cell in row.cells:
            # 设置单元格边距
            cell.margin_left = Inches(0.05)
            cell.margin_right = Inches(0.05)
            cell.margin_top = Inches(0.02)
            cell.margin_bottom = Inches(0.02)
            
            # 设置垂直对齐
            cell.vertical_anchor = 'center'

# 表格跨行跨列（需要合并）
def merge_table_cells(table, start_row, start_col, end_row, end_col):
    """合并表格单元格"""
    cell_start = table.cell(start_row, start_col)
    cell_end = table.cell(end_row, end_col)
    
    # 只能通过 cell 方法合并
    merged = table.cell(start_row, start_col)
    for row in range(start_row, end_row + 1):
        for col in range(start_col, end_col + 1):
            if row == start_row and col == start_col:
                continue
            cell = table.cell(row, col)
            merged = merged.merge(cell)
```

---

## 7. 兼容 PowerPoint 2007/2010

### 问题描述
生成的文件在旧版 PowerPoint 中打不开。

### 解决方案

```python
from pptx import Presentation
from pptx.enum.text import PP_ALIGN

def create_compatible_pptx(output_path):
    """创建兼容旧版 PowerPoint 的文件"""
    
    prs = Presentation()
    
    # 设置兼容模式
    # python-pptx 默认生成 PowerPoint 2007+ 兼容的文件 (.pptx)
    # 这是最广泛兼容的格式
    
    # 避免使用：
    # - 过大的图片
    # - 特殊字体（使用系统内置字体）
    # - 复杂的动画效果
    
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    
    # 使用标准字体
    title = slide.shapes.title
    title.text = '标题'
    
    for para in title.text_frame.paragraphs:
        for run in para.runs:
            run.font.name = 'Arial'  # 标准字体
    
    prs.save(output_path)
    print(f'已保存兼容格式: {output_path}')

# 验证文件
def verify_pptx(file_path):
    """验证 PPTX 文件"""
    from zipfile import ZipFile
    
    try:
        with ZipFile(file_path, 'r') as zf:
            # 检查必要文件
            required = ['[Content_Types].xml', 'ppt/presentation.xml']
            for req in required:
                if req not in zf.namelist():
                    return False, f'缺少关键文件: {req}'
            
            return True, '文件结构正常'
    except Exception as e:
        return False, f'验证失败: {e}'
```

---

## 8. 批量替换文本

### 问题描述
需要批量替换 PPT 中的占位符文本。

### 解决方案

```python
from pptx import Presentation
from pptx.oxml.ns import qn

def batch_replace_text(prs, replacements):
    """
    批量替换 PPT 中的文本
    replacements: dict, {原文本: 新文本}
    """
    replaced_count = 0
    
    for slide in prs.slides:
        for shape in slide.shapes:
            # 处理形状中的文本
            if hasattr(shape, 'text_frame'):
                for paragraph in shape.text_frame.paragraphs:
                    for run in paragraph.runs:
                        for old_text, new_text in replacements.items():
                            if old_text in run.text:
                                run.text = run.text.replace(old_text, new_text)
                                replaced_count += 1
            
            # 处理表格中的文本
            if shape.has_table:
                for row in shape.table.rows:
                    for cell in row.cells:
                        for paragraph in cell.text_frame.paragraphs:
                            for run in paragraph.runs:
                                for old_text, new_text in replacements.items():
                                    if old_text in run.text:
                                        run.text = run.text.replace(old_text, new_text)
                                        replaced_count += 1
    
    return replaced_count

def replace_with_formatting(prs, old_text, new_text, font_name='微软雅黑', font_size=14):
    """替换文本并保留格式"""
    for slide in prs.slides:
        for shape in slide.shapes:
            if hasattr(shape, 'text_frame'):
                tf = shape.text_frame
                for para in tf.paragraphs:
                    for run in para.runs:
                        if old_text in