---
title: "default exception"
publish: false
tags: ["Python"]
---
# default exception

```python
short_list=[1,2,3]
while True:
	value=input('Position [q to quit]')
	if value=='q':
		break
	try:
		position=int (value)
		print(short_list[position])
	except IndexError as err:
		print('Bad index',position)
	except Exception as other:
		print('Something else broke:',other)
```

```python
try:
    raise Exception('spam', 'eggs')
except Exception as inst:
    print(type(inst))    # the exception type
    print(inst.args)     # arguments stored in .args
    print(inst)          # __str__ allows args to be printed directly,
                         # but may be overridden in exception subclasses
    x, y = inst.args     # unpack args
    print('x =', x)
    print('y =', y)
```
