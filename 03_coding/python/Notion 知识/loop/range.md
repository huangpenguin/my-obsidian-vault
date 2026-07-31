---
title: "range"
publish: false
tags: ["Python"]
---
# range

语法 range(start,stop,step)

其中stop是必须的参数，start默认是0，step默认是1，range（）生成对象是可迭代的，使用下面两种方式输出。range这里也是左边start能取到，右边stop取不到。

```python
for i in range(3):
	print(i)
	
list(range(0,3))
```

注意：虽然数组切片和这里的range都是不包含终点的，但是有时候也会有包含终点的情况，比如numpy的linspace
