ws['A4'].number_format = 'YYYY/MM/DD HH:MM'  # 2026/04/21 10:30
    
    wb.save('dates.xlsx')

# 读取日期
def read_dates(file_path):
    wb = load_workbook(file_path)
    ws = wb.active
    
    for row in ws.iter_rows(min_row=1, max_row=5):
        for cell in row:
            if isinstance(cell.value, datetime):
                print(f'日期: {cell.value}, 格式: {cell.number_format}')
```

---

## 5. 列名重复和数据混乱

### 问题描述
读写数据时列名重复导致数据错乱。

### 解决方案

```python
import pandas as pd
from openpyxl import load_workbook

def handle_duplicate_columns():
    """处理重复列名"""
    # 读取时自动处理重复列
    df = pd.read_excel('data.xlsx', sheet_name=0)
    
    # pandas 自动将重复列命名为 xxx.1, xxx.2
    print(df.columns.tolist())
    
    # 如果需要自定义列名
    df.columns = ['列1', '列2', '列3', '列1_copy', '列2_copy']
    
    # 去除重复列
    df = df.loc[:, ~df.columns.duplicated()]

def safe_write_with_headers(file_path):
    """安全写入，避免数据覆盖"""
    wb = load_workbook(file_path)
    ws = wb.active
    
    # 检查是否已存在数据
    if ws.max_row > 1:
        # 追加到下一行
        start_row = ws.max_row + 1
    else:
        start_row = 1
    
    # 写入数据
    new_data = [1, 2, 3]
    for col, value in enumerate(new_data, start=1):
        ws.cell(row=start_row, column=col, value=value)
    
    wb.save(file_path)
```

---

## 6. 兼容性问题

### 问题描述
生成的 Excel 在 WPS 或旧版 Excel 中打不开。

### 解决方案

```python
from openpyxl import Workbook
from openpyxl.writer.excel import ExcelWriter

def create_compatible_file():
    """创建兼容性好的文件"""
    wb = Workbook()
    ws = wb.active
    
    # 避免使用新特性
    ws['A1'] = '数据'
    
    # 保存时指定兼容模式
    wb.save('compatible.xlsx')
    
    # 注意：openpyxl 生成的 xlsx 兼容性较好
    # 如果需要更好的兼容性，可以：
    
    # 方案1：使用 xlsxwriter (写入性能更好，兼容性更好)
    # pip install xlsxwriter
    import xlsxwriter
    
    wb2 = xlsxwriter.Workbook('xlsxwriter_output.xlsx')
    ws2 = wb2.add_worksheet()
    
    ws2.write('A1', 'Hello')
    ws2.write(1, 0, 'World')
    
    wb2.close()
    
    # 方案2：确保文件格式为 xlsx（不是 xlsm 等）
    # openpyxl 默认保存为 xlsx

def check_file_compatibility(file_path):
    """检查文件兼容性"""
    from openpyxl import load_workbook
    
    try:
        wb = load_workbook(file_path)
        print(f'文件可正常打开，共 {len(wb.sheetnames)} 个工作表')
        for sheet_name in wb.sheetnames:
            ws = wb[sheet_name]
            print(f'  - {sheet_name}: {ws.max_row} 行 x {ws.max_column} 列')
        return True
    except Exception as e:
        print(f'兼容性警告: {e}')
        return False
```

---

## 7. 条件格式失效

### 问题描述
设置的条件格式在 Excel 中不生效。

### 解决方案

```python
from openpyxl import Workbook
from openpyxl.formatting.rule import Rule, FormulaRule, ColorScaleRule
from openpyxl.styles import PatternFill, Font
from openpyxl.styles.differential import DifferentialStyle

