# 🐍 Python 自动化脚本实战

> 2026年最新实用的 Python 自动化脚本大全

## 📂 目录

1. [文件自动化管理](#文件自动化管理)
2. [图像批量处理](#图像批量处理)
3. [Excel 办公自动化](#excel-办公自动化)
4. [API 数据处理](#api-数据处理)
5. [网络爬虫基础](#网络爬虫基础)
6. [实用工具脚本](#实用工具脚本)

---

## 📁 文件自动化管理

### 1. 按扩展名自动分类文件

```python
import os
from shutil import move

def sort_files_by_extension(directory_path):
    """
    按文件扩展名自动分类到子文件夹
    """
    # 定义扩展名到文件夹的映射
    file_types = {
        'Images': ['.jpg', '.jpeg', '.png', '.gif', '.bmp', '.webp', '.svg'],
        'Documents': ['.pdf', '.doc', '.docx', '.txt', '.xlsx', '.xls', '.pptx'],
        'Videos': ['.mp4', '.avi', '.mkv', '.mov', '.wmv', '.flv'],
        'Music': ['.mp3', '.wav', '.flac', '.aac', '.ogg', '.m4a'],
        'Archives': ['.zip', '.rar', '.7z', '.tar', '.gz', '.iso'],
        'Code': ['.py', '.js', '.html', '.css', '.java', '.cpp', '.json'],
    }
    
    # 创建子文件夹
    for folder_name in file_types.keys():
        folder_path = os.path.join(directory_path, folder_name)
        if not os.path.exists(folder_path):
            os.makedirs(folder_path)
            print(f'创建文件夹: {folder_name}')
    
    # 移动文件
    for filename in os.listdir(directory_path):
        file_ext = os.path.splitext(filename)[1].lower()
        
        for folder_name, extensions in file_types.items():
            if file_ext in extensions:
                source = os.path.join(directory_path, filename)
                destination = os.path.join(directory_path, folder_name, filename)
                
                # 避免覆盖同名文件
                if os.path.exists(destination):
                    base, ext = os.path.splitext(filename)
                    counter = 1
                    while os.path.exists(destination):
                        filename = f"{base}_{counter}{ext}"
                        destination = os.path.join(directory_path, folder_name, filename)
                        counter += 1
                
                move(source, destination)
                print(f'移动: {filename} -> {folder_name}/')
                break

# 使用示例
# sort_files_by_extension('C:/Users/用户名/Downloads')
```

### 2. 删除空文件夹

```python
import os

def remove_empty_folders(directory_path):
    """
    删除目录中的所有空文件夹
    """
    removed_count = 0
    
    for root, dirs, files in os.walk(directory_path, topdown=False):
        for folder in dirs:
            folder_path = os.path.join(root, folder)
            
            # 检查文件夹是否为空
            if not os.listdir(folder_path):
                os.rmdir(folder_path)
                print(f'删除空文件夹: {folder_path}')
                removed_count += 1
    
    print(f'共删除 {removed_count} 个空文件夹')

# 使用示例
# remove_empty_folders('C:/Users/用户名/Documents')
```

### 3. 批量重命名文件

```python
import os
import re

def batch_rename(directory_path, old_text, new_text, preview=True):
    """
    批量重命名文件
    - old_text: 要替换的文本
    - new_text: 替换后的文本
    - preview: True=预览模式, False=实际重命名
    """
    renamed_count = 0
    
    for filename in os.listdir(directory_path):
        if old_text in filename:
            new_filename = filename.replace(old_text, new_text)
            old_path = os.path.join(directory_path, filename)
            new_path = os.path.join(directory_path, new_filename)
            
            if preview:
                print(f'预览: {filename} -> {new_filename}')
            else:
                os.rename(old_path, new_path)
                print(f'重命名: {filename} -> {new_filename}')
            
            renamed_count += 1
    
    if preview:
        print(f'\n预览完成，共 {renamed_count} 个文件将被重命名')
        print('将 preview=False 传入以实际执行重命名')
    else:
        print(f'\n完成，共重命名 {renamed_count} 个文件')

# 使用示例
# 预览模式
# batch_rename('C:/Users/用户名/Pictures', '.jpeg', '.jpg')
# 实际执行
# batch_rename('C:/Users/用户名/Pictures', '.jpeg', '.jpg', preview=False)
```

### 4. 批量添加前缀/后缀

```python
import os

def add_prefix(directory_path, prefix, extension=None):
    """批量添加文件前缀"""
    count = 0
    for filename in os.listdir(directory_path):
        if extension and not filename.endswith(extension):
            continue
        
        new_filename = prefix + filename
        old_path = os.path.join(directory_path, filename)
        new_path = os.path.join(directory_path, new_filename)
        
        os.rename(old_path, new_path)
        print(f'{filename} -> {new_filename}')
        count += 1
    
    print(f'完成，共处理 {count} 个文件')

def add_suffix(directory_path, suffix, extension=None):
    """批量添加文件后缀"""
    count = 0
    
    for filename in os.listdir(directory_path):
        if extension and not filename.endswith(extension):
            continue
        
        name, ext = os.path.splitext(filename)
        new_filename = name + suffix + ext
        old_path = os.path.join(directory_path, filename)
        new_path = os.path.join(directory_path, new_filename)
        
        os.rename(old_path, new_path)
        print(f'{filename} -> {new_filename}')
        count += 1
    
    print(f'完成，共处理 {count} 个文件')

# 使用示例
# add_prefix('C:/Users/用户名/Pictures', '2026_')
# add_suffix('C:/Users/用户名/Pictures', '_backup', '.jpg')
```

---

## 🖼️ 图像批量处理

### 1. 安装依赖

```bash
pip install Pillow opencv-python
```

### 2. 批量调整图像大小

```python
from PIL import Image
import os

def resize_images(input_folder, output_folder, width, height):
    """
    批量调整图像大小
    """
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)
    
    count = 0
    for filename in os.listdir(input_folder):
        if filename.lower().endswith(('.png', '.jpg', '.jpeg', '.bmp', '.gif')):
            input_path = os.path.join(input_folder, filename)
            output_path = os.path.join(output_folder, filename)
            
            with Image.open(input_path) as img:
                # 保持纵横比调整大小
                img.thumbnail((width, height), Image.Resampling.LANCZOS)
                img.save(output_path)
                print(f'调整大小: {filename} -> {width}x{height}')
                count += 1
    
    print(f'完成，共处理 {count} 张图片')

# 使用示例
# resize_images('C:/input', 'C:/output', 800, 600)
```

### 3. 批量添加水印

```python
from PIL import Image, ImageDraw, ImageFont

def add_watermark(input_folder, output_folder, text, position='bottom-right'):
    """
    批量添加文字水印
    """
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)
    
    count = 0
    for filename in os.listdir(input_folder):
        if filename.lower().endswith(('.png', '.jpg', '.jpeg')):
            input_path = os.path.join(input_folder, filename)
            output_path = os.path.join(output_folder, filename)
            
            img = Image.open(input_path)
            draw = ImageDraw.Draw(img)
            
            # 设置字体大小（根据图片大小自动调整）
            font_size = max(int(img.width / 30), 20)
            
            try:
                font = ImageFont.truetype("arial.ttf", font_size)
            except:
                font = ImageFont.load_default()
            
            # 计算文字位置
            bbox = draw.textbbox((0, 0), text, font=font)
            text_width = bbox[2] - bbox[0]
            text_height = bbox[3] - bbox[1]
            
            if position == 'bottom-right':
                x = img.width - text_width - 20
                y = img.height - text_height - 20
            elif position == 'bottom-left':
                x = 20
                y = img.height - text_height - 20
            elif position == 'elif position == 'top-left':
                x = 20
                y = 20
            else:  # top-right
                x = img.width - text_width - 20
                y = 20
            
            # 添加水印
            draw.text((x, y), text, fill=(255, 255, 255, 128), font=font)
            img.save(output_path)
            print(f'添加水印: {filename}')
            count += 1
    
    print(f'完成，共处理 {count} 张图片')

# 使用示例
# add_watermark('C:/input', 'C:/output', '© 我的网站', position='bottom-right')
```

### 4. 批量转换格式

```python
from PIL import Image
import os

def convert_image_format(input_folder, output_folder, output_format='PNG'):
    """
    批量转换图像格式
    """
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)
    
    format_map = {
        'PNG': '.png',
        'JPEG': '.jpg',
        'BMP': '.bmp',
        'GIF': '.gif',
        'WEBP': '.webp'
    }
    
    ext = format_map.get(output_format.upper(), '.png')
    count = 0
    
    for filename in os.listdir(input_folder):
        if filename.lower().endswith(('.png', '.jpg', '.jpeg', '.bmp', '.gif')):
            input_path = os.path.join(input_folder, filename)
            output_name = os.path.splitext(filename)[0] + ext
            output_path = os.path.join(output_folder, output_name)
            
            with Image.open(input_path) as img:
                # JPEG 需要转换 RGB
                if output_format == 'JPEG' and img.mode in ('RGBA', 'P'):
                    img = img.convert('RGB')
                img.save(output_path, format=output_format)
                print(f'转换格式: {filename} -> {output_name}')
                count += 1
    
    print(f'完成，共处理 {count} 张图片')

# 使用示例
# convert_image_format('C:/input', 'C:/output', 'JPEG')
```

---

## 📊 Excel 办公自动化

### 1. 安装依赖

```bash
pip install openpyxl pandas
```

### 2. 读取 Excel 数据

```python
import openpyxl
from openpyxl import Workbook

def read_excel(file_path, sheet_name=0):
    """
    读取 Excel 文件
    """
    wb = openpyxl.load_workbook(file_path)
    
    # 获取工作表
    if isinstance(sheet_name, int):
        sheet = wb.worksheets[sheet_name]
    else:
        sheet = wb[sheet_name]
    
    # 读取数据
    data = []
    for row in sheet.iter_rows(values_only=True):
        data.append(row)
    
    print(f'读取 {len(data)} 行数据')
    return data

# 使用示例
# data = read_excel('C:/data.xlsx', 'Sheet1')
# for row in data[:5]:
#     print(row)
```

### 3. 写入 Excel 数据

```python
import openpyxl
from openpyxl import Workbook

def write_to_excel(file_path, data, sheet_name='Sheet1'):
    """
    写入数据到 Excel
    """
    wb = Workbook()
    sheet = wb.active
    sheet.title = sheet_name
    
    for row in data:
        sheet.append(row)
    
    wb.save(file_path)
    print(f'已保存 {len(data)} 行数据到 {file_path}')

# 使用示例
# data = [
#     ['姓名', '年龄', '城市'],
#     ['张三', 25, '北京'],
#     ['李四', 30, '上海'],
#     ['王五', 28, '广州']
# ]
# write_to_excel('C:/output.xlsx', data)
```

### 4. 批量处理 Excel

```python
import openpyxl
from openpyxl.styles import Font, PatternFill, Alignment

def process_excel_data(input_file, output_file):
    """
    处理 Excel 数据：添加公式、格式化
    """
    wb = openpyxl.load_workbook(input_file)
    sheet = wb.active
    
    # 添加表头样式
    header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')
    header_font = Font(color='FFFFFF', bold=True)
    
    for cell in sheet[1]:
        cell.fill = header_fill
        cell.font = header_font
        cell.alignment = Alignment(horizontal='center')
    
    # 假设第二列是工资，第三列是奖金，添加合计列
    sheet['D1'] = '合计'
    sheet['D1'].font = header_font
    sheet['D1'].fill = header_fill
    
    for row in range(2, sheet.max_row + 1):
        # 添加公式
        sheet[f'D{row}'] = f'=B{row}+C{row}'
    
    wb.save(output_file)
    print(f'处理完成，保存到 {output_file}')

# 使用示例
# process_excel_data('C:/input.xlsx', 'C:/output.xlsx')
```

---

## 🌐 API 数据处理

### 1. 安装依赖

```bash
pip install requests
```

### 2. GET 请求

```python
import requests
import json

def get_request(url, params=None, headers=None):
    """
    发送 GET 请求
    """
    try:
        response = requests.get(url, params=params, headers=headers, timeout=10)
        response.raise_for_status()  # 检查 HTTP 错误
        
        # 自动解析 JSON
        if response.headers.get('content-type', '').startswith('application/json'):
            return response.json()
        else:
            return response.text
    
    except requests.exceptions.RequestException as e:
        print(f'请求错误: {e}')
        return None

# 使用示例
# url = 'https://api.github.com/users/LBQNB666'
# data = get_request(url)
# print(data)
```

### 3. POST 请求

```python
import requests
import json

def post_request(url, data=None, json_data=None, headers=None):
    """
    发送 POST 请求
    """
    try:
        response = requests.post(
            url, 
            data=data, 
            json=json_data, 
            headers=headers, 
            timeout=10
        )
        response.raise_for_status()
        
        if response.headers.get('content-type', '').startswith('application/json'):
            return response.json()
        else:
            return response.text
    
    except requests.exceptions.RequestException as e:
        print(f'请求错误: {e}')
        return None

# 使用示例
# url = 'https://api.example.com/login'
# data = {'username': 'user', 'password': 'pass'}
# result = post_request(url, json_data=data)
```

### 4. 格式化输出 JSON

```python
import json

def pretty_print_json(data, indent=2):
    """美化打印 JSON 数据"""
    print(json.dumps(data, indent=indent, ensure_ascii=False))

# 使用示例
# data = {'name': '张三', 'age': 25}
# pretty_print_json(data)
```

---

## 🕷️ 网络爬虫基础

### 1. 安装依赖

```bash
pip install requests beautifulsoup4
```

### 2. 基础爬虫

```python
import requests
from bs4 import BeautifulSoup

def basic_crawler(url):
    """
    基础爬虫：获取网页内容
    """
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    }
    
    try:
        response = requests.get(url, headers=headers, timeout=10)
        response.raise_for_status()
        
        # 解析 HTML
        soup = BeautifulSoup(response.text, 'html.parser')
        
        return {
            'title': soup.title.string if soup.title else '',
            'text': soup.get_text()[:500],  # 前500字符
            'links': [a.get('href') for a in soup.find_all('a') if a.get('href')]
        }
    
    except Exception as e:
        print(f'爬取错误: {e}')
        return None

# 使用示例
# result = basic_crawler('https://github.com')
# if result:
#     print(f'标题: {result["title"]}')
#     print(f'链接数: {len(result["links"])}')
```

### 3. 爬取图片

```python
import requests
import os
from bs4 import BeautifulSoup

def download_images(url, save_folder):
    """
    下载网页中的所有图片
    """
    if not os.path.exists(save_folder):
        os.makedirs(save_folder)
    
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    }
    
    response = requests.get(url, headers=headers, timeout=10)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    img_tagsimg_tags = soup.find_all('img')
    print(f'找到 {len(img_tags)} 张图片')
    
    for i, img in enumerate(img_tags):
        img_url = img.get('src') or img.get('data-src')
        
        if not img_url:
            continue
        
        # 处理相对路径
        if img_url.startswith('//'):
            img_url = 'https:' + img_url
        elif img_url.startswith('/'):
            from urllib.parse import urljoin
            img_url = urljoin(url, img_url)
        
        try:
            img_response = requests.get(img_url, headers=headers, timeout=10)
            img_response.raise_for_status()
            
            # 获取图片扩展名
            ext = os.path.splitext(img_url.split('?')[0])[1] or '.jpg'
            if ext not in ['.jpg', '.jpeg', '.png', '.gif', '.bmp', '.webp']:
                ext = '.jpg'
            
            filename = f'image_{i+1}{ext}'
            filepath = os.path.join(save_folder, filename)
            
            with open(filepath, 'wb') as f:
                f.write(img_response.content)
            
            print(f'下载: {filename}')
        
        except Exception as e:
            print(f'下载失败 {img_url}: {e}')

# 使用示例
# download_images('https://example.com', 'C:/images')
```

---

## 🛠️ 实用工具脚本

### 1. 文件相似度检查

```python
from difflib import SequenceMatcher

def check_file_similarity(file1, file2):
    """
    检查两个文件的相似度
    """
    with open(file1, 'r', encoding='utf-8') as f:
        file1_data = f.read()
    
    with open(file2, 'r', encoding='utf-8') as f:
        file2_data = f.read()
    
    similarity = SequenceMatcher(None, file1_data, file2_data).ratio()
    print(f'文件相似度: {similarity * 100:.2f}%')
    return similarity

# 使用示例
# check_file_similarity('file1.txt', 'file2.txt')
```

### 2. 批量下载文件

```python
import requests
import os
from urllib.parse import urljoin

def batch_download(urls, save_folder):
    """
    批量下载文件
    """
    if not os.path.exists(save_folder):
        os.makedirs(save_folder)
    
    for i, url in enumerate(urls):
        try:
            filename = os.path.basename(url.split('?')[0]) or f'file_{i+1}'
            filepath = os.path.join(save_folder, filename)
            
            response = requests.get(url, timeout=30, stream=True)
            response.raise_for_status()
            
            with open(filepath, 'wb') as f:
                for chunk in response.iter_content(chunk_size=8192):
                    f.write(chunk)
            
            print(f'下载完成: {filename}')
        
        except Exception as e:
            print(f'下载失败: {url} - {e}')

# 使用示例
# urls = ['https://example.com/file1.jpg', 'https://example.com/file2.jpg']
# batch_download(urls, 'C:/downloads')
```

### 3. 自动备份文件夹

```python
import os
import shutil
from datetime import datetime

def backup_folder(source, destination):
    """
    备份文件夹（带时间戳）
    """
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_name = f'backup_{timestamp}'
    backup_path = os.path.join(destination, backup_name)
    
    shutil.copytree(source, backup_path)
    print(f'备份完成: {source} -> {backup_path}')
    return backup_path

def cleanup_old_backups(backup_folder_path, keep_count=5):
    """
    清理旧备份，只保留最近的 N 个
    """
    backups = [d for d in os.listdir(backup_folder_path) 
               if d.startswith('backup_')]
    backups.sort(reverse=True)
    
    removed = 0
    for old_backup in backups[keep_count:]:
        old_path = os.path.join(backup_folder_path, old_backup)
        shutil.rmtree(old_path)
        print(f'删除旧备份: {old_backup}')
        removed += 1
    
    print(f'共删除 {removed} 个旧备份')

# 使用示例
# backup_folder('C:/重要文件', 'D:/备份盘')
# cleanup_old_backups('D:/备份盘', keep_count=3)
```

### 4. 定时任务（Windows）

```python
import schedule
import time
import os

def job():
    """定时执行的任务"""
    print(f'任务执行时间: {time.strftime("%Y-%m-%d %H:%M:%S")}')
    # 在这里添加你的任务代码
    # 例如: sort_files_by_extension('C:/Downloads')

def schedule_daily_task(hour=18, minute=0):
    """
    设置每日定时任务
    """
    schedule.every().day.at(f'{hour:02d}:{minute:02d}').do(job)
    
    print(f'定时任务已设置: 每天 {hour:02d}:{minute:02d} 执行')
    
    while True:
        schedule.run_pending()
        time.sleep(60)

# 使用示例
# schedule_daily_task(hour=18, minute=30)  # 每天 18:30 执行
```

---

## 📦 常用依赖安装

```bash
# 文件和图像处理
pip install Pillow opencv-python

# Excel 处理
pip install openpyxl pandas

# 网络请求
pip install requests

# 爬虫
pip install beautifulsoup4 lxml

# 定时任务
pip install schedule
```

---

## 🔄 更新日志

- **2026-04-21**: 创建 Python 自动化脚本