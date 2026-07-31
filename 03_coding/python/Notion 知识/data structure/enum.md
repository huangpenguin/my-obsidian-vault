---
title: "enum"
publish: false
tags: ["Python"]
---
# enum

## 可以用来组织具名常量

```python
from enum import Enum
class Color(Enum):
    RED = 'red'
    GREEN = 'green'
    BLUE = 'blue'

color = Color(input("Enter your choice of 'red', 'blue' or 'green': "))

match color:
    case Color.RED:
        print("I see red!")
    case Color.GREEN:
        print("Grass is green")
    case Color.BLUE:
        print("I'm feeling the blues :(")

print(Color.RED)       # 输出 Color.RED
print(Color.RED.name)  # 输出 'RED'
print(Color.RED.value) # 输出 1

```
