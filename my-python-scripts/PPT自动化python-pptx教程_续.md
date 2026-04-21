# 第3页：规格参数
    slide3 = prs.slides.add_slide(prs.slide_layouts[1])
    slide3.shapes.title.text = '规格参数'
    
    content = slide3.placeholders[1]
    tf = content.text_frame
    tf.clear()
    
    for key, value in product_info['specs'].items():
        p = tf.add_paragraph()
        p.text = f"{key}: {value}"
        p.level = 0
    
    # 第4页：结束页
    slide4 = prs.slides.add_slide(prs.slide_layouts[6])
    
    end_text = slide4.shapes.add_textbox(Inches(1), Inches(3), Inches(8), Inches(2))
    tf = end_text.text_frame
    p = tf.paragraphs[0]
    p.text = "感谢观看"
    p.font.size = Pt(40)
    p.alignment = PP_ALIGN.CENTER
    
    contact = tf.add_paragraph()
    contact.text = product_info['contact']
    contact.font.size = Pt(18)
    contact.alignment = PP_ALIGN.CENTER
    
    prs.save(output_file)
    print(f'PPT已生成: {output_file}')

# 使用
product = {
    'name': '智能手表 Pro',
    'tagline': '让科技改变生活',
    'image': 'product.jpg',
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
        '重量': '32g',
        '连接': '蓝牙5.0'
    },
    'contact': '电话: 400-888-8888'
}

create_product_ppt(product, '产品介绍.pptx')
```

### 案例 2: 批量生成数据汇报 PPT

```python
def create_report_ppt(data_list, output_file):
    """批量生成数据汇报 PPT"""
    
    prs = Presentation()
    
    for data in data_list:
        # 添加幻灯片
        slide = prs.slides.add_slide(prs.slide_layouts[5])  # 标题+内容
        
        # 设置标题
        slide.shapes.title.text = data['title']
        
        # 设置内容
        content = slide.placeholders[1]
        tf = content.text_frame
        tf.clear()
        
        for item in data['items']:
            p = tf.add_paragraph()
            p.text = f"• {item}"
        
        # 添加图表
        if 'chart_data' in data:
            chart = data['chart_data']
            chart_data = CategoryChartData()
            chart_data.categories = chart['categories']
            chart_data.add_series('数据', chart['values'])
            
            x, y, cx, cy = Inches(1), Inches(4), Inches(8), Inches(3)
            slide.shapes.add_chart(
                XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, chart_data
            )
    
    prs.save(output_file)
    print(f'汇报PPT已生成: {output_file}')

# 使用
report_data = [
    {
        'title': '2026年Q1销售报告',
        'items': ['总收入: 100万', '同比增长: 20%', '客户数: 500+'],
        'chart_data': {
            'categories': ['1月', '2月', '3月'],
            'values': (30, 35, 40)
        }
    },
    {
        'title': '2026年Q2销售报告',
        'items': ['总收入: 120万', '同比增长: 25%', '客户数: 600+'],
        'chart_data': {
            'categories': ['4月', '5月', '6月'],
            'values': (35, 40, 45)
        }
    }
]

create_report_ppt(report_data, '季度汇报.pptx')
```

### 案例 3: 创建图片轮播

```python
import os

def create_image_carousel(image_folder, output_file):
    """创建图片轮播 PPT"""
    
    prs = Presentation()
    prs.slide_width = Inches(10)
    prs.slide_height = Inches(7.5)
    
    # 获取所有图片
    images = [f for f in os.listdir(image_folder) 
              if f.lower().endswith(('.jpg', '.png', '.jpeg'))]
    
    for i, image in enumerate(images):
        # 空白布局
        slide = prs.slides.add_slide(prs.slide_layouts[6])
        
        # 添加图片（居中）
        img_path = os.path.join(image_folder, image)
        img = slide.shapes.add_picture(
            img_path,
            Inches(0), Inches(0),
            width=Inches(10), height=Inches(7.5)
        )
        
        # 添加页码
        page_num = slide.shapes.add_textbox(Inches(9), Inches(7), Inches(1), Inches(0.5))
        page_num.text_frame.text = f'{i+1}/{len(images)}'
        page_num.text_frame.paragraphs[0].font.size = Pt(12)
        page_num.text_frame.paragraphs[0].alignment = PP_ALIGN.RIGHT
    
    prs.save(output_file)
    print(f'图片轮播已生成: {output_file}')

# 使用
create_image_carousel('images/', '轮播.pptx')
```

---

## 📦 安装依赖

```bash
pip install python-pptx Pillow
```

---

## 📋 三大 Office 自动化对比

| 方面 | Word (python-docx) | Excel (pandas+openpyxl) | PPT (python-pptx) |
|------|---------------------|--------------------------|-------------------|
| **主要库** | python-docx | pandas + openpyxl | python-pptx |
| **核心功能** | 文档编辑 | 数据处理 | 幻灯片制作 |
| **擅长** | 报告生成、合同 | 数据分析、报表 | 演示文稿 |
| **难度** | 中等 | 中等 | 较简单 |

---

## 🔄 更新日志

- **2026-04-21**: 创建三大办公软件自动化教程
- Word 自动化 (python-docx)
- Excel 自动化 (pandas + openpyxl)
- PPT 自动化 (python-pptx)
