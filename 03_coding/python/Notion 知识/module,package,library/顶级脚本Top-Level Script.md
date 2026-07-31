---
title: "顶级脚本Top-Level Script"
publish: false
tags: ["Python"]
---
# 顶级脚本Top-Level Script

“顶级脚本”（Top-Level Script）是指直接运行的 Python 脚本，而不是作为模块导入到其他脚本中。通常是你通过命令行或 IDE 直接执行的脚本。

### 顶级脚本的特点

1. **直接运行**：顶级脚本是你运行的主要脚本，通常使用命令如 `python script.py` 来执行。这个脚本是 Python 程序的入口点。
2. **`__name__` 等于 `"__main__"`**：在顶级脚本中，特殊变量 `__name__` 的值为 `"__main__"`。这是 Python 判断脚本是否被直接运行的标准。
3. **入口点**：顶级脚本通常包含程序的入口逻辑，比如解析命令行参数、初始化程序、调用主要函数等。

### 例子

```python
# script.py
def main():
    print("Hello, World!")

if __name__ == "__main__":
    main()

```

当你运行 `python script.py` 时：

- `script.py` 是顶级脚本。
- `__name__` 的值为 `"__main__"`，所以 `main()` 函数会被调用。

### 顶级脚本 vs 模块

- **顶级脚本** 是直接运行的 Python 文件，通常是程序的入口点。
- **模块** 是可以被导入到其他 Python 文件中使用的文件。在模块中，`__name__` 的值是模块名，而不是 `"__main__"`。

```python
# module.py
def greet():
    print("Hello from module")

# another_script.py
import module

module.greet()  # 输出 "Hello from module"

```

在 `another_script.py` 中，`module.py` 被作为模块导入，`module.py` 不是顶级脚本。

### 为什么顶级脚本不能使用相对导入

相对导入依赖于当前模块在包中的位置和层次结构。当你直接运行顶级脚本时，它不再属于任何包，因此无法使用相对导入。这是因为相对导入要求脚本是包的一部分，而顶级脚本是独立的，不属于任何包。

如果在顶级脚本中尝试使用相对导入，比如：

```python
from .my_module import something  # 错误：相对导入在顶级脚本中无效

```

这将导致 `ImportError`，因为 Python 无法识别 `.` 的含义。

### 解决办法

如果你需要在顶级脚本中导入模块，应该使用绝对导入。例如：

```python
from section9.my_func import good  # 使用绝对导入

```

这可以确保无论脚本是否作为顶级脚本运行，导入路径都是明确的。
