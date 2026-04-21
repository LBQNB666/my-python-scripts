# 🐍 我的 Python 脚本收藏

> 日常使用的 Python 自动化脚本

## 📁 脚本列表

| 脚本名 | 功能 | 难度 |
|--------|------|------|
| file_renamer.py | 批量重命名文件 | ⭐ 基础 |
| folder_organizer.py | 文件夹整理 | ⭐ 基础 |
| backup_script.py | 自动备份 | ⭐⭐ 中等 |

## 🚀 快速开始

### 环境要求
- Python 3.8+
- pip 包管理器

### 安装依赖
```bash
pip install requests openpyxl Pillow
```

---

## 📝 脚本详情

### 1. file_renamer.py - 批量重命名

```python
import os

def rename_files(folder_path, old_text, new_text):
    """批量重命名文件"""
    count = 0
    for filename in os.listdir(folder_path):
        if old_text in filename:
            new_filename = filename.replace(old_text, new_text)
            old_path = os.path.join(folder_path, filename)
            new_path = os.path.join(folder_path, new_filename)
            os.rename(old_path, new_path)
            print(f'重命名: {filename} -> {new_filename}')
            count += 1
    print(f'共重命名 {count} 个文件')

# 使用示例
# rename_files('C:/Users/你的文件夹', '.jpeg', '.jpg')
```

### 2. folder_organizer.py - 文件夹整理

```python
import os
import shutil

def organize_downloads(download_folder):
    """自动整理下载文件夹"""
    file_types = {
        'Images': ['.jpg', '.jpeg', '.png', '.gif', '.bmp'],
        'Documents': ['.pdf', '.doc', '.docx', '.txt', '.xlsx'],
        'Videos': ['.mp4', '.avi', '.mkv', '.mov'],
        'Archives': ['.zip', '.rar', '.7z'],
    }
    
    # 创建子文件夹
    for folder_name in file_types.keys():
        folder_path = os.path.join(download_folder, folder_name)
        if not os.path.exists(folder_path):
            os.makedirs(folder_path)
    
    # 移动文件
    for filename in os.listdir(download_folder):
        file_ext = os.path.splitext(filename)[1].lower()
        for folder_name, extensions in file_types.items():
            if file_ext in extensions:
                src = os.path.join(download_folder, filename)
                dst = os.path.join(download_folder, folder_name, filename)
                shutil.move(src, dst)
                print(f'移动: {filename} -> {folder_name}/')
                break

# 使用示例
# organize_downloads('C:/Users/用户名/Downloads')
```

### 3. backup_script.py - 自动备份

```python
import os
import shutil
from datetime import datetime

def backup_folder(source, destination):
    """备份文件夹"""
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_name = f'backup_{timestamp}'
    backup_path = os.path.join(destination, backup_name)
    shutil.copytree(source, backup_path)
    print(f'备份完成: {source} -> {backup_path}')
    return backup_path

# 使用示例
# backup_folder('C:/重要文件', 'D:/备份盘/备份')
```

---

## 📚 学习笔记

### Python 基础要点

1. **文件操作**
   ```python
   # 读取文件
   with open('file.txt', 'r') as f:
       content = f.read()
   
   # 写入文件
   with open('file.txt', 'w') as f:
       f.write('Hello!')
   ```

2. **文件夹操作**
   ```python
   import os
   os.listdir(path)      # 列出文件
   os.makedirs(path)      # 创建文件夹
   ```

---

## 🔄 更新日志

- **2026-04-21**: 创建 Python 脚本收藏
