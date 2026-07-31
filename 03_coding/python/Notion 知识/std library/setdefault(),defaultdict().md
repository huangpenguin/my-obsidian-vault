---
title: "setdefault(),defaultdict()"
publish: false
tags: ["Python"]
---
# setdefault(),defaultdict()

### `setdefault()` 方法

`setdefault()` 是字典对象的一个方法，用于获取指定键的值。如果键不存在于字典中，会将键及其默认值添加到字典中，并返回该默认值。

### 语法

```python
dict.setdefault(key, default=None)

```

- `key`: 字典中的键。
- `default`: 如果键不存在，则插入字典的默认值（默认为 `None`）。

### 示例

```python
d = {'a': 1, 'b': 2}
print(d.setdefault('a', 10))  # 输出: 1，键 'a' 存在，所以返回其值
print(d)  # 输出: {'a': 1, 'b': 2}

print(d.setdefault('c', 30))  # 输出: 30，键 'c' 不存在，所以插入并返回默认值
print(d)  # 输出: {'a': 1, 'b': 2, 'c': 30}

```

### `defaultdict` 类

`defaultdict` 是 `collections` 模块中的一个类，它是 `dict` 的子类。与普通字典不同，当访问的键不存在时，`defaultdict` 会调用一个提供的默认工厂函数来生成键的默认值。

### 语法

```python
from collections import defaultdict

defaultdict(default_factory)

```

- `default_factory`: 一个返回默认值的可调用对象（如函数）。

### 示例

```python

from collections import defaultdict

# 使用 int 作为 default_factory，默认值为 0
d = defaultdict(int)#这里其实相当于是可调用对象int（），默认返回0
d['a'] += 1
print(d['a'])  # 输出: 1
print(d['b'])  # 输出: 0，键 'b' 不存在，所以返回默认值 0

# 使用 list 作为 default_factory，默认值为空列表
d = defaultdict(list)
d['a'].append(1)
print(d['a'])  # 输出: [1]
print(d['b'])  # 输出: []，键 'b' 不存在，所以返回默认值 []

#
#d = defaultdict(list)#默认返回是空列表[]
#d = defaultdict(dict)#默认返回是空字典{}
```

### 区别

1. **功能**:
    - `setdefault()` 方法是在访问或设置字典值时使用的，它不会改变字典的默认行为。
    - `defaultdict` 类改变了字典的行为，使得每当访问不存在的键时，会自动生成一个默认值。
2. **用途**:
    - `setdefault()` 适用于你希望在访问字典时提供默认值，但不改变字典整体行为的场景。
    - `defaultdict` 适用于需要频繁处理默认值的场景，它会简化代码，并且避免了许多显式检查键是否存在的代码。
3. **简洁性**:
    - `defaultdict` 通常会使代码更加简洁和易读，特别是在需要处理复杂默认值的情况下。
    - `setdefault()` 在单次插入时很方便，但在频繁需要检查和设置默认值时，代码会显得冗长。

### 总结

- **`setdefault()`**: 用于在访问字典键时，如果键不存在，则将其插入字典并设置为默认值。
- **`defaultdict`**: 用于创建一个带有默认值行为的字典，当访问不存在的键时，会自动插入默认值。

```python
#另一个defaultdcit的例子
from collections import defaultdict
def no_idea():
	return 'huh?'
a=defaultdict(no_idea)
a['a']='abominable snowman'
a['b']='big ball'
print(a['a'])
print(a['b'])
print(a['c'])#此时没有参数，则会调用no_idea
```

defaultdict的一个用法：免去初始化

对比上下两段代码

```python
from collections import defaultdcit
food_counter=defaultdict(int)
for food in ['spam','egg','spam','spam']:
	food_counter[food]+=1

for food,count in food_counter.iteams():
	print(food,count)
```

```python
#需要初始化
dict_counter={}
for food in ['spam','egg','spam','spam']:
	if not food in dict_counter:
		dict_counter[food]=0
	dict_counter[food]+=1

for food,count in dict_counter.items():
	print(food,count)
```
