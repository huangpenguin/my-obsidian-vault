---
title: "list"
publish: false
tags: ["Python"]
---
# list

## 生成

```python

l1=['pig',{'gen':'boy'},'Feb.3']:中括号
l2=list('abc')
l3=list('a','b','c')
l4=list(range(10))

#语法糖   内包表記
a_list=[x**2 for x in range(10)]
#还可以加上判断
a_list=[x**2 for x in range(10) if x%2!=0]
#二维形式的内包表記
'''
tips
注意这里是中括号，如果是小括号则是generator内包表记

'''
cells=[(row,col)for row in rows for col in cols]#内包表记的结果为（row，col）形式的tuple，再存入cells
#tuple unpack 不允许在 for 循环中使用括号来解包元组。相反，可以直接在 for 循环中使用逗号来解包元组
for row，col in cells:
	print（row，col）
```

## 切片

```python
abc=['a','b','c']
a1=abc[0]
a2=abc[::,-2]

abc.reverse()#破坏性地逆转
abc=['a','b','c']
print(abc[-6:-3])#等价于abc[-3:-3]

#用slice插入时，右边哪怕大于左边也可以插入
a=[1,2,3,4]
a[1:3]=[4,3,2,1,0]#也就是说1和3决定的只是引用的start和end
print(a)

#iterator
a[1:3]='abcdefghijk?'
```

## 增删查改

```python
#append()
a1.append('a')

#insert()
a1=[1,23,3,4,5,6]
a1.insert(0,2)#前面的是插入位置offset，后面的是插入的对象

#extend();+=操作符
a1=['a','b']
a2=['c','d']
a1.extend(a2)#在后面一个一个追加
a1+=a2#同上
a1.append(a2)#将a2这个list视为一个整体，插到最后一个位置上
```

```python
#del()  Python语法，删除指定索引上的值
#类似与赋值号=的反向操作，将python对象从这个名字上面剥夺出来，如果对象失去了最后一个参照，则释放这个对象的内存
del a[1]

#remove list方法，取出指定的第一个出现的元素
a.remove('a')
```

```python
#pop()方法默认是取出最后一个索引的对象
a.pop()
a.pop(0)#弹出首位
```

```python
#index()返回第一个出现该值的索引
a_index=a.index('a')
#如果不存在会报错所以最好结合in()
a=['a','b','c']
if ('i'in abc):
    a_index=a.index('i')
    print(a_index)
    
'''
list和tuple的in的探索时间是O(n)但是集合和字典则是O(C)
'''    
```

```python
#经典的join()和split()
friends=['a','b','c']
separator=','
joined= separator.join(friends)
print(joined)
separated=joined.split(separator)
print(separated)
```

```python
#排序
#sort()是list函数，破坏性的
#sorted()是組み込み関数，生成新的排序后的列表
a.sort(reverse=true)#使用关键字reverse可以降序
a1=sorted(a)

#类似的組み込み関数还有len()
a_length=len(a)
```

## 拷贝相关

```python
'''
Python 中的简单赋值绝不会复制数据。 
当你将一个列表赋值给一个变量时，该变量将引用 现有的列表。
你通过一个变量对列表所做的任何更改都会被引用它的所有其他变量看到。
'''
a=[1,2,3]
b=a
a[0]=8#b也会跟着变

'''
切片操作返回包含请求元素的新列表。以下切片操作会返回列表的浅拷贝
拷贝了第一层1，2，【8，9】
但是【8，9】本身作为一个可迭代对象的性质也保留了下来，
也就是说在对浅拷贝后的【8，9】的修改也会对原对象造成修改
在面对迭代的mutable对象时还是会跟着a变化
'''
a=[1,2,[8,9]]
b=a.copy()
c=list(a)
d=a[:]#切片

a[2][1]=1000
a[1]=100
print(f"{a=},\n{b=},\n{c=},\n{d=},")

'''
<深拷贝>真正做到仅拷贝，不会跟着拷贝前对象变动
'''
import copy 
a=[1,2,[8,9]]
b=copy.deepcopy(a)
```

## zip(list,list)

```python

days=[1,2,3]
fruits=['a','b']
for day, fruit in zip(days,fruits):#可以便捷地打印出相同offset上的内容，但是在第一个list停下来的地方就会停下来
		print(day,":eat ")

#zip()处理后得到的是可迭代的值，可以转换成list或者字典
list(zip(days,fruits))
dict(zip(days,fruits))

```

## list comprehension

```python
result = ['{:#04x}'.format(x) for x in range(256) if x % 2 == 0]
```

## range()与list

[`range()`](https://docs.python.org/zh-cn/3.12/library/stdtypes.html#range) 返回的对象在很多方面和列表的行为一样，但其实它和列表不一样。该对象只有在被迭代时才一个一个地返回所期望的列表项，并没有真正生成过一个含有全部项的列表，从而节省了空间。
这种对象称为可迭代对象 [iterable](https://docs.python.org/zh-cn/3.12/glossary.html#term-iterable)，适合作为需要获取一系列值的函数或程序构件的参数。[`for`](https://docs.python.org/zh-cn/3.12/reference/compound_stmts.html#for) 语句就是这样的程序构件；以可迭代对象作为参数的函数例如 [`sum()`](https://docs.python.org/zh-cn/3.12/library/functions.html#sum)：

### listcomp转置矩阵

本来是行存储的

```python
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
]
```

```python
'''
转成列存储
内层是来源于同一column，故外层循环就是column，则内层循环就是column固定后的不同row的循环
'''
[[row[i] for row in matrix] for i in range(4)]
```

```python
#等价于下面的代码
transposed = []
for i in range(4):
    transposed.append([row[i] for row in matrix])

transposed
[[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
```
