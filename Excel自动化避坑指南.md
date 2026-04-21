# ⚠️ Excel 自动化避坑指南

> 真正踩过坑才能总结出的经验

## 🔥 常见问题与解决方案

---

## 1. 超大文件内存溢出

### 问题描述
处理百万行数据时内存占用过高，甚至崩溃。

### 解决方案

```python
from openpyxl import Workbook

def write_large_file_optimized(data, output_file):
    """优化的大文件写入 - 节省内存"""
    
    # 关键：使用 write_only 模式
    wb = Workbook(write_only=True)
    
    # 创建工作表
    ws = wb.create_sheet(title='大数据')
    
    # 逐行写入，不要一次性加载
    for row_data in data:
        ws.append(row_data)
    
    wb.save(output_file)
    print(f'大文件已保存: {output_file}')

# 对比测试
def test_memory():
    import tracemalloc
    
    tracemalloc.start()
    
    # 普通模式 - 100万行约占用 3GB 内存
    # wb = Workbook()
    # ws = wb.active
    # for i in range(1000000):
    #     ws.append([i, i+1, i+2])
    
    # write_only 模式 - 内存占用大幅降低
    wb = Workbook(write_only=True)
    ws = wb.create_sheet()
    for i in range(1000000):
        ws.append([i, i+1, i+2])
    
    wb.save('large_file.xlsx')
    
    current, peak = tracemalloc.get_traced_memory()
    print(f'当前: {current/1024/1024:.1f}MB, 峰值: {peak/1024/1024:.1f}MB')
    tracemalloc.stop()
```

### 读取大文件的优化

```python
from openpyxl import load_workbook

def read_large_file_optimized(file_path):
    """优化的大文件读取"""
    
    # 方式1：只读模式
    wb = load_workbook(file_path, read_only=True)
    ws = wb.active
    
    for row in ws.iter_rows(min_row=1, max_row=1000):
        # 处理每一行
        pass
    
    wb.close()  # 必须关闭释放内存
    
    # 方式2：使用 pandas 分块读取
    import pandas as pd
    
    for chunk in pd.read_csv('data.csv', chunksize=10000):
        # 处理每个块
        process(chunk)
```

---

## 2. 公式计算结果为 None

### 问题描述
使用 openpyxl 写入公式后，读取时值为 None。

### 原因
openpyxl 只能写入公式，**不会计算公式**。Excel 打开时才会计算。

### 解决方案

```python
from openpyxl import load_workbook
from openpyxl.utils import get_column_letter

# 方案1：使用 data_only=True 读取已计算的值
def read_calculated_values(file_path):
    """读取已计算过的值"""
    wb = load_workbook(file_path, data_only=True)  # 关键参数
    ws = wb.active
    
    # 注意：只有之前在 Excel 中保存过的文件才能读到值
    # 新建的公式会是 None
    for row in ws.iter_rows(min_row=1, max_row=10):
        for cell in row:
            print(cell.value)

# 方案2：强制 Excel 重新计算
# 使用 pywin32 (Windows only)
try:
    from win32com.client import Dispatch
    
    def recalculate_excel(file_path):
        """打开 Excel 并重新计算所有公式"""
        excel = Dispatch('Excel.Application')
        excel.Visible = False
        
        wb = excel.Workbooks.Open(file_path)
        wb.Save()  # 保存时会重新计算
        wb.Close()
        excel.Quit()

except ImportError:
    print('需要安装 pywin32: pip install pywin32')

# 方案3：使用 xlwings
# pip install xlwings
import xlwings as xw

def recalculate_with_xlwings(file_path):
    """使用 xlwings 强制计算"""
    wb = xw.Book(file_path)
    wb.app.calculate()  # 重新计算所有公式
    wb.save()
    wb.close()
```

### 写入公式的正确方式

