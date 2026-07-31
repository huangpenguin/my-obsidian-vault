---
title: "_ _ main _ _"
publish: false
tags: ["Python"]
---
# _ _ main _ _

在 Python 中，`__main__` 是一个特殊的名称，用于标识顶级脚本环境。当一个 Python 文件作为脚本直接运行时，它的 `__name__` 属性被设置为 `"__main__"`。这一特性可以用来区分脚本是直接运行的还是被导入的。

### 使用 `__main__` 判断脚本运行方式

通过在脚本中使用 `if __name__ == "__main__":`，可以确保某些代码只有在脚本直接运行时才会执行，而在该脚本作为模块被导入时不会执行。这种用法对于编写可复用的模块和测试代码非常有用。

### 示例

1. **基本示例：**

```python
# example.py

def main():
    print("This is the main function.")

if __name__ == "__main__":
    main()

```

当你直接运行 `example.py` 时：

```
$ python example.py
This is the main function.

```

但是，当你将 `example.py` 作为模块导入另一个脚本时：

```python
# another_script.py

import example

print("example module imported.")

```

运行 `another_script.py`：

```
$ python another_script.py
example module imported.

```

在这种情况下，`example.py` 中的 `main()` 函数不会被执行。

1. **实际应用示例：**

通常情况下，你可能会在脚本中包含一些函数定义和测试代码，通过 `if __name__ == "__main__":` 来确保测试代码只有在脚本直接运行时才会执行，而在模块被导入时不会干扰其他代码。

```python
# utils.py

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

if __name__ == "__main__":
    # 只有直接运行该脚本时，以下测试代码才会执行
    print("Testing add function:", add(2, 3))  # 输出: 5
    print("Testing subtract function:", subtract(5, 2))  # 输出: 3

```

### 总结

- `if __name__ == "__main__":` 用于检查当前脚本是否是被直接运行。
- 通过这种方式，可以确保测试代码或脚本逻辑只有在直接运行时才会执行，而在作为模块导入时不会执行。
- 这一特性帮助开发者编写更加模块化和可复用的代码，同时方便进行脚本测试。
