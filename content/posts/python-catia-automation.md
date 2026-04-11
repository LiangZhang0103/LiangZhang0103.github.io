---
title: "用 Python 实现 CATIA 二次开发自动化"
date: 2025-02-10
emoji: "🐍"
tags: ["Python", "CATIA", "自动化"]
draft: false
summary: "基于 CAA/RADE 和 Python 的自动化方案，提升日常设计效率。"
---

## 为什么用 Python 做 CATIA 二次开发？

CATIA 的二次开发有几种方式：

| 方式 | 语言 | 难度 | 灵活度 |
|------|------|------|--------|
| CAA/RADE | C++ | 高 | 极高 |
| CATIA VBA | VBA | 低 | 中 |
| CATIA API + Python | Python | 中 | 高 |

Python 的优势在于：

- 学习曲线平缓，团队上手快
- 丰富的第三方库（pandas、openpyxl、numpy）
- 可以快速集成数据处理、文件管理等功能
- 适合批量处理和自动化场景

## 环境搭建

### 安装依赖

```bash
pip install pywin32 pycatia
```

### 基本连接

```python
from pycatia import CATIA

# 连接到已打开的 CATIA 实例
catia = CATIA()
documents = catia.documents()
print(f"当前打开的文档数: {documents.count}")
```

## 实用案例

### 1. 批量导出零件属性

```python
from pycatia import CATIA

catia = CATIA()
part_document = catia.active_document()
part = part_document.part()

# 获取零件属性
properties = {
    "零件号": part.name,
    "材料": part.density if hasattr(part, 'density') else "N/A",
    "体积": round(part.get_volume(), 2),
    "质量": round(part.get_mass(), 2),
}

for key, value in properties.items():
    print(f"{key}: {value}")
```

### 2. 批量更新标准件库

```python
import os
from pycatia import CATIA

def update_standard_parts(directory, parameter_name, new_value):
    """批量更新标准件的参数值"""
    catia = CATIA()
    updated = 0

    for filename in os.listdir(directory):
        if filename.endswith(".CATPart"):
            filepath = os.path.join(directory, filename)
            catia.documents.open(filepath)
            
            part = catia.active_document().part()
            params = part.parameters
            
            for param in params:
                if param.name.endswith(parameter_name):
                    param.value = new_value
                    updated += 1
            
            part.update()
            catia.active_document().save()
            catia.active_document().close()

    return updated

# 使用示例
count = update_standard_parts("/path/to/parts", "长度", 150.0)
print(f"已更新 {count} 个参数")
```

### 3. BOM 自动提取

```python
import pandas as pd
from pycatia import CATIA

def extract_bom():
    """从 CATIA Product 提取 BOM 信息"""
    catia = CATIA()
    product = catia.active_document().product()
    
    bom_data = []
    
    def walk_product(prod, level=0):
        for child in prod.products:
            bom_data.append({
                "层级": level,
                "零件号": child.name,
                "实例名": child.instance_name,
                "数量": 1,  # 简化处理
            })
            walk_product(child, level + 1)
    
    walk_product(product)
    return pd.DataFrame(bom_data)

# 导出为 Excel
df = extract_bom()
df.to_excel("bom_export.xlsx", index=False)
print(f"已导出 {len(df)} 条 BOM 记录")
```

## 注意事项

1. **COM 接口限制** — Python 通过 COM 接口访问 CATIA，部分高级 API 需要CAA
2. **性能考虑** — 大批量操作建议用批处理模式，避免频繁打开/关闭文档
3. **错误处理** — COM 调用要加入异常处理，避免 CATIA 进程残留
4. **版本兼容** — pywin32 和 pycatia 的版本要匹配 CATIA 版本

---

> 自动化的本质不是替代人，而是让人专注于更有价值的工作。
