---
title: "enumerate"
publish: false
tags: ["Python"]
---
# enumerate

在序列中循环时，用 [`enumerate()`](https://docs.python.org/zh-cn/3.12/library/functions.html#enumerate) 函数可以同时取出位置索引和对应的值：

```python
**>>> for** i, v **in** enumerate(['tic', 'tac', 'toe']):
**...**     print(i, v)
**...**0 tic
1 tac
2 toe
```

返回的是一个迭代器，一个带序号的迭代器

下面两段代码实质等价

```python
seasons = ['Spring', 'Summer', 'Fall', 'Winter']
list(enumerate(seasons))
[(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
list(enumerate(seasons, start=1))
[(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]
```

```python
def enumerate(iterable, start=0):
    n = start
    for elem in iterable:
        yield n, elem
        n += 1
```

其他循环技巧

[https://docs.python.org/zh-cn/3.12/tutorial/datastructures.html#looping-techniques](https://docs.python.org/zh-cn/3.12/tutorial/datastructures.html#looping-techniques)
