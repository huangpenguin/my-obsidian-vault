---
title: "glob"
publish: false
tags: ["Python"]
---
# glob

```python
    detected_image = image.copy()
    for pattern_folder in sorted(glob(pattern_folder + '/*')):
        if ('01' or '02') not in pattern_folder:
            continue
        image, detected_image, lines_bbox = replace_pattern(pattern_folder, image, detected_image=detected_image)
        lines.extend(lines_bbox)
```

1. **`glob`** 先匹配 **`pattern_folder`** 下的所有子项。
2. **`sorted`** 对结果按字母顺序排序。
3. 最终返回一个 **有序的路径列表**，供后续循环处理。
