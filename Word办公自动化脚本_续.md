run.font.name = '微软雅黑'
    run.font.size = Pt(60)
    run.font.color.rgb = RGBColor(128, 128, 128)  # 灰色
    run._element.rPr.rFonts.set(qn('w:eastAsia'), '微软雅黑')
    
    doc.save(output_file)
    print(f'水印已添加: {output_file}')

# 使用
add_watermark_to_docx('原始文档.docx', '带水印文档.docx')
```

---

## 7. 批量转换文档为 PDF

```python
from docx import Document
import os
from pathlib import Path

def batch_convert_to_pdf(folder_path, output_folder='PDF文件'):
    """批量将 Word 文档转换为 PDF"""
    
    # 注意：需要安装 python-docx 和 pandoc
    # pip install python-docx
    # 需要系统安装 pandoc: https://pandoc.org/
    
    os.makedirs(output_folder, exist_ok=True)
    
    folder = Path(folder_path)
    docx_files = folder.glob('*.docx')
    
    for file_path in docx_files:
        doc = Document(file_path)
        
        # pandoc 转换命令
        output_path = Path(output_folder) / f'{file_path.stem}.pdf'
        
        # 使用 python-docx 直接保存（如果支持）
        # 或者调用 pandoc
        import subprocess
        cmd = f'pandoc "{file_path}" -o "{output_path}"'
        subprocess.run(cmd, shell=True)
        
        print(f'已转换: {output_path.name}')

# 使用
batch_convert_to_pdf('Word文档', 'PDF文件')
```

---

## 8. 批量删除空白行

```python
from docx import Document

def remove_empty_lines(input_file, output_file):
    """删除 Word 文档中的空白行"""
    
    doc = Document(input_file)
    
    # 删除空白段落
    paragraphs_to_remove = []
    for para in doc.paragraphs:
        if not para.text.strip():
            paragraphs_to_remove.append(para)
    
    for para in paragraphs_to_remove:
        p = para._element
        p.getparent().remove(p)
    
    # 删除空白表格行
    for table in doc.tables:
        rows_to_remove = []
        for row in table.rows:
            if not any(cell.text.strip() for cell in row.cells):
                rows_to_remove.append(row)
        
        for row in rows_to_remove:
            tr = row._tr
            tr.getparent().remove(tr)
    
    doc.save(output_file)
    print(f'空白行已删除: {output_file}')

# 使用
remove_empty_lines('原始文档.docx', '清洁文档.docx')
```

---

## 9. 生成员工手册目录

```python
from docx import Document
from docx.shared import Pt
from docx.enum.text import WD_PARAGRAPH_ALIGNMENT

def generate_toc_page():
    """生成目录页"""
    
    doc = Document()
    
    # 添加标题
    title = doc.add_heading('目 录', level=0)
    title.alignment = WD_PARAGRAPH_ALIGNMENT.CENTER
    
    # 目录内容
    toc_items = [
        ('第一章', '公司简介', '3'),
        ('第二章', '企业文化', '5'),
        ('第三章', '组织架构', '7'),
        ('第四章', '规章制度', '9'),
        ('第五章', '薪酬福利', '12'),
        ('第六章', '员工发展', '15'),
        ('第七章', '行为规范', '18'),
    ]
    
    for chapter, title, page in toc_items:
        p = doc.add_paragraph()
        p.add_run(f'{chapter}  {title}')
        # 添加页码（使用制表符对齐）
        p.add_run(f'\t{page}')
    
    doc.save('目录.docx')
    print('目录已生成')

# 使用
generate_toc_page()
```

---

## 📋 脚本清单

| 脚本 | 功能 | 适用场景 |
|------|------|----------|
| `batch_generate_invitations` | 批量生成邀请函 | 活动邀请 |
| `generate_certificate` | 生成证书 | 颁奖、认证 |
| `replace_template` | 批量替换模板 | 数据填充 |
| `merge_word_documents` | 合并文档 | 资料整理 |
| `extract_images_from_docx` | 提取图片 | 资源回收 |
| `add_watermark_to_docx` | 添加水印 | 保密文档 |
| `batch_convert_to_pdf` | 批量转PDF | 格式转换 |
| `remove_empty_lines` | 删除空白行 | 文档清理 |
| `generate_toc_page` | 生成目录 | 手册制作 |
