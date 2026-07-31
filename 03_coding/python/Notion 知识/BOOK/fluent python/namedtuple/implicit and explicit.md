---
title: "implicit and explicit"
publish: false
tags: ["Python"]
---
# implicit and explicit

"Iteration is often implicit." 这句话的意思是：在 Python 中，迭代过程通常是隐式的，即不需要明确地编写迭代的代码细节，Python 会在背后自动处理迭代过程。

### Implicit vs Explicit

### Implicit Iteration（隐式迭代）

隐式迭代指的是迭代过程被隐藏在高层次的操作背后，用户不需要直接写出迭代的逻辑。Python 中许多内置函数和结构都支持隐式迭代。例如，列表推导式和生成器表达式都隐式地处理了迭代过程。

**示例：隐式迭代**

```python
# 列表推导式
squares = [x**2 for x in range(10)]
print(squares)  # 输出：[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 内置函数
total = sum([1, 2, 3, 4, 5])
print(total)  # 输出：15

# 使用map函数
result = map(lambda x: x*2, [1, 2, 3, 4])
print(list(result))  # 输出：[2, 4, 6, 8]

```

在这些示例中，迭代过程是隐式的，用户不需要明确地编写循环逻辑，Python 会在背后处理这些细节。

### Explicit Iteration（显式迭代）

显式迭代指的是用户明确地写出迭代的逻辑，通常使用 `for` 或 `while` 循环。显式迭代给了用户对迭代过程的完全控制，可以在循环中执行更复杂的操作。

**示例：显式迭代**

```python
# 使用for循环的显式迭代
squares = []
for x in range(10):
    squares.append(x**2)
print(squares)  # 输出：[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 使用while循环的显式迭代
total = 0
numbers = [1, 2, 3, 4, 5]
i = 0
while i < len(numbers):
    total += numbers[i]
    i += 1
print(total)  # 输出：15

```
