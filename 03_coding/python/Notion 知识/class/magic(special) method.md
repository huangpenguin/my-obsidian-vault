---
title: "magic(special) method"
publish: false
tags: ["Python"]
---
# magic(special) method

### `__str__` 方法

- `__str__` 是一个特殊方法（魔术方法），用于定义对象的“可打印”表示。
- 当使用 `print` 函数或 `str()` 函数来转换对象时，Python 会自动调用对象的 `__str__` 方法。
- `__str__` 应该返回一个字符串，这个字符串是对象的可读表示。

### `__str__` 方法的作用

假设你有一个自定义类：

```python
class MyClass:
    def __init__(self, value):
        self.value = value
```

如果你尝试直接打印这个类的实例：

```python

obj = MyClass(10)
print(obj)
```

输出将类似于：

```
<__main__.MyClass object at 0x7f1c8a5d5ac0>

```

这是因为没有定义 `__str__` 方法，默认情况下会调用 `__repr__` 方法，它提供的是一个非用户友好的表示。

### 定义 `__str__` 方法

为了让 `print(obj)` 输出自定义的字符串，你需要定义 `__str__` 方法：

```python
class MyClass:
    def __init__(self, value):
        self.value = value

    def __str__(self):
        return f"MyClass with value: {self.value}"
```

现在再打印对象：

```python
obj = MyClass(10)
print(obj)
```

输出将是：

```
MyClass with value: 10
```

这说明 `print` 函数调用了 `__str__` 方法，并打印了它返回的字符串。

### 为什么 `__str__` 可以直接 `print`

当你定义了 `__str__` 方法后，Python 的 `print` 函数在遇到对象时，会自动调用这个方法以获取对象的字符串表示，然后输出：

```python
class MyClass:
    def __init__(self, value):
        self.value = value

    def __str__(self):
        return f"MyClass with value: {self.value}"

obj = MyClass(10)
print(obj)  # 自动调用 obj.__str__()
```

### `__repr__` 方法

还有一个相关的魔术方法是 `__repr__`，它用于提供对象的官方表示，主要用于调试。通常，`__repr__` 应该返回一个字符串，这个字符串应该尽可能准确和详细，适合在调试器和解释器中使用：

```python
class MyClass:
    def __init__(self, value):
        self.value = value

    def __repr__(self):
        return f"MyClass(value={self.value})"

```

在没有 `__str__` 方法的情况下，`print` 函数会使用 `__repr__` 方法：

```python
obj = MyClass(10)
print(obj)  # 自动调用 obj.__repr__()
```

[magic method/Dunder Methods](../function/magic%20method%20Dunder%20Methods/%E7%B4%A2%E5%BC%95.md)
