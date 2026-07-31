---
title: "absolute path and relative path"
publish: false
tags: ["Python"]
---
# absolute path and relative path

## 1. 绝对路径（Absolute Path）

**特点：**

- 以根目录（如 Linux/Unix 的 `/` 或 Windows 的 `C:\`）开头。
- 无论当前工作目录在哪，都可以唯一定位文件或目录。
- 适用于在不同位置运行代码时确保路径的正确性。

**示例：**

```python
python
复制编辑
from pathlib import Path

# 绝对路径（Linux/Unix）
abs_path = Path("/home/user/test.txt")

# 绝对路径（Windows）
abs_path_win = Path("C:/Users/user/test.txt")

```

**如何判断是否是绝对路径？**

```python
python
复制编辑
print(abs_path.is_absolute())  # True

```

---

## 2. 相对路径（Relative Path）

**特点：**

- 不以根目录开头，而是相对于当前工作目录（`cwd()`）或某个基准路径。
- 依赖执行代码的当前工作目录。
- 适用于项目内部路径引用，便于移植。
