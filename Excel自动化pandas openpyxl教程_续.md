thin_border = Border(
        left=Side(style='thin'),
        right=Side(style='thin'),
        top=Side(style='thin'),
        bottom=Side(style='thin')
    )
    
    # 4. 写入表头
    headers = ['月份', '产品', '销售额', '成本', '利润']
    for col, header in enumerate(headers, start=1):
        cell = ws.cell(row=1, column=col, value=header)
        cell.font = header_font
        cell.fill = header_fill
        cell.alignment = header_alignment
        cell.border = thin_border
    
    # 5. 写入数据
    for row_idx, row_data in enumerate(sales_data, start=2):
        ws.cell(row=row_idx, column=1, value=row_data['月份']).border = thin_border
        ws.cell(row=row_idx, column=2, value=row_data['产品']).border = thin_border
        ws.cell(row=row_idx, column=3, value=row_data['销售额']).border = thin_border
        ws.cell(row=row_idx, column=4, value=row_data['成本']).border = thin_border
        ws.cell(row=row_idx, column=5, value=row_data['利润']).border = thin_border
    
    # 6. 设置列宽
    ws.column_dimensions['A'].width = 12
    ws.column_dimensions['B'].width = 15
    ws.column_dimensions['C'].width = 12
    ws.column_dimensions['D'].width = 12
    ws.column_dimensions['E'].width = 12
    
    # 7. 添加汇总行
    last_row = len(sales_data) + 2
    ws.cell(row=last_row, column=1, value='合计')
    ws.cell(row=last_row, column=3, value=f'=SUM(C2:C{last_row-1})')
    ws.cell(row=last_row, column=4, value=f'=SUM(D2:D{last_row-1})')
    ws.cell(row=last_row, column=5, value=f'=SUM(E2:E{last_row-1})')
    
    wb.save(output_file)
    print(f'报表已生成: {output_file}')

# 使用
sales_data = [
    {'月份': '2026-01', '产品': '产品A', '销售额': 10000, '成本': 6000, '利润': 4000},
    {'月份': '2026-02', '产品': '产品A', '销售额': 12000, '成本': 7000, '利润': 5000},
    {'月份': '2026-03', '产品': '产品B', '销售额': 8000, '成本': 4000, '利润': 4000},
]

generate_sales_report(sales_data, '销售报表.xlsx')
```

### 案例 2: 批量处理多个 Excel 文件

```python
import pandas as pd
import os

def batch_process_excel(input_folder, output_folder):
    """批量处理 Excel 文件"""
    
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)
    
    for filename in os.listdir(input_folder):
        if filename.endswith('.xlsx'):
            input_path = os.path.join(input_folder, filename)
            
            # 读取
            df = pd.read_excel(input_path)
            
            # 处理（例如：添加汇总行）
            df['总计'] = df.sum(axis=1)
            
            # 保存
            output_path = os.path.join(output_folder, filename)
            df.to_excel(output_path, index=False)
            
            print(f'已处理: {filename}')

# 使用
batch_process_excel('input/', 'output/')
```

### 案例 3: 多表合并

```python
import pandas as pd

def merge_excel_sheets(file_path, output_file):
    """合并同一个 Excel 的多个工作表"""
    
    # 读取所有工作表
    all_sheets = pd.read_excel(file_path, sheet_name=None)
    
    # 合并所有数据
    merged_df = pd.concat(all_sheets.values(), ignore_index=True)
    
    # 去重
    merged_df = merged_df.drop_duplicates()
    
    # 保存
    merged_df.to_excel(output_file, index=False)
    print(f'已合并 {len(all_sheets)} 个工作表到 {output_file}')

# 使用
merge_excel_sheets('多表.xlsx', '合并后.xlsx')
```

### 案例 4: Excel 数据验证

```python
from openpyxl import Workbook
from openpyxl.worksheet.datavalidation import DataValidation

def create_validation_sheet():
    """创建带数据验证的工作表"""
    
    wb = Workbook()
    ws = wb.active
    
    # 设置表头
    ws['A1'] = '姓名'
    ws['B1'] = '年龄'
    ws['C1'] = '部门'
    ws['D1'] = '评级'
    
    # 数据验证：年龄范围
    age_validation = DataValidation(
        type='whole',
        operator='between',
        formula1='18',
        formula2='65',
        error='年龄必须在18-65之间',
        errorTitle='输入错误'
    )
    ws.add_data_validation(age_validation)
    age_validation.add('B2:B100')
    
    # 数据验证：下拉列表
    dept_validation = DataValidation(
        type='list',
        formula1='"技术部,市场部,人事部,财务部"',
        showDropDown=False
    )
    ws.add_data_validation(dept_validation)
    dept_validation.add('C2:C100')
    
    # 数据验证：评级
    rating_validation = DataValidation(
        type='list',
        formula1='"A,B,C,D"',
        showDropDown=False
    )
    ws.add_data_validation(rating_validation)
    rating_validation.add('D2:D100')
    
    wb.save('验证数据.xlsx')
    print('验证