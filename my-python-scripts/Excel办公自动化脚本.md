# 📊 Excel 办公自动化脚本集

> 日常办公中常用的 Excel 自动化脚本

## 📦 依赖安装

```bash
pip install pandas openpyxl matplotlib
```

---

## 1. 批量生成工资条

```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

def generate_payroll(employee_data, base_salary=10000, output_file='工资条.xlsx'):
    """批量生成工资条"""
    
    wb = Workbook()
    ws = wb.active
    ws.title = '工资条'
    
    # 样式定义
    header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')
    header_font = Font(name='微软雅黑', bold=True, color='FFFFFF', size=11)
    header_alignment = Alignment(horizontal='center', vertical='center')
    
    thin_border = Border(
        left=Side(style='thin'),
        right=Side(style='thin'),
        top=Side(style='thin'),
        bottom=Side(style='thin')
    )
    
    # 表头
    headers = ['工号', '姓名', '部门', '基本工资', '绩效', '奖金', '扣款', '实发工资']
    for col, header in enumerate(headers, start=1):
        cell = ws.cell(row=1, column=col, value=header)
        cell.font = header_font
        cell.fill = header_fill
        cell.alignment = header_alignment
        cell.border = thin_border
    
    # 数据行
    row = 2
    for emp in employee_data:
        ws.cell(row=row, column=1, value=emp['id']).border = thin_border
        ws.cell(row=row, column=2, value=emp['name']).border = thin_border
        ws.cell(row=row, column=3, value=emp['department']).border = thin_border
        
        # 计算工资
        base = base_salary
        performance = emp.get('performance', 0) * 1000
        bonus = emp.get('bonus', 0) * 500
        deduction = emp.get('deduction', 0) * 100
        actual = base + performance + bonus - deduction
        
        ws.cell(row=row, column=4, value=base).border = thin_border
        ws.cell(row=row, column=5, value=performance).border = thin_border
        ws.cell(row=row, column=6, value=bonus).border = thin_border
        ws.cell(row=row, column=7, value=deduction).border = thin_border
        ws.cell(row=row, column=8, value=actual).border = thin_border
        
        row += 1
    
    # 设置列宽
    col_widths = [10, 10, 12, 12, 10, 10, 10, 12]
    for i, width in enumerate(col_widths, start=1):
        ws.column_dimensions[chr(64+i)].width = width
    
    wb.save(output_file)
    print(f'工资条已生成: {output_file}')

# 使用
employees = [
    {'id': '001', 'name': '张三', 'department': '技术部', 'performance': 0.8, 'bonus': 1, 'deduction': 0},
    {'id': '002', 'name': '李四', 'department': '市场部', 'performance': 0.9, 'bonus': 2, 'deduction': 1},
    {'id': '003', 'name': '王五', 'department': '人事部', 'performance': 0.7, 'bonus': 1, 'deduction': 0},
]

generate_payroll(employees)
```

---

## 2. 数据透视表生成器

```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

def create_pivot_table_report(source_file, output_file):
    """生成数据透视表报告"""
    
    # 读取数据
    df = pd.read_excel(source_file)
    
    # 创建透视表
    pivot = pd.pivot_table(
        df,
        values='销售额',
        index='部门',
        columns='月份',
        aggfunc='sum',
        fill_value=0
    )
    
    # 创建新的工作簿
    wb = Workbook()
    ws = wb.active
    ws.title = '透视报告'
    
    # 样式
    header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')
    header_font = Font(bold=True, color='FFFFFF')
    header_alignment = Alignment(horizontal='center', vertical='center')
    
    # 写入表头
    headers = ['部门'] + list(pivot.columns)
    for col, header in enumerate(headers, start=1):
        cell = ws.cell(row=1, column=col, value=header)
        cell.font = header_font
        cell.fill = header_fill
        cell.alignment = header_alignment
    
    # 写入数据
    for row_idx, (idx, row_data) in enumerate(pivot.iterrows(), start=2):
        ws.cell(row=row_idx, column=1, value=idx)
        for col_idx, col_name in enumerate(pivot.columns, start=2):
            ws.cell(row=row_idx, column=col_idx, value=row_data[col_name])
    
    # 添加汇总行
    last_row = len(pivot) + 2
    ws.cell(row=last_row, column=1, value='合计')
    for col_idx in range(2, len(pivot.columns) + 2):
        col_letter = chr(64 + col_idx)
        ws.cell(row=last_row, column=col_idx, 
                value=f'=SUM({col_letter}2:{col_letter}{last_row-1})')
    
    wb.save(output_file)
    print(f'透视表报告已生成: {output_file}')

# 使用
# create_pivot_table_report('销售数据.xlsx', '透视报告.xlsx')
```

---

## 3. 批量合并 Excel 文件

```python
import pandas as pd
import os
from pathlib import Path

def batch_merge_excel(input_folder, output_file, sheet_name=0):
    """批量合并多个 Excel 文件"""
    
    folder = Path(input_folder)
    excel_files = list(folder.glob('*.xlsx')) + list(folder.glob('*.xls'))
    
    print(f'找到 {len(excel_files)} 个 Excel 文件')
    
    all_data = []
    
    for file_path in excel_files:
        print(f'正在读取: {file_path.name}')
        df = pd.read_excel(file_path, sheet_name=sheet_name)
        df['来源文件'] = file_path.name  # 添加来源标记
        all_data.append(df)
    
    # 合并所有数据
    merged_df = pd.concat(all_data, ignore_index=True)
    
    # 保存
    merged_df.to_excel(output_file, index=False)
    print(f'合并完成: {output_file}')
    print(f'总计 {len(merged_df)} 行数据')

# 使用
batch_merge_excel('Excel文件夹', '合并结果.xlsx')
```

---

## 4. 自动生成图表报表

```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.chart import BarChart, LineChart, PieChart, Reference
from openpyxl.styles import Font, PatternFill, Alignment
from openpyxl.chart.label import DataLabelList

def generate_chart_report(data, output_file='图表报表.xlsx'):
    """生成带图表的数据报表"""
    
    wb = Workbook()
    
    # Sheet 1: 原始数据
    ws_data = wb.active
    ws_data.title = '数据'
    
    # 写入数据
    headers = ['月份', '销售额', '成本', '利润']
    ws_data.append(headers)
    
    for row in data:
        ws_data.append(row)
    
    # Sheet 2: 图表
    ws_chart = wb.create_sheet('图表分析')
    
    # 柱状图
    chart1 = BarChart()
    chart1.title = '月度销售对比'
    chart1.style = 10
    chart1.y_axis.title = '金额'
    chart1.x_axis.title = '月份'
    
    data = Reference(ws_data, min_col=2, min_row=1, max_col=4, max_row=len(data)+1)
    cats = Reference(ws_data, min_col=1, min_row=2, max_row=len(data)+1)
    chart1.add_data(data, titles_from_data=True)
    chart1.set_categories(cats)
    chart1.shape = 4
    ws_chart.add_chart(chart1, 'A2')
    
    # 折线图
    chart2 = LineChart()
    chart2.title = '销售趋势'
    chart2.style = 13
    chart2.y_axis.title = '金额'
    chart2.x_axis.title = '月份'
    
    chart2.add_data(data, titles_from_data=True)
    chart2.set_categories(cats)
    ws_chart.add_chart(chart2, '