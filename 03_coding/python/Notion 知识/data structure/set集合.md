---
title: "set集合"
publish: false
tags: ["Python"]
---
# set集合

```python
#生成
set1={1，2，3，4，5，6}
set2=set('abcc')#会自动去除重复
set3=set(['a','b'])#from list
set4=set(('a','b'))#from tuple
set5=set(('a':1,'b':2))#z只会将keys值转换进set
```

```python
#add()等方法
s=set((1,2,3))
s.add(4)
s.remove(3)
```

```python
#比较
drink = {
    'Martini': {'vodka', 'vermouth'},
    'White Russian': {'vodka', 'cream', 'coffee liqueur'},
    'Screwdriver': {'vodka', 'orange juice'},
    'Margarita': {'tequila', 'lime juice', 'triple sec'}
}

for name contents in drink.items():
	if 'vodka' in contents and not ('vermouth' in contents or 'cream' in contents):
		print(name)
		
#使用&运算符简化
for name ,contents in drink.items():
	if 'vodka' in contents & not contents &{'vermouth','cream'}:
		print(name)

```

```python
#集合
a={1,2}
b={2,3}

##积集合
c=a&b
c=a.intersection(b)#注意这里一定要使用返回值，这个intersection函数不会更新a的值
#和集合
c=a|b 
c=a.union(b)
#差集合
c=a-b
c=a.difference(b)
#xor
c=a^b
c=a.symmetric_difference(b)
#subset
a<=b#bool
a.issubset(b)
#superset
a>=b
a.issuperset(b)
```

```python
#frozenset  immutable set
#参数为可迭代对象
frozenset([1,2,3])#list
frozenset(set([1,3,2]))#set
frozenset({1,2,3})#set
frozenset((1,2,3))#tuple
```

在 Python 中，`set` 类型是一个无序集合，不支持通过索引访问元素。这是因为集合的内部实现是基于哈希表，元素在集合中的位置是根据其哈希值计算的，而不是按插入顺序存储。因此，无法使用中括号（即索引）来访问集合中的特定元素。

以下是一些关于集合的重要特性和用法：

1. **无序性**：集合中的元素没有固定顺序，因此不支持通过索引访问。
2. **唯一性**：集合中的元素是唯一的，没有重复元素。
3. **哈希表实现**：集合的底层实现是基于哈希表，因此可以实现快速的成员测试、插入和删除操作。

由于这些特性，集合的设计是为了快速查找和验证成员资格，而不是按位置访问元素。如果需要按位置访问元素，可以将集合转换为列表或其他有序的数据结构。

---

## set comprehension

`results = {c for c in 'abracadabra' if c not in 'abc'}` 将生成字符串集合 `{'r', 'd'}`
