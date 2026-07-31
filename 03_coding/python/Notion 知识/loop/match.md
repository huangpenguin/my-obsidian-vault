---
title: "match"
publish: false
tags: ["Python"]
---
# match

## match

```python
*# point is an (x, y) tuple* 
**match** point:
    **case** (0, 0):
        print("Origin")
    **case** (0, y):
        print(f"Y=**{**y**}**")
    **case** (x, 0):
        print(f"X=**{**x**}**")
    **case** (x, y):
        print(f"X=**{**x**}**, Y=**{**y**}**")
    **case** **_**:
        **raise** **ValueError**("Not a point")
```

第一个模式有两个字面值，可视为前述字面值模式的扩展。接下来的两个模式结合了一个字面值和一个变量，变量 *绑定* 了来自主语（`point`）的一个值。第四个模式捕获了两个值，使其在概念上与解包赋值 `(x, y) = point` 类似。

```python
class Point:
def __init__(self, x, y):
        self.x = x
        self.y = y

def where_is(point):
match point:
case Point(x=0, y=0):
            print("Origin")
case Point(x=0, y=y):
            print(f"Y={y}")
case Point(x=x, y=0):
            print(f"X={x}")
case Point():
            print("Somewhere else")
case_:
            print("Not a point")
```

```python
class Point:
    __match_args__ = ('x', 'y')
def __init__(self, x, y):
        self.x = x
        self.y = y

match points:
    case []:
        print("No points")
    case [Point(0, 0)]:
        print("The origin")
    case [Point(x, y)]:
        print(f"Single point{x},{y}")
    case [Point(0, y1), Point(0, y2) as p2]:
        print(f"Two on the Y axis at{y1},{y2}")
    case _:
        print("Something else")
```

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
```
