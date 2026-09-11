---
title: Python学习笔记
date: 2026-07-29
categories:
  - 学习笔记
  - Python相关
tags:
  - Python
  - python
---

# 文件相关

## **高级文件操作模块** shutil

### 文件复制

```python
import shutil

# 复制文件内容 + 权限
shutil.copy('src.txt', 'dst.txt')

# 复制文件内容 + 完整元数据（时间戳等）
shutil.copy2('src.txt', 'dst.txt')

# 复制文件到目录
shutil.copy('src.txt', '/backup/')  # 自动放入该目录

# 复制整个目录树
shutil.copytree('src_folder', 'dst_folder')
```

### 文件移动

```python
# 移动文件或目录（类似 mv 命令）
shutil.move('src.txt', '/new/location/')

# 重命名（移动同目录下）
shutil.move('old_name.txt', 'new_name.txt')
```

### 文件删除

```python
# 删除整个目录树（rm -rf）
shutil.rmtree('/tmp/temp_folder')

# 删除单个文件（os.remove 的替代）
# 注意：shutil 没有直接的删除文件函数，通常用 os.remove()
```

### 磁盘空间管理

```python
# 查询磁盘使用情况
total, used, free = shutil.disk_usage('/')
print(f"可用空间: {free // 2**30} GB")
```

### 归档/压缩（打包）

```python
# 创建压缩包
shutil.make_archive('backup', 'zip', '/path/to/folder')
# 支持格式：zip, tar, gztar, bztar, xztar

# 解压缩（需要额外模块）
import zipfile
with zipfile.ZipFile('backup.zip', 'r') as zf:
    zf.extractall('extract_folder')
```

### 文件查找

```python
# 查找可执行文件路径
python_path = shutil.which('python')
# 返回：'/usr/bin/python' 或 None
```

### 为什么用shutil ，而不用 os

| 操作     | os 模块                   | shutil 模块                  |
| :------- | :------------------------ | :--------------------------- |
| 复制文件 | 需要手动 open/read/write  | `shutil.copyfile()` 一行搞定 |
| 复制目录 | 需要递归实现              | `shutil.copytree()` 一行搞定 |
| 删除目录 | `os.rmdir()` 只能删空目录 | `shutil.rmtree()` 递归删除   |
| 移动文件 | `os.rename()` 有限制      | `shutil.move()` 跨设备也支持 |

## 实际应用场景

### 场景1：备份文件

```python
import shutil
from datetime import datetime

# 备份带时间戳
backup_name = f"backup_{datetime.now():%Y%m%d}"
shutil.copytree('/data', f'/backup/{backup_name}')
```

### 场景2： 批量整理文件

```python
import shutil
import os

# 按扩展名分类
for file in os.listdir('.'):
    if file.endswith('.txt'):
        shutil.move(file, 'txt_files/')
    elif file.endswith('.pdf'):
        shutil.move(file, 'pdf_files/')
```

### 场景3：安全删除（先备份）

```python
def safe_delete(folder):
    backup = f"{folder}_backup"
    shutil.copytree(folder, backup) # 可以用 ignore 参数过滤文件
    shutil.rmtree(folder)
```



# 时间相关

