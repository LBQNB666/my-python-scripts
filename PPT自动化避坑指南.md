# ⚠️ PPT 自动化避坑指南

> 真正踩过坑才能总结出的经验

## 🔥 常见问题与解决方案

---

## 1. 内存占用过高

### 问题描述
批量生成 PPT 时内存占用过高，程序变慢或崩溃。

### 解决方案

```python
from pptx import Presentation
import gc

def create_ppts_batch_optimized(template_path, data_list, output_folder):
    """优化批量生成 PPT"""
    
    for i, data in enumerate(data_list):
        try:
            # 使用模板（如果需要）
            if template_path:
                prs = Presentation(template_path)
            else:
                prs = Presentation()
            
            # 添加内容
            slide = prs.slides.add_slide(prs.slide_layouts[1])
            # ... 处理内容
            
            # 保存
            output_path = f'{output_folder}/output_{i}.pptx'
            prs.save(output_path)
            
        except Exception as e:
            print(f'处理第 {i} 个时出错: {e}')
        
        finally:
            # 关键：显式删除对象
            del prs
            del slide
            gc.collect()  # 强制垃圾回收

# 使用上下文管理器
from contextlib import contextmanager

@contextmanager
def managed_presentation():
    """管理 PPT 对象生命周期"""
    prs = None
    try:
        prs = Presentation()
        yield prs
    finally:
        if prs:
            del prs
        gc.collect()
```

---

## 2. 图片失真问题

### 问题描述
插入的图片模糊或失真。

### 解决方案

```python
from pptx import Presentation
from pptx.util import Inches, Emu
from PIL import Image

def add_image_correctly(slide, image_path, left, top, width=None, height=None):
    """正确插入图片，保持清晰度"""
    
    # 方法1：指定宽度，高度自动计算
    # 适用于知道目标宽度的情况
    slide.shapes.add_picture(
        image_path,
        left, top,
        width=Inches(4)  # 指定宽度
    )
    
    # 方法2：使用 Emu 单位（更精确）
    # 1 inch = 914400 EMU
    width_emu = Inches(4)
    height_emu = Inches(3)
    
    slide.shapes.add_picture(
        image_path,
        left, top,
        width=width_emu,
        height=height_emu
    )

def resize_image_for_ppt(image_path, max_width_inches=8, max_height_inches=6):
    """调整图片大小适合 PPT"""
    with Image.open(image_path) as img:
        width, height = img.size
    
    # 计算比例
    ratio = min(max_width_inches * 100 / width, max_height_inches * 100 / height)
    
    if ratio < 1:
        new_width = int(width * ratio)
        new_height = int(height * ratio)
        
        # 保存调整后的图片
        resized_path = image_path.replace('.', '_resized.')
        img.resize((new_width, new_height), Image.LANCZOS).save(resized_path)
        return resized_path
    
    return image_path

def add_image_with_high_quality(prs, image_path, left, top, width):
    """高质量插入图片"""
    # 确保图片格式正确（PNG/JPEG）
    # 如果是其他格式，先转换
    with Image.open(image_path) as img:
        if img.mode not in ('RGB', 'RGBA'):
            img = img.convert('RGB')
        
        # 保存为临时高清文件
        temp_path = 'temp_high_quality.jpg'
        img.save(temp_path, quality=95)
    
    # 插入
    slide = prs.slides.add_slide(prs.slide_layouts[6])
    slide.shapes.add_picture(temp_path, left, top, width=width)
```

---

## 3. 字体不显示问题

### 问题描述
设置的字体在演示时不显示，变成默认字体。

### 解决方案

```python
from pptx import Presentation
from pptx.util import Pt
from pptx.dml.color import RgbColor
from pptx.enum.text import PP_ALIGN

def set_font_correctly(text_frame):
    """正确设置字体"""
    for paragraph in text_frame.paragraphs:
        for run in paragraph.runs:
            # 设置英文字体
            run.font.name = 'Arial'
            
            # 设置中文字体（关键）
            run.font.name.eastAsia = '微软雅黑'
            
            # 设置大小
            run.font.size = Pt(14)
            
            # 设置颜色
            run.font.color.rgb = RgbColor(0, 0, 0)

def create_textbox_with_font(slide, left, top, width, height, text, 
                              font_name='微软雅黑', font_size=14):
    """创建带字体的文本框"""
    textbox = slide.shapes.add_textbox(left, top, width, height)
    tf = textbox.text_frame
    tf.word_wrap = True  # 自动换行
    
    p = tf.paragraphs[0]
    p.text = text
    p.font.name = font_name
    p.font.size = Pt(font_size)
    p.alignment = PP_ALIGN.LEFT
    
    return textbox

# 检查字体可用性
import os

def check_font_available(font_name):
    """检查字体是否可用"""
    # Windows 字体目录
    font_dirs = [
        r'C:\Windows\Fonts',
        r'C:\Users\{}\AppData\Local\Microsoft\Windows\Fonts'.format(os.getenv('USERNAME', ''))
    ]
    
    font_files = [f'{font_name}.ttf', f'{font_name}.TTF', f'{font_name}.otf', f'{font_name}.OTF']
    
    for font_dir in font_dirs:
        if os.path.exists(font_dir):
            for font_file in font_files:
                if os.path.exists(os.path.join(font_dir, font_file)):
                    return True
    return False
```

---

## 4. 布局选择错误

### 问题描述
选择的幻灯片布局不对，内容显示错乱。

### 解决方案

```python
from pptx import Presentation
from pptx.util import Inches

def select_layout_correctly(prs):
    """正确选择布局"""
    
    # 打印所有可用布局
    print('可用布局:')
    for i, layout in enumerate(prs.slide_layouts):
        print(f'{i}: {layout.name}')
        # 打印布局中的占位符
        for ph in layout.placeholders:
            print(f'   - Placeholder: {ph.placeholder_format.idx}, {ph.name}')
    
    # 常用布局
    # 0: 标题幻灯片 (Title Slide)
    # 1: 标题和内容 (Title and Content)
    # 2: 节标题 (Section Header)
    # 5: 空白 (Blank)
    # 6: 标题和内容（带日期）
    # 7: 标题和内容（带图片）
    
    return prs.slide_layouts[1]  # 返回"标题和内容"布局

def use_layout_with_placeholders(prs):
    """使用带占位符的布局"""
    
    # 标题和内容布局
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    
    # 获取占位符
    title_placeholder = slide.shapes.title
    if title_placeholder:
        title_placeholder.text = '这是标题'
    
    # 获取内容占位符（通常 index 为 1）
    content_placeholder = None
    for ph in slide.placeholders:
        if ph.placeholder_format.idx == 1:  # 内容占位符通常 idx=1
            content_placeholder = ph
            break
    
    if content_placeholder:
        tf = content_placeholder.text_frame
        tf.clear()
        
        # 添加内容
        for item in ['第一点', '第二点', '第三点']:
            p = tf.add_paragraph()
            p.text = item
            p.level = 0

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
            p = tf