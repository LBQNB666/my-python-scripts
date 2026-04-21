# 📝 Word 办公自动化脚本集

> 日常办公中常用的 Word 自动化脚本

## 📦 依赖安装

```bash
pip install python-docx
```

---

## 1. 批量生成邀请函

```python
from docx import Document
from docx.shared import Pt, Inches
from docx.enum.text import WD_PARAGRAPH_ALIGNMENT
from datetime import datetime

def batch_generate_invitations(names, event_name, event_time, event_location, output_folder='邀请函'):
    """批量生成邀请函"""
    
    for name in names:
        doc = Document()
        
        # 添加标题
        title = doc.add_heading('邀 请 函', level=0)
        title.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
        
        # 添加内容
        content = f'''
尊敬的 {name}  先生/女士：

    您好！

    诚挚邀请您参加「{event_name}」活动。

    活动时间：{event_time}
    活动地点：{event_location}

    期待您的光临！

此致
敬礼


2026年4月21日
'''
        doc.add_paragraph(content)
        
        # 保存
        filename = f'{output_folder}/{name}_邀请函.docx'
        doc.save(filename)
        print(f'已生成: {filename}')

# 使用
names = ['张三', '李四', '王五', '赵六', '钱七']
batch_generate_invitations(
    names=['张三', '李四', '王五'],
    event_name='Python技术分享会',
    event_time='2026年4月25日 14:00',
    event_location='北京市海淀区中关村大街1号',
    output_folder='邀请函'
)
```

---

## 2. 批量生成证书

```python
from docx import Document
from docx.shared import Pt, Cm
from docx.enum.text import WD_PARAGRAPH_ALIGNMENT

def generate_certificate(name, certificate_type, date, output_file):
    """生成证书"""
    
    doc = Document()
    
    # 设置页面边距
    sections = doc.sections
    for section in sections:
        section.top_margin = Cm(2)
        section.bottom_margin = Cm(2)
        section.left_margin = Cm(3)
        section.right_margin = Cm(3)
    
    # 标题
    title = doc.add_paragraph()
    title.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    run = title.add_run('证 书')
    run.font.size = Pt(44)
    run.font.bold = True
    
    # 副标题
    subtitle = doc.add_paragraph()
    subtitle.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    run = subtitle.add_run(certificate_type)
    run.font.size = Pt(20))
    
    # 空白行
    doc.add_paragraph()
    
    # 证书内容
    content = doc.add_paragraph()
    content.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    run = content.add_run(f'授予：{name}')
    run.font.size = Pt(28)
    
    doc.add_paragraph()
    
    # 说明
    desc = doc.add_paragraph()
    desc.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    run = desc.add_run('在本次活动中表现优异，特发此证。')
    run.font.size = Pt(16)
    
    doc.add_paragraph()
    doc.add_paragraph()
    
    # 日期
    date_para = doc.add_paragraph()
    date_para.alignment = WD_PARAGRAPH_ALIGNMENT.RIGHT
    run = date_para.add_run(date)
    run.font.size = Pt(16)
    
    doc.save(output_file)
    print(f'证书已生成: {output_file}')

# 使用
generate_certificate(
    name='张三',
    certificate_type='优秀学员证书',
    date='2026年4月21日',
    output_file='证书_张三.docx'
)
```

---

## 3. 批量替换 Word 模板内容

