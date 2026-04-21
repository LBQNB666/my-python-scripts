chart_data={
        'categories': ['1月', '2月', '3月'],
        'series_name': '销售额(万元)',
        'values': (280, 320, 400)
    },
    output_file='季度汇报.pptx'
)
```

---

## 3. 批量插入图片到 PPT

```python
from pptx import Presentation
from pptx.util import Inches, Pt
import os
from pathlib import Path

def batch_insert_images(image_folder, output_file, images_per_slide=1):
    """批量插入图片到 PPT"""
    
    prs = Presentation()
    
    # 获取所有图片
    image_extensions = ('.jpg', '.jpeg', '.png', '.bmp', '.gif')
    images = [f for f in os.listdir(image_folder) 
              if f.lower().endswith(image_extensions)]
    
    print(f'找到 {len(images)} 张图片')
    
    for i, image_name in enumerate(images):
        # 创建幻灯片
        if images_per_slide == 1:
            slide = prs.slides.add_slide(prs.slide_layouts[6])  # 空白
        else:
            slide = prs.slides.add_slide(prs.slide_layouts[5])  # 标题+内容
        
        # 添加标题（图片名）
        title_box = slide.shapes.add_textbox(Inches(0.5), Inches(0.3), Inches(9), Inches(0.8))
        tf = title_box.text_frame
        p = tf.paragraphs[0]
        p.text = image_name
        p.font.size = Pt(16)
        p.font.bold = True
        
        # 添加图片
        image_path = os.path.join(image_folder, image_name)
        
        if images_per_slide == 1:
            # 单图模式：图片填满页面
            slide.shapes.add_picture(
                image_path,
                Inches(0.5), Inches(1),
                width=Inches(9), height=Inches(6)
            )
        else:
            # 多图模式
            slide.shapes.add_picture(
                image_path,
                Inches(1), Inches(1.5),
                width=Inches(8), height=Inches(5)
            )
        
        # 添加页码
        page_num = slide.shapes.add_textbox(Inches(9), Inches(7), Inches(1), Inches(0.5))
        page_num.text_frame.text = f'{i+1}/{len(images)}'
        page_num.text_frame.paragraphs[0].font.size = Pt(12)
        page_num.text_frame.paragraphs[0].alignment = PP_ALIGN.RIGHT
        
        print(f'已添加: {image_name}')
    
    prs.save(output_file)
    print(f'PPT已生成: {output_file}')

# 使用
batch_insert_images('images/', '图片展示.pptx')
```

---

## 4. 批量生成培训课件

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RgbColor

def create_training_course(course_title, chapters, output_file):
    """批量生成培训课件"""
    
    prs = Presentation()
    
    # 第1页：封面
    slide_cover = prs.slides.add_slide(prs.slide_layouts[6])
    
    # 标题
    title_box = slide_cover.shapes.add_textbox(Inches(1), Inches(2.5), Inches(8), Inches(1.5))
    tf = title_box.text_frame
    p = tf.paragraphs[0]
    p.text = course_title
    p.font.size = Pt(44)
    p.font.bold = True
    p.alignment = PP_ALIGN.CENTER
    
    # 副标题
    subtitle = slide_cover.shapes.add_textbox(Inches(1), Inches(4.5), Inches(8), Inches(1))
    tf = subtitle.text_frame
    p = tf.paragraphs[0]
    p.text = '内部培训资料'
    p.font.size = Pt(24)
    p.alignment = PP_ALIGN.CENTER
    
    # 目录页
    slide_toc = prs.slides.add_slide(prs.slide_layouts[1])
    slide_toc.shapes.title.text = '目录'
    
    content = slide_toc.placeholders[1]
    tf = content.text_frame
    tf.clear()
    
    for i, chapter in enumerate(chapters, start=1):
        p = tf.add_paragraph()
        p.text = f'第{i}章 {chapter["title"]}'
        p.level = 0
    
    # 各章节内容页
    for i, chapter in enumerate(chapters, start=1):
        # 章节标题页
        slide_chapter = prs.slides.add_slide(prs.slide_layouts[6])
        
        chapter_box = slide_chapter.shapes.add_textbox(Inches(1), Inches(3), Inches(8), Inches(1))
        tf = chapter_box.text_frame
        p = tf.paragraphs[0]
        p.text = f'第{i}章'
        p.font.size = Pt(24)
        p.alignment = PP_ALIGN.CENTER
        
        p = tf.add_paragraph()
        p.text = chapter['title']
        p.font.size = Pt(40)
        p.font.bold = True
        p.alignment = PP_ALIGN.CENTER
        
        # 章节内容页
        slide_content = prs.slides.add_slide(prs.slide_layouts[1])
        slide_content.shapes.title.text = chapter['title']
        
        content = slide_content.placeholders[1]
        tf = content.text_frame
        tf.clear()
        
        for point in chapter['points']:
            p = tf.add_paragraph()
            p.text = f'• {point}'
        
        # 添加思考题（如果有）
        if 'quiz' in chapter:
            slide_quiz = prs.slides.add_slide(prs.slide_layouts[1])
            slide_quiz.shapes.title.text = '思考题'
            
            content = slide_quiz.placeholders[1]
            tf = content.text_frame
            tf.clear()
            
            for j, question in enumerate(chapter['quiz'], start=1):
                p = tf.add_paragraph()
                p.text = f'{j}. {question}'
    
    # 结束页
    slide_end = prs.slides.add_slide(prs.slide_layouts[6])
    
    end_box = slide_end.shapes.add_textbox(Inches(1), Inches(3), Inches(8), Inches(2))
    tf = end_box.text_frame
    p = tf.paragraphs[0]
    p.text = '谢谢观看'
    p.font.size = Pt(48)
    p.alignment = PP_ALIGN.CENTER
    
    p = tf.add_paragraph()
    p.text = '如有疑问，请联系培训部'
    p.font.size = Pt(18)
    p.alignment = PP_ALIGN.CENTER
    
    prs.save(output_file)
    print(f'培训课件已生成: {output_file}')

# 使用
chapters = [
    {
        'title': '产品介绍',
        'points': [
            '产品定位与目标用户',
            '核心功能与优势',
            '使用方法演示'
        ],
        'quiz': ['产品的核心竞争力是什么？', '如何向客户介绍产品？']
    },
    {
        'title': '销售技巧',
        'points': [
            '客户需求分析',
            '产品展示技巧',
            '处理客户异议'
        ]
    },
    {
        'title': '常见问题',
        'points': [
            '价格相关问题',
            '技术支持问题',
            '售后服务政策'
        ]
    }
]

create_training_course('产品培训', chapters, '培训课件.pptx')
```

---

## 5. 批量生成会议纪要 PPT

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from datetime import datetime

def create_meeting_minutes(meeting_info, output_file):
    """生成会议纪要 PPT"""
    
    prs = Presentation()
    
    # 第1页：会议信息
    slide1 = prs.slides.add_slide(prs.slide_layouts[1])
    slide1.shapes.title.text = '会议纪要'
    
    content = slide1.placeholders[1]
    tf = content.text_frame
    tf.clear()
    
    info_items = [
        f"会议名称：{meeting_info['title']}",
        f"会议时间：{meeting_info['time']}",
        f"会议地点：{meeting_info['location']}",
        f"主持人：{meeting_info['host']}",
        f"记录人：{meeting_info['recorder']}",
        f"参加人员：{', '.join(meeting_info['attendees'])}"
    ]
    
    for item in info_items:
        p = tf.add_paragraph()
        p.text = item
    
    # 第2页：会议议程
    slide2 = prs.sl