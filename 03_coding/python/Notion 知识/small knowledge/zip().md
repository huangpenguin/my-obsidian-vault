---
title: "zip()"
publish: false
tags: ["Python"]
---
# zip()

[`zip()`](https://docs.python.org/zh-cn/3.12/library/functions.html#zip) 是 Python 中的一个内置函数，它用于将多个可迭代对象（如列表、元组、字符串等）中的元素打包成一个个元组，返回一个 `zip` 对象。这个 `zip` 对象是一个迭代器，可以通过 `list()` 函数转换为一个列表。不妨换一种方式认识 [`zip()`](https://docs.python.org/zh-cn/3.12/library/functions.html#zip) ：它会把行变成列，把列变成行。这类似于 [矩阵转置](https://en.wikipedia.org/wiki/Transpose) 。

### `zip()` 的作用

`zip()` 函数主要用于将多个可迭代对象中的元素按索引位置一一配对。

### `zip()` 的返回值

`zip()` 返回一个 `zip` 对象。每个元素是一个元组，包含输入的可迭代对象中对应位置的元素。

### 用法示例

**将两个列表配对：**

```python

list1 = [1, 2, 3]
list2 = ['a', 'b', 'c']
zipped = zip(list1, list2)
print(list(zipped))#[(1, 'a'), (2, 'b'), (3, 'c')]

```

```python

list1 = [1, 2, 3, 4]
list2 = ['a', 'b', 'c']
zipped = zip(list1, list2)
print(list(zipped))#[(1, 'a'), (2, 'b'), (3, 'c')]

```

如果可迭代对象的长度不一致，`zip()` 会以最短的可迭代对象为准，配对到最短对象的末尾为止。

**使用 `zip` 对象进行迭代：**

```python

list1 = [1, 2, 3]
list2 = ['a', 'b', 'c']
zipped = zip(list1, list2)
for item in zipped:
    print(item)

```

输出：

```arduino
arduino复制代码
(1, 'a')
(2, 'b')
(3, 'c')

```

**解压缩已配对的列表：**

```python
zipped = [(1, 'a'), (2, 'b'), (3, 'c')]
list1, list2 = zip(*zipped)
print(list1)
print(list2)

```

输出：

```arduino
arduino复制代码
(1, 2, 3)
('a', 'b', 'c')

```

### 结论

`zip()` 是一个强大的工具，特别适用于需要将多个序列的数据配对处理的情况。通过了解 `zip()` 的返回值和用法，可以更高效地进行数据处理和操作。
