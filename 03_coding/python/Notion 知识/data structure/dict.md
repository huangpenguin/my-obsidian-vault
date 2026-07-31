---
title: "dict"
publish: false
tags: ["Python"]
---
# dict

## 生成

```python

name={'a':1,'b':2}
##dict()生成
name=dict('a'=1,b=2,c=3)
print(name)
##dict()转换
###所有两要素的序列都可以转成dict
name=dict([['a',1],['b',2]])#两要素的list的list
name=dict([('a',1),('b',2)])#两要素的tuple的list
name=dict((['a',1],['b',2]))#两要素的list的tuple
name=dict(['ab','bc','cd'])#两文字的string的list
```

### [key]要素追加、变更

```python

###注意所有的key都是唯一
name['a']=2
name['d']=4#

#[key]要素取得
###如果取不到的话则会抛出错误,可以使用in关键字或者get方法来规避
a_value=name['a']
##get()
a=name.get('e',5)#第二个参数是取不到时的默认返回值
```

```python
#keys方法返回值是可迭代对象<class 'dict_keys'>
a_keys=name.keys()
type(name.keys())
#如果要使用可以转换成list
a_keys=list(name.keys())
a_values=list(name.values())
a_all_items=list(name.items())
```

```python
# **双乘号运算符
##效果是按照从左到右的顺序浅拷贝(shallow copy)
name1={'a':1,'b':2}
name2={'b':100,'c':3}
new_name={**name1,**name2}

##update()效果也一眼
name1.update(name2)

##pop()函数和在list中表现得一样
name1.pop('b')

##clear()
name1.clear()

##in()
if 'c' in name1:
	print(name1['c'])
```

```python
#拷贝
##copy()也和list一样是采取的浅拷贝
o=name1.copy()#name1变，0也会跟着变

##deepcopy()
import copy
name1={'a':1,'b':2}
name1_copy=copy.deepcopy(name1)
```

```python
#循环处理
name1={'a':1,'b':2}
for i in name:#默认处理keys
	print(i)
for i in name1.keys():
	
for i in name1.values():
	
for i in name1.item():组合使用
	

##两个一起用
for keys,names in name1.item():
	print(keys,names)
```

```python
#内包表記
word='letters'
letter_counts={letter:word.count(letter)for letter in word}
##如果不想取重复的字母可以用集合先处理一遍
letter_counts={letter:word.count(letter)for letter in set(word)}
##带循环判断的
letter_counts={letter:word.count(letter)for letter in set(word) if letter in 'abcet'}
```

---

## dict comprehension

```python
results = {n: n ** 2 for n in range(10)}
print(results)#{0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25, 6: 36, 7: 49, 8: 64, 9: 81}
```

```python
knights = {'gallahad': 'the pure', 'robin': 'the brave'}
for k, v in knights.items():
    print(k, v)

gallahad the pure
robin the brave
```
