---
title: "None or 0?"
publish: false
tags: ["Python"]
---
# None or 0?

空tuple，空list，空集合，0的数值等等，虽然在判定上和None同为False，

但是在实质上是不一样的，可以借助is关键词来检查

```python
def whatis(thing):
	if thing is None:
		print('nothing')
	else:
		print('something')
```
