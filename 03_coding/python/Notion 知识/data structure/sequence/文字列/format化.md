---
title: "format化"
publish: false
tags: ["Python"]
---
# format化

# python2.6之后的使用新的style

## format_string.format(data)

```python
#顺序问题

thing='apple'
place='lake'
'the {} is in the {}.'.format(thing,place)
'the {1} is in the {0}.'.format(place,thing)#可以使用数字来指定参数顺序
'the {thing} is in the {place}.'.format(thing='duck',place='bathtub')#也可以通过关键词传递

#还可以通过字典传递
d={'thing':'duck','place':'bathtub'}#键值对
'the {0[thing]} is in the {0[place]}.'.format(d)#0表示第1个参数，也就是这里的字典d
```

```python
#具体的style指定方法可以参照官方文档
'the {:10s} is {:10s}'.format(thing, place)
```

# 最新的格式可以使用f字符串

```python
thing='duck'
place='lake'
print(f'the {thing.capitalize():>20s} is in the {place.rjsut(20):.^20}')
```

```python
#还可以自动通过等号输出变量值，方便debug

print(f'{thing[-4:]=:>4.4},{place.title()=:>4.4}')
```
