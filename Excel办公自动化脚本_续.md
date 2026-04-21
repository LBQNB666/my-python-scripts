chart2.add_chart(chart2, 'K2')
    
    # 饼图
    chart3 = PieChart()
    chart3.title = '成本占比'
    
    chart3.add_data(data, titles_from_data=True)
    chart3.set_categories(cats)
    ws_chart.add_chart(chart3, 'A20')
    
    wb.save(output_file)
    print(f'图表报表已生成: {output_file}')

# 使用
data = [
    ['1月', 10000, 6000, 4000],
    ['2月', 12000, 7000, 5000],
    ['3月', 11000, 6500, 4500],
    ['4月', 14000, 8000, 6000],
]

generate_chart_report(data)
```

---

## 5. 数据验证与清洗

```python
import pandas as pd
import re

def clean_and_validate_data(input_file, output_file):
    """数据验证与清洗"""
    
    df = pd.read_excel(input_file)
    
    print('=== 数据清洗报告 ===')
    print(f'原始数据: {len(df)} 行')
    
    # 1. 删除重复行
    before = len(df)
    df = df.drop_duplicates()
    print(f'删除重复: {before - len(df)} 行')
    
    # 2. 处理空值
    null_counts = df.isnull().sum()
    print(f'\n空值统计:\n{null_counts[null_counts > 0]}')
    
    # 填充空值
    df['姓名'] = df['姓名'].fillna('未知')
    df['年龄'] = df['年龄'].fillna(df['年龄'].median())
    df['工资'] = df['工资'].fillna(0)
    
    # 3. 数据类型转换
    df['年龄'] = pd.to_numeric(df['年龄'], errors='coerce')
    df['工资'] = pd.to_numeric(df['工资'], errors='coerce')
    
    # 4. 数据格式标准化
    # 手机号格式
    def format_phone(phone):
        if pd.isna(phone):
            return phone
        phone = str(phone).strip()
        if re.match(r'^1[3-9]\d{9}$', phone):
            return f'{phone[:3]}-{phone[3:7]}-{phone[7:]}'
        return phone
    
    if '手机' in df.columns:
        df['手机'] = df['手机'].apply(format_phone)
    
    # 5. 数据范围验证
    if '年龄' in df.columns:
        invalid_age = df[(df['年龄'] < 18) | (df['年龄'] > 100)]
        if len(invalid_age) > 0:
            print(f'\n异常年龄数据: {len(invalid_age)} 行')
            df.loc[(df['年龄'] < 18) | (df['年龄'] > 100), '年龄'] = None
    
    # 6. 保存清洗后的数据
    df.to_excel(output_file, index=False)
    print(f'\n清洗后数据: {len(df)} 行')
    print(f'已保存到: {output_file}')

# 使用
# clean_and_validate_data('脏数据.xlsx', '清洗后.xlsx')
```

---

## 6. 条件格式报表

```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.formatting.rule import ColorScaleRule, DataBarRule, FormulaRule
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

def create_conditional_report(data, output_file='条件格式报表.xlsx'):
    """创建带条件格式的报表"""
    
    wb = Workbook()
    ws = wb.active
    ws.title = '销售报表'
    
    # 写入数据
    headers = ['产品', '销量', '完成率', '评级']
    ws.append(headers)
    
    for row in data:
        ws.append(row)
    
    # 设置表头样式
    header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')
    header_font = Font(bold=True, color='FFFFFF')
    for cell in ws[1]:
        cell.fill = header_fill
        cell.font = header_font
        cell.alignment = Alignment(horizontal='center')
    
    # 条件格式 - 颜色比例尺（完成率列）
    ws.conditional_formatting.add(
        'C2:C10',
        ColorScaleRule(
            start_type='min', start_color='F8696B',
            mid_type='percentile', mid_value=50, mid_color='FFEB84',
            end_type='max', end_color='63BE7B'
        )
    )
    
    # 条件格式 - 数据条（销量列）
    ws.conditional_formatting.add(
        'B2:B10',
        DataBarRule(
            start_type='min', end_type='max',
            bar_color='4472C4'
        )
    )
    
    # 条件格式 - 图标集（评级列）
    # 高评级绿色，低评级红色
    ws.conditional_formatting.add(
        'D2:D10',
        FormulaRule(
            formula=['D2="A"'],
            fill=PatternFill(start_color='63BE7B', end_color='63BE7B', fill_type='solid')
        )
    )
    ws.conditional_formatting.add(
        'D2:D10',
        FormulaRule(
            formula=['D2="C"'],
            fill=PatternFill(start_color='F8696B', end_color='F8696B', fill_type='solid')
        )
    )
    
    # 设置列宽
    ws.column_dimensions['A'].width = 15
    ws.column_dimensions['B'].width = 12
    ws.column_dimensions['C'].width = 12
    ws.column_dimensions['D'].width = 10
    
    wb.save(output_file)
    print(f'条件格式报表已生成: {output_file}')

# 使用
data = [
    ['产品A', 1000, 0.95, 'A'],
    ['产品B', 800, 0.75, 'B'],
    ['产品C', 600, 0.55, 'C'],
    ['产品D', 1200, 1.00, 'A'],
    ['产品E', 400, 0.35, 'C'],
]

create_conditional_report(data)
```

---

## 7. 批量生成工作表

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment
from datetime import datetime, timedelta

def generate_monthly_sheets(year=2026, output_file='月度工作表.xlsx'):
    """生成12个月的工作表"""
    
    wb = Workbook()
    
    # 删除默认工作表
    wb.remove(wb.active)
    
    months = ['1月', '2月', '3月', '4月', '5月', '6月',
              '7月', '8月', '9月', '10月', '11月', '12月']
    
    # 样式
    header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')
    header_font = Font(bold=True, color='FFFFFF')
    header_alignment = Alignment(horizontal='center', vertical='center')
    
    for month in months:
        ws = wb.create_sheet(title=month)
        
        # 标题行
        ws['A1'] = f'{year}年{month}工作记录'
        ws['A1'].font = Font(size=16, bold=True)
        ws.merge_cells('A1:D1')
        
        # 表头
        headers = ['日期', '任务', '完成情况', '备注']
        for col, header in enumerate(headers, start=1):
            cell = ws.cell(row=3, column=col, value=header)
            cell.fill = header_fill
            cell.font = header_font
            cell.alignment = header_alignment
        
        # 设置列宽
        ws.column_dimensions['A'].width = 15
        ws.column_dimensions['B'].width = 30
        ws.column_dimensions['C'].width = 15
        ws.column_dimensions['D'].width = 20
        
        # 填充日期（假设每月30天）
        for day in range(1, 31):
            date_str = f'{year}-{months.index(month)+1:02d}-{day:02d}'
            ws.cell(row=day+3, column=1, value=date_str)
    
    wb.save(output_file)
    print(f'12个月度工作表已生成: {output_file}')

# 使用
generate_monthly_sheets(2026)
```

---

## 8. 数据分类汇总

```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

def classify_and_summarize(input_file, output_file):
    """按分类汇总数据"""
    
    df = pd.read_excel(input_file)
    
    # 按部门分类
    summary = df.groupby