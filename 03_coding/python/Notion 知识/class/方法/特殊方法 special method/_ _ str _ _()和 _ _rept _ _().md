---
title: "_ _ str _ _()和 _ _rept _ _()"
publish: false
tags: ["Python"]
---
# _ _ str _ _()和 _ _rept _ _()

### 特殊方法 `__str__()`

- **定义**: `__str__()` 是一个特殊方法，用于返回对象的“用户友好”的字符串表示。
- **作用**: 它应该返回一个能够描述对象的字符串，这个字符串应该对用户有意义，能够清晰地表达对象的状态。

### 用法

你可以在类中实现 `__str__()` 方法，以自定义对象的字符串表示。例如：

```python
class Person:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

    def __str__(self):
        return f"Person(name={self.name}, age={self.age})"

# 使用示例
p = Person("Alice", 30)
print(p)  # 输出: Person(name=Alice, age=30)

```

### 解释

- **定义类**: 类 `Person` 有两个属性 `name` 和 `age`。
- **实现 `__str__()` 方法**: `__str__()` 方法返回一个描述对象状态的字符串。这个字符串会在 `print(p)` 调用时被输出。
- **输出**: 当 `print(p)` 被调用时，Python 会自动调用 `p.__str__()` 方法，并打印返回的字符串。

### 比较 `__str__()` 和 `__repr__()`

除了 `__str__()`，Python 还有另一个特殊方法 `__repr__()`，用于返回对象的“官方”字符串表示。`__repr__()` 方法的目的是提供一个可以用来重新创建对象的字符串。

- **`__str__()`**: 主要用于用户友好的输出，应该是易读且简洁的。
- **`__repr__()`**: 主要用于调试和开发，应该返回一个尽可能准确地表示对象的字符串，通常是可以用来重新创建对象的表达式。

### 示例对比

```python
class Person:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

    def __str__(self):
        return f"Person(name={self.name}, age={self.age})"

    def __repr__(self):
        return f"Person(name={self.name!r}, age={self.age!r})"

# 使用示例
p = Person("Alice", 30)
print(str(p))  # 使用 __str__(): 输出: Person(name=Alice, age=30)
print(repr(p)) # 使用 __repr__(): 输出: Person(name='Alice', age=30)

```

- **`__str__()`**: 返回的字符串更适合于用户的显示。
- **`__repr__()`**: 返回的字符串包含了更详细的信息，可以用来准确地表示对象状态，通常更适合用于调试。

### 总结

- `__str__()` 用于定义对象的用户友好字符串表示，通常用于打印或显示。
- `__repr__()` 用于定义对象的官方字符串表示，适合调试和开发，通常返回一个可以用来重建对象的字符串。
- 你可以在类中实现这两个方法，以提供对象的不同视图。
