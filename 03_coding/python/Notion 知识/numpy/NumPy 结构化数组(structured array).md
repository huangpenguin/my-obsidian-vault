---
title: "NumPy 结构化数组(structured array)"
publish: false
tags: ["Python"]
---
# NumPy 结构化数组(structured array)

`np.dtype([...])` 定义了一个**结构化数据类型**，类似于数据库表或 pandas DataFrame。

```python
color = np.dtype([("r", np.ubyte, 1),
                  ("g", np.ubyte, 1),
                  ("b", np.ubyte, 1),
                  ("a", np.ubyte, 1)])

rgba_array=np.array([
    (255, 0, 0, 255),       # 不透明红色
    (0, 255, 0, 128),       # 半透明绿色（Alpha=128）
    (0, 0, 255, 0),         # 完全透明蓝色
    (128, 128, 128, 255)    # 不透明灰色
], dtype=color)

rgba_array["r"] #这个元素数组的第一列，即红色通道
```

- `("r", np.ubyte, 1)` 说明：
    - `"r"` 是字段名（列名）
    - `np.ubyte` 是数据类型（无符号 8 位整数，范围 0~255）
    - `1` 表示每个元素在该字段中占 1 个 `np.ubyte`

```python
color = np.dtype([
    ("r", np.ubyte, 1),
    ("g", np.ubyte, 1),
    ("b", np.ubyte, 1),
    ("a", np.ubyte, 1)
])

# 创建一个结构化数组
data = np.array([(255, 0, 0, 255), (0, 255, 0, 255)], dtype=color)

print(data)
# 输出：
# [(255,   0,   0, 255) (  0, 255,   0, 255)]

# 访问单个字段（类似于 DataFrame 取列）
print(data["r"])  # [255   0]
print(data["g"])  # [  0 255]
print(data["b"])  # [  0   0]
print(data["a"])  # [255 255]

# 访问单个元素
print(data[0])  # (255, 0, 0, 255)
print(data[0]["r"])  # 255
```

### **总结**

- `r`、`g`、`b`、`a` 是**列名（字段名）**，可以像字典键一样访问。
- 结构化数组的每一行相当于一个 RGBA 颜色。
- 取 `data["r"]` 可以得到所有行的 `r` 值（即第 1 列）。
- 取 `data[0]` 得到第 1 行，`data[0]["r"]` 取第 1 行的 `r` 值。