```python
from docx import Document
import re

def replace_template(template_file, replacements, output_file):
    """批量替换 Word 模板中的占位符"""
    
    doc = Document(template_file)
    
    # 遍历所有段落
    for paragraph in doc.paragraphs:
        for old_text, new_text in replacements.items():
            if old_text in paragraph.text:
                # 替换文本
                for run in paragraph.runs:
                    if old_text in run.text:
                        run.text = run.text.replace(old_text, new_text)
    
    # 遍历所有表格
    for table in doc.tables:
        for row in table.rows:
            for cell in row.cells:
                for paragraph in cell.paragraphs:
                    for old_text, new_text in replacements.items():
                        if old_text in paragraph.text:
                            for run in paragraph.runs:
                                if old_text in run.text:
                                    run.text = run.text.replace(old_text, new_text)
    
    doc.save(output_file)
    print(f'文件已生成: {output_file}')

# 使用
replacements = {
    '{{姓名}}': '张三',
    '{{部门}}': '技术部',
    '{{职位}}': '高级工程师',
    '{{日期}}': '2026年4月21日',
    '{{公司}}': '某某科技有限公司'
}

replace_template('模板.docx', replacements, '填写好的文档.docx')
```

---

## 4. 批量合并多个 Word 文档

```python
from docx import Document
import os
from pathlib import Path

def merge_word_documents(folder_path, output_file):
    """合并文件夹中所有 Word 文档"""
    
    merged_doc = Document()
    
    # 获取所有 docx 文件
    folder = Path(folder_path)
    docx_files = sorted(folder.glob('*.docx'))
    
    print(f'找到 {len(docx_files)} 个文档')
    
    for i, file_path in enumerate(docx_files):
        print(f'正在合并: {file_path.name}')
        
        doc = Document(file_path)
        
        # 添加分隔页
        if i > 0:
            merged_doc.add_page_break()
        
        # 添加文档标题
        heading = merged_doc.add_heading(file_path.stem, level=1)
        
        # 复制所有段落
        for para in doc.paragraphs:
            text = para.text.strip()
            if text:
                new_para = merged_doc.add_paragraph(text)
                # 复制样式
                if para.style:
                    new_para.style = para.style
        
        # 复制表格
        for table in doc.tables:
            new_table = merged_doc.add_table(
                rows=len(table.rows),
                cols=len(table.columns)
            )
            for i, row in enumerate(table.rows):
                for j, cell in enumerate(row.cells):
                    new_table.cell(i, j).text = cell.text
    
    merged_doc.save(output_file)
    print(f'合并完成: {output_file}')

# 使用
merge_word_documents('文档文件夹', '合并文档.docx')
```

---

## 5. 提取 Word 中的所有图片

```python
from docx import Document
import os
from pathlib import Path
import zipfile
import shutil

def extract_images_from_docx(docx_file, output_folder='提取的图片'):
    """提取 Word 文档中的所有图片"""
    
    # 创建输出文件夹
    os.makedirs(output_folder, exist_ok=True)
    
    # docx 本质上是 zip 文件
    with zipfile.ZipFile(docx_file, 'r') as zip_ref:
        # 查找所有图片
        image_files = [f for f in zip_ref.namelist() 
                      if f.startswith('word/media/')]
        
        print(f'找到 {len(image_files)} 张图片')
        
        for image_file in image_files:
            # 提取图片
            filename = os.path.basename(image_file)
            extract_path = os.path.join(output_folder, filename)
            
            with zip_ref.open(image_file) as source:
                with open(extract_path, 'wb') as target:
                    target.write(source.read())
            
            print(f'已提取: {filename}')
    
    return len(image_files)

# 使用
count = extract_images_from_docx('文档.docx', 'images')
print(f'共提取 {count} 张图片')
```

---

## 6. 给 Word 文档添加水印

```python
from docx import Document
from docx.shared import Pt, RGBColor, Cm
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

def add_watermark_to_docx(input_file, output_file, watermark_text='CONFIDENTIAL'):
    """给 Word 文档添加水印"""
    
    doc = Document(input_file)
    
    # 获取 header
    section = doc.sections[0]
    header = section.header
    header.is_linked_to_previous = False
    
    # 添加水印
    header_para = header.paragraphs[0]
    header_para.alignment = 1  # center
    
    # 创建水印 Run
    run = header_para.add_run()
    run.text = watermark_text
    
    # 设置字体
    run.font.name = '微软雅黑'
    run.font.size = Pt(60)
