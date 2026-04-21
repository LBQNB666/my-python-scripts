employees = [
    {'id': '001', 'name': '张三', 'department': '技术部', 'position': '工程师'},
    {'id': '002', 'name': '李四', 'department': '市场部', 'position': '经理'},
    {'id': '003', 'name': '王五', 'department': '人事部', 'position': '专员'},
]

create_employee_report(employees, '员工报告.docx')
```

### 案例 2: 批量生成合同文档

```python
def create_contract(name, id_card, phone, address, start_date, end_date, salary):
    """生成劳动合同"""
    doc = Document()
    
    # 标题
    doc.add_heading('劳动合同', level=0)
    
    # 合同内容
    contract_text = f'''
甲方（用人单位）：某某公司
乙方（劳动者）：{name}
身份证号：{id_card}
联系电话：{phone}
地址：{address}

根据《劳动合同法》及相关法律法规，甲乙双方经平等自愿、协商一致，签订本合同：

一、合同期限
    合同期限从 {start_date} 至 {end_date}

二、工作内容
    乙方同意在甲方担任相关职务。

三、工资待遇
    月工资为 {salary} 元。

四、其他条款
    （略）

甲方（盖章）：___________    乙方（签字）：___________
日期：{start_date}
'''
    
    doc.add_paragraph(contract_text)
    filename = f'合同_{name}.docx'
    doc.save(filename)
    print(f'合同已生成: {filename}')

# 使用
create_contract(
    name='张三',
    id_card='110101199001011234',
    phone='13812345678',
    address='北京市朝阳区某某路1号',
    start_date='2026-01-01',
    end_date='2026-12-31',
    salary='10000'
)
```

### 案例 3: 批量合并 Word 文档

```python
from docx import Document
import os

def merge_documents(input_files, output_file):
    """合并多个 Word 文档"""
    merged_doc = Document()
    
    for file_path in input_files:
        if os.path.exists(file_path):
            doc = Document(file_path)
            
            # 添加标题
            merged_doc.add_heading(os.path.basename(file_path), level=1)
            
            # 添加内容
            for element in doc.element.body:
                merged_doc.element.body.append(element)
            
            # 添加分页
            merged_doc.add_page_break()
            print(f'已合并: {file_path}')
    
    merged_doc.save(output_file)
    print(f'合并完成: {output_file}')

# 使用
files = ['文档1.docx', '文档2.docx', '文档3.docx']
merge_documents(files, '合并文档.docx')
```

### 案例 4: 读取 Word 内容

```python
def read_word_content(file_path):
    """读取 Word 文档内容"""
    doc = Document(file_path)
    
    content = []
    for para in doc.paragraphs:
        if para.text.strip():
            content.append({
                'type': 'paragraph',
                'style': para.style.name,
                'text': para.text
            })
    
    # 读取表格
    for table in doc.tables:
        for row in table.rows:
            row_data = [cell.text for cell in row.cells]
            content.append({
                'type': 'table_row',
                'data': row_data
            })
    
    return content

# 使用
content = read_word_content('员工报告.docx')
for item in content:
    print(item)
```

---

## 📦 安装依赖

```bash
pip install python-docx
```

---

## 🔄 更新日志

- **2026-04-21**: 创建 Word 自动化教程
- 文档创建和编辑
- 文本和格式操作
- 表格操作
- 实用案例