```python
from openpyxl import Workbook

def write_formula_correctly():
    wb = Workbook()
    ws = wb.active
    
    # 写入数据
    ws['A1'] = 100
    ws['A2'] = 200
    
    # 写入公式（必须是字符串格式，以 = 开头）
    ws['A3'] = '=SUM(A1:A2)'
    
    # 批量写入公式
    for row in range(1, 100):
        ws[f'B{row}'] = f'=A{row}*2'
    
    wb.save('formula_test.xlsx')
    print('公式已写入，Excel 打开时会自动计算')

write_formula_correctly()
```

---

## 3. 图表失真问题

### 问题描述
使用 openpyxl 创建的图表在 Excel 中显示异常。

### 解决方案

```python
from openpyxl import Workbook
from openpyxl.chart import BarChart, Reference
from openpyxl.chart.series import DataPoint
from openpyxl.drawing.fill import PatternFillProperties, ColorChoice

def create_chart_correctly():
    """正确创建图表"""
    wb = Workbook()
    ws = wb.active
    
    # 准备数据
    ws['A1'] = '产品'
    ws['B1'] = '销量'
    data = [('A', 100), ('B', 200), ('C', 150)]
    for i, (name, value) in enumerate(data, start=2):
        ws[f'A{i}'] = name
        ws[f'B{i}'] = value
    
    # 创建图表
    chart = BarChart()
    chart.type = 'col'
    chart.style = 10  # 选择预设样式
    chart.title = '销售统计'
    chart.y_axis.title = '销量'
    chart.x_axis.title = '产品'
    
    # 设置数据范围
    data = Reference(ws, min_col=2, min_row=1, max_col=2, max_row=4)
    cats = Reference(ws, min_col=1, min_row=2, max_row=4)
    
    chart.add_data(data, titles_from_data=True)
    chart.set_categories(cats)
    
    # 设置图表大小
    chart.width = 15  # cm
    chart.height = 10  # cm
    
    # 添加到工作表
    ws.add_chart(chart, 'D2')
    
    wb.save('chart_test.xlsx')

def create_chart_with_colors():
    """创建带颜色的图表"""
    wb = Workbook()
    ws = wb.active
    
    # 数据
    for i in range(1, 6):
        ws[f'A{i}'] = f'项目{i}'
        ws[f'B{i}'] = i * 10
    
    chart = BarChart()
    chart.type = 'col'
    
    data = Reference(ws, min_col=2, min_row=1, max_row=5)
    cats = Reference(ws, min_col=1, min_row=2, max_row=5)
    chart.add_data(data, titles_from_data=True)
    chart.set_categories(cats)
    
    # 设置系列颜色
    from openpyxl.chart.series import DataPoint
    from openpyxl.drawing.fill import PatternFillProperties, ColorChoice
    
    # 设置整个系列的颜色
    chart.series[0].graphicalProperties.solidFill = '4472C4'
    
    ws.add_chart(chart, 'D2')
    wb.save('colored_chart.xlsx')
```

---

## 4. 日期格式问题

### 问题描述
写入日期后变成数字，或格式不正确。

### 解决方案

```python
from openpyxl import Workbook
from openpyxl.styles import numbers
from datetime import datetime, date

def handle_dates_correctly():
    """正确处理日期"""
    wb = Workbook()
    ws = wb.active
    
    # 方法1：直接写入 date 对象
    ws['A1'] = date(2026, 4, 21)
    ws['A1'].number_format = 'YYYY-MM-DD'  # 格式化
    
    # 方法2：写入 datetime 对象
    ws['A2'] = datetime(2026, 4, 21, 10, 30, 0)
    ws['A2'].number_format = 'YYYY-MM-DD HH:MM:SS'
    
    # 方法3：写入字符串（确保格式一致）
    ws['A3'] = '2026-04-21'
    ws['A3'].number_format = '@'  # 文本格式
    
    # 方法4：常用日期格式
    ws['A4'] = datetime.now()
    ws['A4'].number_format = 'MM/DD/YYYY'  # 04/21/2026
    ws['A4'].number_format = 'DD-MMM-YYYY'  # 21-Apr-2026
    ws['A4'].number_format = 'YYYY/MM/DD HH:MM'  # 2026/04/