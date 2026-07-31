---
title: "or运算符"
publish: false
tags: ["Python"]
---
# or运算符

`or` 运算符在 Python 中会返回第一个非空的值。因此，`("A" or "E") in image_file.stem` 实际上等效于：

```python
"A" in image_file.stem
```

只能写成如下的代码

```python
if ("A" in image_file.stem or "E" in image_file.stem):
    # 执行逻辑

```
