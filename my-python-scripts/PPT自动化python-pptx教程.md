# 📊 PPT 自动化 python-pptx 完整教程

> 用 Python 操作 PowerPoint 的全面指南

## 📂 目录

1. [安装与基础](#安装与基础)
2. [创建幻灯片](#创建幻灯片)
3. [文本操作](#文本操作)
4. [图片和形状](#图片和形状)
5. [表格和图表](#表格和图表)
6. [实用案例](#实用案例)

---

## 📦 安装与基础

### 安装库

```bash
pip install python-pptx Pillow
```

### 导入

```python
from pptx import Presentation
from pptx.util import Inches, Pt, Emu
from pptx.dml.color import RgbColor
from pptx.enum.text import PP_ALIGN
```

---

## 📄 创建幻灯片

### 1. 创建新演示文稿

```python
prs = Presentation()
prs.save('demo.pptx')
```

### 2. 打开现有文件

```python
prs = Presentation('existing.pptx')
prs.save('modified.pptx')
```

### 3. 选择布局

```python
# 可用布局 (0-9)
layouts = prs.slide_layouts
for i, layout in enumerate(layouts):
    print(f'{i}: {layout.name}')

# 常用布局
blank_layout = prs.slide_layouts[6]  # 空白
title_layout = prs.slide_layouts[0]  # 标题
title_content_layout = prs.slide_layouts[1]  # 标题+内容
```

### 4. 添加幻灯片

```python
# 使用空白布局
slide = prs.slides.add_slide(prs.slide_layouts[6])

# 使用标题布局
slide = prs.slides.add_slide(prs.slide_layouts[0])
```

---

## ✍️ 文本操作

### 1. 添加文本框

```python
from pptx.util import Inches, Pt

# 添加文本框
left = Inches(1)
top = Inches(1)
width = Inches(8)
height = Inches(1)

textbox = slide.shapes.add_textbox(left, top, width, height)
text_frame = textbox.text_frame
text_frame.text = "这是文本框内容"
```

### 2. 设置文本样式

```python
# 获取或创建段落
paragraph = text_frame.paragraphs[0]

# 设置字体
paragraph.font.name = '微软雅黑'
paragraph.font.size = Pt(24)
paragraph.font.bold = True
paragraph.font.italic = False
paragraph.font.color.rgb = RgbColor(0, 112, 192)  # 蓝色

# 设置对齐
paragraph.alignment = PP_ALIGN.CENTER
```

### 3. 多级文本

```python
# 多段落
p1 = text_frame.paragraphs[0]
p1.text = "标题"
p1.level = 0  # 第一级

p2 = text_frame.add_paragraph()
p2.text = "副标题"
p2.level = 1  # 第二级

p3 = text_frame.add_paragraph()
p3.text = "内容"
p3.level = 2  # 第三级
```

### 4. 使用占位符

```python
# 如果布局有占位符
title_placeholder = slide.shapes.title
title_placeholder.text = "幻灯片标题"

content_placeholder = slide.placeholders[1]
content_placeholder.text = "这是内容"
```

---

## 🖼️ 图片和形状

### 1. 添加图片

```python
# 添加图片
slide.shapes.add_picture('image.jpg', Inches(1), Inches(2), width=Inches(4))

# 设置图片大小
slide.shapes.add_picture('image.jpg', Inches(1), Inches(2), width=Inches(3), height=Inches(2))
```

### 2. 添加形状

```python
from pptx.enum.shapes import MSO_SHAPE

# 添加矩形
rect = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, Inches(1), Inches(1), Inches(2), Inches(1))
rect.fill.solid()
rect.fill.fore_color.rgb = RgbColor(0, 112, 192)

# 添加圆形
ellipse = slide.shapes.add_shape(MSO_SHAPE.OVAL, Inches(4), Inches(1), Inches(1), Inches(1))

# 添加箭头
arrow = slide.shapes.add_shape(MSO_SHAPE.RIGHT_ARROW, Inches(1), Inches(3), Inches(2), Inches(0.5))
```

### 3. 形状样式

```python
# 设置填充色
shape.fill.solid()
shape.fill.fore_color.rgb = RgbColor(255, 0, 0)

# 设置边框
shape.line.color.rgb = RgbColor(0, 0, 0)
shape.line.width = Pt(2)

# 设置透明度
shape.fill.solid()
shape.fill.fore_color.rgb = RgbColor(255, 0, 0)
shape.fill.fore_color.brightness = 0.5
```

---

## 📊 表格和图表

### 1. 添加表格

```python
from pptx.util import Inches

# 添加表格 (3行4列)
table = slide.shapes.add_table(3, 4, Inches(1), Inches(2), Inches(6), Inches(1.5)).table

# 设置表格内容
table.cell(0, 0).text = '姓名'
table.cell(0, 1).text = '年龄'
table.cell(0, 2).text = '部门'
table.cell(0, 3).text = '职位'

# 设置列宽
table.columns[0].width = Inches(1.5)
table.columns[1].width = Inches(1)
table.columns[2].width = Inches(1.5)
table.columns[3].width = Inches(2)
```

### 2. 添加图表

```python
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE

# 创建图表数据
chart_data = CategoryChartData()
chart_data.categories = ['东区', '西区', '南区', '北区']
chart_data.add_series('销售额', (19.2, 21.4, 16.7, 22.3))

# 添加柱状图
x, y, cx, cy = Inches(1), Inches(2), Inches(5), Inches(3)
chart = slide.shapes.add_chart(
    XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, chart_data
).chart

# 设置图表标题
chart.has_title = True
chart.chart_title.text_frame.text = '区域销售统计'
```

### 3. 图表类型

```python
# 柱状图
XL_CHART_TYPE.COLUMN_CLUSTERED
XL_CHART_TYPE.BAR_CLUSTERED

# 折线图
XL_CHART_TYPE.LINE

# 饼图
XL_CHART_TYPE.PIE

# 面积图
XL_CHART_TYPE.AREA
```

---

## 🎯 实用案例

### 案例 1: 生成产品介绍 PPT

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RgbColor

def create_product_ppt(product_info, output_file):
    """生成产品介绍 PPT"""
    
    prs = Presentation()
    
    # 第1页：封面
    slide = prs.slides.add_slide(prs.slide_layouts[6])
    
    # 添加标题
    title_box = slide.shapes.add_textbox(Inches(1), Inches(2), Inches(8), Inches(1.5))
    tf = title_box.text_frame
    p = tf.paragraphs[0]
    p.text = product_info['name']
    p.font.size = Pt(44)
    p.font.bold = True
    p.alignment = PP_ALIGN.CENTER
    
    # 添加副标题
    subtitle = tf.add_paragraph()
    subtitle.text = product_info['tagline']
    subtitle.font.size = Pt(24)
    subtitle.alignment = PP_ALIGN.CENTER
    
    # 添加产品图片
    if 'image' in product_info:
        slide.shapes.add_picture(
            product_info['image'],
            Inches(3), Inches(4), width=Inches(4)
        )
    
    # 第2页：产品特点
    slide2 = prs.slides.add_slide(prs.slide_layouts[1])
    slide2.shapes.title.text = '产品特点'
    
    content = slide2.placeholders[1]
    tf = content.text_frame
    tf.clear()
    
    for feature in product_info['features']:
        p = tf.add_paragraph()
        p.text = f"✓ {feature}"
        p.level = 0
    
    # 第3页：规格参数
    slide3 = prs.slides.add