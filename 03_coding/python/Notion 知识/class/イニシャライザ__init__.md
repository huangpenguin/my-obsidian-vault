---
title: "イニシャライザ__init__"
publish: false
tags: ["Python"]
---
# イニシャライザ__init__

注意在调用这个函数之前，object就已经生成了，这个函数充其量是对类的变量进行了初始化，与c++中的constructor不同。

---

从字典创建的话，可以用**操作符进行解包字典

```python
object_from_dict=Animal(**dict_ani)
```