def apply_conditional_formatting_correctly():
    """正确应用条件格式"""
    wb = Workbook()
    ws = wb.active
    
    # 写入数据
    for i in range(1, 20):
        ws[f'A{i}'] = i * 10
    
    # 方案1：基于值的颜色比例
    ws.conditional_formatting.add(
        'A1:A20',
        ColorScaleRule(
            start_type='min', start_color='F8696B',  # 红色
            mid_type='percentile', mid_value=50, mid_color='FFEB84',  # 黄色
            end_type='max', end_color='63BE7B'  # 绿色
        )
    )
    
    # 方案2：基于公式（最灵活）
    # 高亮大于平均值的单元格
    ws.conditional_formatting.add(
        'A1:A20',
        FormulaRule(
            formula=['A1>AVERAGE($A$1:$A$20)'],
            fill=PatternFill(start_color='C6EFCE', end_color='C6EFCE', fill_type='solid'),
            font=Font(color='006100')
        )
    )
    
    # 方案3：使用 Rule 对象
    red_fill = PatternFill(start_color='FFC7CE', end_color='FFC7CE', fill_type='solid')
    red_font = Font(color='9C0006')
    
    rule = Rule(type='cellIs', operator='lessThan', formula=['0'], fill=red_fill, font=red_font)
    ws.conditional_formatting.add('B1:B20', rule)
    
    wb.save('conditional_format.xlsx')

def remove_conditional_formatting(ws, range_string):
    """删除条件格式"""
    if range_string in ws.conditional_formatting:
        del ws.conditional_formatting[range_string]
```

---

## 8. 数据验证失效

### 问题描述
设置的数据验证不生效。

### 解决方案

```python
from openpyxl import Workbook
from openpyxl.worksheet.datavalidation import DataValidation

def apply_validation_correctly():
    """正确应用数据验证"""
    wb = Workbook()
    ws = wb.active
    
    # 1. 下拉列表验证
    dv = DataValidation(
        type='list',
        formula1='"选项1,选项2,选项3"',  # 下拉选项
        allow_blank=True,  # 允许空白
        showDropDown=False  # 显示下拉箭头
    )
    dv.error = '请选择有效选项'
    dv.errorTitle = '输入错误'
    dv.prompt = '请从下拉列表中选择'
    dv.promptTitle = '选择'
    
    ws.add_data_validation(dv)
    dv.add('A1:A10')
    
    # 2. 数值范围验证
    dv2 = DataValidation(
        type='whole',  # 整数
        operator='between',
        formula1='0',
        formula2='100',
        allow_blank=True
    )
    dv2.error = '数值必须在 0-100 之间'
    ws.add_data_validation(dv2)
    dv2.add('B1:B10')
    
    # 3. 日期范围验证
    dv3 = DataValidation(
        type='date',
        operator='between',
        formula1='2026-01-01',
        formula2='2026-12-31'
    )
    ws.add_data_validation(dv3)
    dv3.add('C1:C10')
    
    # 4. 自定义公式验证
    dv4 = DataValidation(
        type='custom',
        formula1='=LEN(A1)=11',  # 手机号长度验证
        allow_blank=True
    )
    ws.add_data_validation(dv4)
    dv4.add('D1:D10')
    
    wb.save('validation.xlsx')

def clear_validation(ws, range_string):
    """清除数据验证"""
    if range_string in ws.data_validations.dataValidation:
        ws.data_validations.dataValidation.remove(range_string)
```

---

## 📋 Excel 问题速查表

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 内存溢出 | 普通模式写入大文件 | 使用 `write_only=True` |
| 公式值为 None | openpyxl 不计算公式 | 用 `data_only=True` 读取已计算的文件 |
| 图表失真 | 数据范围设置错误 | 正确设置 `Reference` 的范围 |
| 日期变数字 | 数字被当作日期序列号 | 设置 `number_format` |
| 条件格式失效 | 公式引用不正确 | 使用 `$` 固定引用 |
| 数据验证失效 | 未正确添加到工作表 | 调用 `ws.add_data_validation()` |
| 兼容性问题 | 使用了新特性 | 使用 xlsxwriter 或确保格式为 xlsx |
| 列名重复 | 重复添加列 | 使用 `df.loc[:, ~df.columns.duplicated()]` |
