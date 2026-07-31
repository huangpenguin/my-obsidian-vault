---
title: "assert"
publish: false
tags: ["Python"]
---
# assert

`assert` 语句是 Python 中用于调试的一种工具，它用于测试某个条件是否为真。如果条件为假，`assert` 语句会抛出一个 `AssertionError`，并可以选择性地附带一条错误消息。

### 基本语法

```python
assert condition, optional_message
```

- **`condition`**: 一个返回布尔值的表达式。如果 `condition` 为 `True`，程序继续运行。如果为 `False`，抛出 `AssertionError`。
- **`optional_message`**: （可选）一个在断言失败时显示的错误消息，通常用于说明失败原因。

### 示例

1. **简单的断言**:
    
    ```python
    x = 10
    assert x > 5  # 条件为真，程序继续运行
    
    ```
    
2. **带错误消息的断言**:
    
    ```python
    x = 3
    assert x > 5, "x 必须大于 5"  # 条件为假，抛出 AssertionError，并显示错误消息
    
    ```
    
    结果：
    
    ```makefile
    AssertionError: x 必须大于 5
    
    ```
    
3. **用于测试函数输出**:
    
    ```python
    def divide(a, b):
        assert b != 0, "除数不能为零"
        return a / b
    
    result = divide(10, 2)  # 正常运行
    result = divide(10, 0)  # 抛出 AssertionError: 除数不能为零
    
    ```
