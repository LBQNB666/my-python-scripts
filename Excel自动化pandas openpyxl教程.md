# 📊 Excel 自动化 pandas + openpyxl 完整教程

> 用 Python 操作 Excel 的全面指南

## 📂 目录

1. [安装与基础](#安装与基础)
2. [pandas 数据处理](#pandas-数据处理)
3. [openpyxl 样式设置](#openpyxl-样式设置)
4. [图表操作](#图表操作)
5. [实用案例](#实用案例)

---

## 📦 安装与基础

### 安装库

```bash
pip install pandas openpyxl matplotlib
```

### 导入

```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.chart import BarChart, LineChart, PieChart, Reference
```

---

## 🐼 pandas 数据处理

### 1. 读取 Excel

```python
# 读取单个工作表
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# 读取所有工作表
all_sheets = pd.read_excel('data.xlsx', sheet_name=None)

# 读取指定列
df = pd.read_excel('data.xlsx', usecols=['姓名', '年龄', '工资'])

# 读取指定行
df = pd.read_excel('data.xlsx', skiprows=2, nrows=10)
```

### 2. DataFrame 基本操作

```python
# 查看数据
print(df.head())      # 前5行
print(df.tail())      # 后5行
print(df.info())      # 信息
print(df.describe())  # 统计摘要

# 选择列
names = df['姓名']
selected = df[['姓名', '工资']]

# 选择行
df[df['年龄'] > 25]
df[(df['年龄'] > 25) & (df['工资'] > 5000)]

# 排序
df.sort_values('工资', ascending=False)
df.sort_values(['部门', '工资'], ascending=[True, False])

# 添加列
df['年薪'] = df['工资'] * 12
df['等级'] = df['工资'].apply(lambda x: '高' if x > 10000 else '低')
```

### 3. 数据清洗

```python
# 删除重复行
df.drop_duplicates()

# 删除空值
df.dropna()                    # 删除包含空值的行
df.dropna(how='all')          # 只删除全为空的行
df.fillna(0)                   # 用0填充空值

# 填充空值
df['年龄'].fillna(df['年龄'].mean(), inplace=True)

# 替换值
df['部门'].replace({'销售部': '销售中心', '技术部': '研发中心'}, inplace=True)
```

### 4. 分组聚合

```python
# 按部门分组求平均工资
df.groupby('部门')['工资'].mean()

# 多列分组
df.groupby(['部门', '职位'])['工资'].agg(['mean', 'sum', 'count'])

# 透视表
pd.pivot_table(df, values='工资', index='部门', columns='职位', aggfunc='sum')
```

### 5. 导出 Excel

```python
# 保存到 Excel
df.to_excel('output.xlsx', index=False, sheet_name='数据')

# 保存多个工作表
with pd.ExcelWriter('output.xlsx') as writer:
    df1.to_excel(writer, sheet_name='数据1')
    df2.to_excel(writer, sheet_name='数据2')

# 追加到现有文件
with pd.ExcelWriter('output.xlsx', mode='a', engine='openpyxl') as writer:
    df.to_excel(writer, sheet_name='新数据')
```

---

## 🎨 openpyxl 样式设置

### 1. 创建工作簿

```python
from openpyxl import Workbook

wb = Workbook()
ws = wb.active
ws.title = '数据'
```

### 2. 单元格样式

```python
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

# 设置字体
font = Font(name='微软雅黑', size=12, bold=True, color='FFFFFF')

# 设置背景色
fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')

# 设置对齐
alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)

# 设置边框
thin = Side(style='thin', color='000000')
border = Border(left=thin, right=thin, top=thin, bottom=thin)

# 应用样式
cell = ws['A1']
cell.font = font
cell.fill = fill
cell.alignment = alignment
cell.border = border
```

### 3. 行列设置

```python
# 设置列宽
ws.column_dimensions['A'].width = 15
ws.column_dimensions['B'].width = 12

# 设置行高
ws.row_dimensions[1].height = 30

# 冻结首行
ws.freeze_panes = 'A2'

# 筛选
ws.auto_filter.ref = 'A1:E100'
```

### 4. 合并单元格

```python
# 合并单元格
ws.merge_cells('A1:D1')
ws.merge_cells('A2:A4')

# 取消合并
ws.unmerge_cells('A1:D1')
```

### 5. 条件格式

```python
from openpyxl.formatting.rule import ColorScaleRule, DataBarRule

# 颜色刻度
ws.conditional_formatting.add('B2:B10',
    ColorScaleRule(start_type='min', start_color='FF0000',
                   mid_type='percentile', mid_value=50, mid_color='FFFF00',
                   end_type='max', end_color='00FF00'))

# 数据条
ws.conditional_formatting.add('C2:C10',
    DataBarRule(start_type='min', end_type='max',
                bar_color='4472C4'))
```

---

## 📈 图表操作

### 1. 创建柱状图

```python
from openpyxl.chart import BarChart, Reference

# 准备数据
ws['A1'] = '月份'
ws['B1'] = '销售额'
data = [['1月', 1000], ['2月', 1500], ['3月', 1200], ['4月', 1800]]
for row in data:
    ws.append(row)

# 创建图表
chart = BarChart()
chart.type = 'col'
chart.style = 10
chart.title = '月度销售统计'
chart.y_axis.title = '销售额'
chart.x_axis.title = '月份'

# 设置数据
data = Reference(ws, min_col=2, min_row=1, max_col=2, max_row=5)
cats = Reference(ws, min_col=1, min_row=2, max_row=5)
chart.add_data(data, titles_from_data=True)
chart.set_categories(cats)

# 插入图表
ws.add_chart(chart, 'D2')
```

### 2. 创建折线图

```python
from openpyxl.chart import LineChart, Reference

chart = LineChart()
chart.title = '销售趋势'
chart.style = 13
chart.y_axis.title = '金额'
chart.x_axis.title = '月份'

data = Reference(ws, min_col=2, min_row=1, max_row=6)
cats = Reference(ws, min_col=1, min_row=2, max_row=6)
chart.add_data(data, titles_from_data=True)
chart.set_categories(cats)

ws.add_chart(chart, 'D10')
```

### 3. 创建饼图

```python
from openpyxl.chart import PieChart, Reference

chart = PieChart()
chart.title = '市场份额'

data = Reference(ws, min_col=2, min_row=1, max_row=5)
cats = Reference(ws, min_col=1, min_row=2, max_row=5)
chart.add_data(data, titles_from_data=True)
chart.set_categories(cats)

ws.add_chart(chart, 'F2')
```

---

## 🎯 实用案例

### 案例 1: 生成销售报表

```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils.dataframe import dataframe_to_rows

def generate_sales_report(sales_data, output_file):
    """生成销售报表"""
    
    # 1. 数据处理
    df = pd.DataFrame(sales_data)
    df['月份'] = pd.to_datetime(df['月份'])
    df['月份'] = df['月份'].dt.strftime('%Y-%m')
    
    # 2. 创建工作簿
    wb = Workbook()
    ws = wb.active
    ws.title = '销售数据'
    
    # 3. 样式定义
    header_font = Font(name='微软雅黑', bold=True, color='FFFFFF', size=12)
    header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')
    header_alignment = Alignment(horizontal='center', vertical='center')
    
    thin_border = Border(
        left=Side(style='thin'),
        right=