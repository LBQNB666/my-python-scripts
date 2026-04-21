char2)
    
    run = para.add_run(' 页')
    run.font.size = Pt(9)
```

---

## 7. 批量处理大文件内存问题

### 问题描述
处理大量 Word 文档时内存占用过高。

### 解决方案

```python
import gc
from docx import Document

def process_large_documents(file_list, output_folder):
    """批量处理大文件时释放内存"""
    for i, file_path in enumerate(file_list):
        try:
            doc = Document(file_path)
            
            # 处理文档...
            
            # 保存
            doc.save(f'{output_folder}/processed_{i}.docx')
            
        except Exception as e:
            print(f'处理 {file_path} 时出错: {e}')
        
        finally:
            # 显式删除文档对象
            del doc
            # 强制垃圾回收
            gc.collect()

# 使用生成器处理
def process_documents_generator(file_list):
    """使用生成器逐个处理，减少内存占用"""
    for file_path in file_list:
        doc = Document(file_path)
        yield doc
        del doc
        gc.collect()
```

---

## 8. 文档损坏修复

### 问题描述
打开损坏的 Word 文档时程序崩溃。

### 解决方案

```python
from docx import Document
import zipfile
from lxml import etree

def try_repair_docx(file_path):
    """尝试修复损坏的 Word 文档"""
    
    # 尝试标准方式打开
    try:
        doc = Document(file_path)
        return doc, '成功'
    except Exception as e:
        print(f'标准方式打开失败: {e}')
    
    # 尝试修复 XML 结构
    try:
        # docx 本质是 zip，解压后可以尝试修复 XML
        with zipfile.ZipFile(file_path, 'r') as zip_ref:
            # 检查关键文件是否存在
            required_files = ['word/document.xml', '[Content_Types].xml']
            for req_file in required_files:
                if req_file not in zip_ref.namelist():
                    return None, f'缺少关键文件: {req_file}'
            
            # 读取并解析 XML
            xml_content = zip_ref.read('word/document.xml')
            root = etree.fromstring(xml_content)
            
            return root, 'XML 结构正常'
            
    except Exception as e:
        return None, f'修复失败: {e}'

def extract_text_from_corrupted(file_path):
    """从损坏的文档中提取文本"""
    try:
        with zipfile.ZipFile(file_path, 'r') as zip_ref:
            xml_content = zip_ref.read('word/document.xml')
            
            # 解析 XML
            root = etree.fromstring(xml_content)
            
            # 提取所有文本
            namespaces = {'w': 'http://schemas.openxmlformats.org/wordprocessingml/2006/main'}
            texts = root.xpath('//w:t/text()', namespaces=namespaces)
            
            return ' '.join(texts)
    except Exception as e:
        return f'提取失败: {e}'
```

---

## 9. 兼容旧版 Word (.doc)

### 问题描述
需要兼容 Word 97-2003 格式。

### 解决方案

```python
# python-docx 主要支持 .docx 格式
# 如需支持 .doc，需要使用其他库

# 方案1: 使用 pywin32 (仅 Windows)
try:
    from win32com.client import Dispatch
    import os
    
    def convert_doc_to_docx(doc_path, output_path):
        """使用 Word 将 .doc 转换为 .docx"""
        word = Dispatch('Word.Application')
        word.Visible = False
        
        doc = word.Documents.Open(os.path.abspath(doc_path))
        doc.SaveAs(os.path.abspath(output_path), FileFormat=16)  # 16 = wdFormatXMLDocument
        doc.Close()
        word.Quit()

except ImportError:
    print('pywin32 未安装，Windows 上可用: pip install pywin32')

# 方案2: 使用 aspose.words (跨平台，商业库)
# pip install aspose-words
```

---

## 📋 Word 问题速查表

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 中文乱码 | 未设置 eastAsia 字体 | `run._element.rPr.rFonts.set(qn('w:eastAsia'), '微软雅黑')` |
| 表格跨页断开 | 缺少跨页设置 | 添加 `<w:cantSplit/>` 元素 |
| 合并单元格报错 | 不是矩形区域 | 确保 start_row <= end_row, start_col <= end_col |
| 样式不生效 | 直接修改全局样式 | 新建 run 单独设置 |
| 内存占用高 | 未释放对象 | `del doc; gc.collect()` |
| 打开损坏文档 | XML 结构错误 | 尝试提取 XML 修复 |
| 需要 .doc 兼容 | 格式限制 | 使用 pywin32 或 aspose.words |
