# 🤖 Python 自动化脚本入门

> 从零开始学习 Python 自动化

## 📖 什么是自动化脚本？

自动化脚本就是让计算机自动完成重复性的任务，比如：
- 批量重命名文件
- 自动填写表单
- 定时备份数据
- 自动发送邮件

## 🛠️ 常用 Python 自动化库

| 库名 | 用途 | 安装命令 |
|------|------|----------|
| `os` | 文件操作 | 内置无需安装 |
| `shutil` | 文件高级操作 | 内置无需安装 |
| `glob` | 文件模式匹配 | 内置无需安装 |
| `selenium` | 浏览器自动化 | `pip install selenium` |
| `pyautogui` | GUI 自动化 | `pip install pyautogui` |
| `requests` | HTTP 请求 | `pip install requests` |
| `openpyxl` | Excel 操作 | `pip install openpyxl` |

## 📝 实用脚本示例

### 1. 批量重命名文件

```python
import os

# 获取当前目录所有文件
files = os.listdir('.')

for file in files:
    # 检查是否是 .txt 文件
    if file.endswith('.txt'):
        # 生成新文件名
        new_name = file.replace('.txt', '_备份.txt')
        # 重命名
        os.rename(file, new_name)
        print(f'{file} -> {new_name}')
```

### 2. 批量下载文件（简单爬虫）

```python
import requests
import os

def download_file(url, filename):
    """下载文件"""
    response = requests.get(url)
    with open(filename, 'wb') as f:
        f.write(response.content)
    print(f'下载完成: {filename}')
```

### 3. 自动创建文件夹结构

```python
import os

def create_project_structure(project_name):
    """创建项目文件夹结构"""
    folders = [
        f'{project_name}/src',
        f'{project_name}/tests',
        f'{project_name}/docs',
        f'{project_name}/assets',
    ]
    
    for folder in folders:
        os.makedirs(folder, exist_ok=True)
        print(f'创建文件夹: {folder}')
```

### 4. 读取和写入文本文件

```python
# 写入文件
with open('hello.txt', 'w', encoding='utf-8') as f:
    f.write('你好，世界！\n')
    f.write('这是我的第一个自动化脚本')

# 读取文件
with open('hello.txt', 'r', encoding='utf-8') as f:
    content = f.read()
    print(content)
```

### 5. 处理 Excel 文件

```python
import openpyxl

# 创建工作簿
wb = openpyxl.Workbook()
ws = wb.active
ws.title = '学生成绩'

# 添加数据
ws['A1'] = '姓名'
ws['B1'] = '数学'
ws['C1'] = '语文'

ws['A2'] = '张三'
ws['B2'] = 95
ws['C2'] = 88

# 保存
wb.save('成绩单.xlsx')
print('Excel 文件创建成功！')
```

## 🎯 实战练习

### 练习 1：整理下载文件夹
创建一个脚本，自动将下载文件夹中的文件按类型分类到不同子文件夹。

### 练习 2：批量修改文件名
将指定文件夹中所有文件的扩展名从 `.jpeg` 改为 `.jpg`。

### 练习 3：自动备份
每天自动将重要文件复制到备份文件夹。

## 📚 学习资源

- [Python 官方文档](https://docs.python.org/3/)
- [Real Python](https://realpython.com/) - Python 教程网站
- [Automate the Boring Stuff](https://automatetheboringstuff.com/) - 自动化办公经典教材

---

## 🔄 更新日志

- **2026-04-21**: 创建仓库，开始学习 Python 自动化
