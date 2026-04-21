# 📊 PPT 办公自动化脚本集

> 日常办公中常用的 PowerPoint 自动化脚本

## 📦 依赖安装

```bash
pip install python-pptx Pillow
```

---

## 1. 批量生成产品介绍 PPT

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RgbColor
from pptx.enum.shapes import MSO_SHAPE

def batch_create_product_ppt(products, output_folder='产品PPT'):
    """批量生成产品介绍 PPT"""
    
    import os
    os.makedirs(output_folder, exist_ok=True)
    
    for product in products:
        prs = Presentation()
        
        # 第1页：封面
        slide1 = prs.slides.add_slide(prs.slide_layouts[6])
        
        # 添加背景色块
        shape = slide1.shapes.add_shape(
            MSO_SHAPE.RECTANGLE, Inches(0), Inches(0), 
            Inches(10), Inches(7.5)
        )
        shape.fill.solid()
        shape.fill.fore_color.rgb = RgbColor(68, 114, 196)
        shape.line.fill.background()
        
        # 标题
        title_box = slide1.shapes.add_textbox(Inches(1), Inches(2.5), Inches(8), Inches(1))
        tf = title_box.text_frame
        p = tf.paragraphs[0]
        p.text = product['name']
        p.font.size = Pt(48)
        p.font.bold = True
        p.font.color.rgb = RgbColor(255, 255, 255)
        p.alignment = PP_ALIGN.CENTER
        
        # 副标题
        subtitle_box = slide1.shapes.add_textbox(Inches(1), Inches(4), Inches(8), Inches(1))
        tf = subtitle_box.text_frame
        p = tf.paragraphs[0]
        p.text = product['tagline']
        p.font.size = Pt(24)
        p.font.color.rgb = RgbColor(255, 255, 255)
        p.alignment = PP_ALIGN.CENTER
        
        # 第2页：产品特点
        slide2 = prs.slides.add_slide(prs.slide_layouts[1])
        slide2.shapes.title.text = '产品特点'
        
        content = slide2.placeholders[1]
        tf = content.text_frame
        tf.clear()
        
        for feature in product['features']:
            p = tf.add_paragraph()
            p.text = f'✓ {feature}'
            p.level = 0
        
        # 第3页：规格参数
        slide3 = prs.slides.add_slide(prs.slide_layouts[1])
        slide3.shapes.title.text = '规格参数'
        
        content = slide3.placeholders[1]
        tf = content.text_frame
        tf.clear()
        
        for spec_key, spec_value in product['specs'].items():
            p = tf.add_paragraph()
            p.text = f'{spec_key}：{spec_value}'
        
        # 第4页：结束页
        slide4 = prs.slides.add_slide(prs.slide_layouts[6])
        
        shape = slide4.shapes.add_shape(
            MSO_SHAPE.RECTANGLE, Inches(0), Inches(0),
            Inches(10), Inches(7.5)
        )
        shape.fill.solid()
        shape.fill.fore_color.rgb = RgbColor(68, 114, 196)
        shape.line.fill.background()
        
        end_text = slide4.shapes.add_textbox(Inches(1), Inches(3), Inches(8), Inches(2))
        tf = end_text.text_frame
        p = tf.paragraphs[0]
        p.text = '感谢观看'
        p.font.size = Pt(48)
        p.font.color.rgb = RgbColor(255, 255, 255)
        p.alignment = PP_ALIGN.CENTER
        
        # 保存
        output_file = f'{output_folder}/{product["name"]}.pptx'
        prs.save(output_file)
        print(f'已生成: {output_file}')

# 使用
products = [
    {
        'name': '智能手表 Pro',
        'tagline': '让科技改变生活',
        'features': [
            '1.43英寸 AMOLED 高清显示屏',
            '支持心率、血氧、血压监测',
            '7天超长续航',
            '50米防水深度',
            '支持GPS定位'
        ],
        'specs': {
            '屏幕': '1.43英寸 AMOLED',
            '电池': '450mAh',
            '防水': '50米',
            '重量': '32g'
        }
    },
    {
        'name': '无线耳机 X1',
        'tagline': '沉浸式音乐体验',
        'features': [
            '主动降噪技术',
            '30小时续航',
            'Hi-Fi音质',
            '触控操作',
            '蓝牙5.2'
        ],
        'specs': {
            '驱动单元': '12mm',
            '电池': '50mAh',
            '重量': '5g/只',
            '连接': '蓝牙5.2'
        }
    }
]

batch_create_product_ppt(products)
```

---

## 2. 生成数据汇报 PPT

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RgbColor
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE

def create_data_report(title, data_summary, chart_data, output_file):
    """生成数据汇报 PPT"""
    
    prs = Presentation()
    
    # 第1页：封面
    slide1 = prs.slides.add_slide(prs.slide_layouts[6])
    
    title_box = slide1.shapes.add_textbox(Inches(1), Inches(2.5), Inches(8), Inches(1.5))
    tf = title_box.text_frame
    p = tf.paragraphs[0]
    p.text = title
    p.font.size = Pt(44)
    p.font.bold = True
    p.alignment = PP_ALIGN.CENTER
    
    # 日期
    from datetime import datetime
    date_box = slide1.shapes.add_textbox(Inches(1), Inches(4.5), Inches(8), Inches(0.5))
    tf = date_box.text_frame
    p = tf.paragraphs[0]
    p.text = datetime.now().strftime('%Y年%m月%d日')
    p.font.size = Pt(18)
    p.alignment = PP_ALIGN.CENTER
    
    # 第2页：数据摘要
    slide2 = prs.slides.add_slide(prs.slide_layouts[1])
    slide2.shapes.title.text = '数据摘要'
    
    content = slide2.placeholders[1]
    tf = content.text_frame
    tf.clear()
    
    for key, value in data_summary.items():
        p = tf.add_paragraph()
        p.text = f'{key}：{value}'
        p.level = 0
    
    # 第3页：图表分析
    slide3 = prs.slides.add_slide(prs.slide_layouts[5])
    slide3.shapes.title.text = '数据分析'
    
    # 创建图表
    chart = CategoryChartData()
    chart.categories = chart_data['categories']
    chart.add_series(chart_data['series_name'], chart_data['values'])
    
    x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(4)
    chart_shape = slide3.shapes.add_chart(
        XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, chart
    )
    chart_shape.chart.title.text = chart_data.get('title', '数据图表')
    
    # 第4页：结论
    slide4 = prs.slides.add_slide(prs.slide_layouts[1])
    slide4.shapes.title.text = '结论与建议'
    
    content = slide4.placeholders[1]
    tf = content.text_frame
    tf.clear()
    
    conclusions = [
        '1. 本季度业绩稳步增长',
        '2. 建议加大市场推广力度',
        '3. 继续优化产品结构'
    ]
    
    for conclusion in conclusions:
        p = tf.add_paragraph()
        p.text = conclusion
    
    prs.save(output_file)
    print(f'数据汇报PPT已生成: {output_file}')

# 使用
create_data_report(
    title='2026年Q1季度报告',
    data_summary={
        '总收入': '1000万元',
        '同比增长': '25%',
        '新客户': '150位',
        '客户满意度': '95%'
    },
    chart_data={
        'categories': ['1月', '2月', '3月'],
        'series_name': '销售额(万元)',
        'values': (