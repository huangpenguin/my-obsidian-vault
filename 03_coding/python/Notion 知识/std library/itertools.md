---
title: "itertools"
publish: false
tags: ["Python"]
---
# itertools

```python
import itertools
#chain会将后面所有的参数当成一个迭代对象进行处理
for item in itertools.chain([1,2],['a','b']):
	print(item)
#cyle无限循环
for item in itertools.cycle([1,2]):
	print(item)
#累乘
for item in itertools.accumulate([1,2],multiply):
	print(item)

```
