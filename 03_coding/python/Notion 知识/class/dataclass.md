---
title: "dataclass"
publish: false
tags: ["Python"]
---
# dataclass

`dataclass` 是 Python 3.7 引入的一个模块，用于简化类的创建，尤其是那些主要用来存储数据的类。`dataclass` 通过自动生成常见的特殊方法（如 `__init__()`、`__repr__()`、`__eq__()` 等）来减少样板代码。

### 使用 `dataclass`

要使用 `dataclass`，首先需要从 `dataclasses` 模块导入 `dataclass` 装饰器。以下是一个基本示例：

```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int

# 使用示例
p = Person("Alice", 30)#位置参数
p = Person( age=30,str="Alice")#关键词参数
print(p)  # 输出: Person(name='Alice', age=30)
print(p.name)  # 输出: Alice
print(p.age)   # 输出: 30

```

### 功能和优势

- **自动生成 `__init__()`**: 自动生成包含所有字段的初始化方法。
- **自动生成 `__repr__()`**: 自动生成包含所有字段的字符串表示方法。
- **自动生成 `__eq__()`**: 自动生成用于比较两个实例是否相等的方法。
- **类型提示**: `dataclass` 强烈依赖类型提示，以便自动生成合适的方法。

### 默认值和默认工厂

你可以为字段提供默认值或默认工厂函数：

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Person:
    name: str
    age: int = 30  # 默认值
    hobbies: List[str] = field(default_factory=list)  # 默认工厂

# 使用示例
p = Person("Alice")
print(p)  # 输出: Person(name='Alice', age=30, hobbies=[])

```

### 不可变数据类

可以通过设置 `frozen=True` 参数，使数据类的实例变得不可变（类似于 `namedtuple`）：

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class ImmutablePerson:
    name: str
    age: int

# 使用示例
p = ImmutablePerson("Alice", 30)
print(p)  # 输出: ImmutablePerson(name='Alice', age=30)
# p.age = 31  # 这行代码会抛出错误：dataclasses.FrozenInstanceError: cannot assign to field 'age'

```

### 其他选项

- **排序**: 通过设置 `order=True` 参数，可以让数据类生成用于排序的方法。
- **排除字段**: 可以使用 `field(init=False)` 排除某些字段，使其不包含在 `__init__` 方法中。

```python
from dataclasses import dataclass, field

@dataclass(order=True)
class Person:
    name: str
    age: int = field(compare=True)
    salary: float = field(compare=False)

# 使用示例
p1 = Person("Alice", 30, 50000)
p2 = Person("Bob", 25, 60000)
print(p1 < p2)  # 输出: False

```

### 总结

`dataclass` 提供了一种简洁的方式来创建只包含数据的类，减少了大量样板代码。通过自动生成常见的特殊方法，并支持默认值、默认工厂、不可变实例和排序功能，`dataclass` 使得定义和使用数据类更加方便和高效。
