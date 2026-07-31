---
title: "map()"
publish: false
tags: ["Python"]
---
# map()

**`map()` 是 Python 的一个内置函数，用于将一个函数应用到一个或多个可迭代对象（如列表、元组等）的每个元素上，并返回一个迭代器，包含所有函数的返回值。**

### **`map()` 的语法**

```python

**map(function, iterable, ...)**

```

- **`function`：一个函数，用于应用到每个元素。**
- **`iterable`：一个或多个可迭代对象。**

### **`map()` 的返回值**

**`map()` 返回一个迭代器，可以使用 `list()` 或 `tuple()` 将其转换为列表或元组。**

### **使用示例**

```python
**def square(x):
    return x * x

numbers = [1, 2, 3, 4, 5]
squared = map(square, numbers)
print(list(squared))  # 输出 [1, 4, 9, 16, 25]**

```

**使用 lambda 表达式：**

```python
**numbers = [1, 2, 3, 4, 5]
squared = map(lambda x: x * x, numbers)
print(list(squared))  # 输出 [1, 4, 9, 16, 25]**

```

**多个可迭代对象：**

```python
**def add(x, y):
    return x + y

numbers1 = [1, 2, 3]
numbers2 = [4, 5, 6]
summed = map(add, numbers1, numbers2)
print(list(summed))  # 输出 [5, 7, 9]**

```

**字符串操作**

```python
**words = ['apple', 'banana', 'cherry']
capitalized = map(str.upper, words)
print(list(capitalized))  # 输出 ['APPLE', 'BANANA', 'CHERRY']**

```

### **`map()` 与列表解析**

**`map()` 常用于需要对序列的每个元素应用相同操作的情况。列表解析（list comprehension）也可以实现类似的功能，但两者的使用场景有所不同。**

**列表解析：**

```python
**numbers = [1, 2, 3, 4, 5]
squared = [x * x for x in numbers]
print(squared)  # 输出 [1, 4, 9, 16, 25]**

```

**`map()`：**

```python
**numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x * x, numbers))
print(squared)  # 输出 [1, 4, 9, 16, 25]**
```

---

`map(lambda x: x**2, range(10))` 返回的是一个 `map` 对象，这是因为在 Python 3 中，`map()` 函数返回的结果是一个惰性计算的迭代器，而不是像在 Python 2 中那样返回一个列表。

### 惰性计算的迭代器

惰性计算的迭代器意味着元素是在需要的时候（例如遍历它时）才被计算和生成。这种方式具有以下优点：

1. **节省内存**：特别是当输入可迭代对象非常大时，惰性计算避免了一次性加载所有元素到内存中。
2. **提高效率**：元素在需要时才计算，避免了不必要的计算。
