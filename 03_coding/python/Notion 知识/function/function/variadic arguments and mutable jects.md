---
title: "variadic arguments and mutable jects"
publish: false
tags: ["Python"]
---
# variadic arguments and mutable jects

### **1. 可变参数（Variadic Arguments）**

在 Python 中，用 `*args` 或 `**kwargs` 定义 **可变数量的参数**。

### **示例**

```python
def example(*args):
    print("Positional arguments:", args)

def example_with_kwargs(**kwargs):
    print("Keyword arguments:", kwargs)

example(1, 2, 3)  # Positional arguments: (1, 2, 3)
example_with_kwargs(name="Alice", age=30)  # Keyword arguments: {'name': 'Alice', 'age': 30}
```

---

### **2. 可变对象（Mutable Objects）**

可变对象是指 **可以直接修改其内容的对象**，如 `list`、`dict`、`set` 等。**它与 `*args` 并没有关系。**

### **示例：传入可变对象**

```python
def modify_list(my_list):
    my_list.append(100)  # 修改传入的列表
    print("Inside function:", my_list)

lst = [1, 2, 3]
modify_list(lst)
print("Outside function:", lst)

```

**输出**

```bash
Inside function: [1, 2, 3, 100]
Outside function: [1, 2, 3, 100]

```

这里 `my_list` 直接传入的是一个 **可变对象 `list`**，不需要 `*` 来标记它。

---

### **区别总结**

| 概念 | 用途 | 示例 |
| --- | --- | --- |
| `*args` | 接收任意数量的位置参数 | `def func(*args)` |
| `**kwargs` | 接收任意数量的关键字参数 | `def func(**kwargs)` |
| 可变对象 | 可以直接修改内容的对象 | `def func(my_list)` |

### **简单记忆**

- `args` 和 `*kwargs` 是函数参数数量的灵活处理方法。
- 可变对象是数据结构的特性，与是否使用  无关。
