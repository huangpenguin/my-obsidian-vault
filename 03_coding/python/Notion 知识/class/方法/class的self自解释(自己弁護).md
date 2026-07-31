---
title: "class的self自解释(自己弁護)"
publish: false
tags: ["Python"]
---
# class的self自解释(自己弁護)

其实就是每次调用class的方法时，会按照self将object代入，此时如果直接指定object也可以

```python
class Laser:
    def does(self) -> str:# if no self here,when calling does() it will cause TypeError
        return "disintegrate"

class Claw:
    def does(self) -> str:
        return "crush"

class SmartPhone:
    def does(self) -> str:
        return "ring"

class Robot:
    def __init__(self) -> None:
        self.laser = Laser()#make instance
        self.claw = Claw()#make instance
        self.smartphone = SmartPhone()#make instance

    def does(self) -> str:
        return f"i can {self.laser.does()},\n\
and i can {self.claw.does()},\n\
and i can {self.smartphone.does()}"
```

在 Python 中，类的方法需要显式地传递实例引用（通常命名为 `self`）作为第一个参数。这是因为 Python 没有像一些其他面向对象编程语言那样的隐式 `this` 变量。通过 `self` 参数，方法可以访问和修改实例的属性和调用其他方法.

### 为什么 `self` 是必要的

`self` 参数是指向实例本身的引用。通过它，类的方法可以访问和修改实例的属性，并调用该实例的其他方法。没有 `self` 参数，Python 不知道该把调用绑定到哪个实例上。

由于方法调用 `laser.does()` 实际上被解释为 `Laser.does(laser)`，Python 试图将实例 `laser` 作为参数传递给 `does` 方法。然而，`does` 方法没有接收参数的定义，这会导致类型错误。

### 正确使用 `self`

通过显式地使用 `self` 参数，可以确保实例方法正确地引用实例：

```python
class Laser:
    def does(self) -> str:
        return "disintegrate"

```

这样，`laser.does()` 会正确地工作，因为 `self` 参数显式地接收了实例引用：

```python
laser = Laser()
print(laser.does())  # 输出：disintegrate

```

### 在你的代码中

在你的代码中，所有的组件类 `Laser`, `Claw`, 和 `SmartPhone` 都定义了一个实例方法 `does`，这需要 `self` 参数来正确调用这些方法：

```python
python复制代码
class Laser:
    def does(self) -> str:
        return "disintegrate"

class Claw:
    def does(self) -> str:
        return "crush"

class SmartPhone:
    def does(self) -> str:
        return "ring"

class Robot:
    def __init__(self) -> None:
        self.laser = Laser()
        self.claw = Claw()
        self.smartphone = SmartPhone()

    def does(self) -> str:
        return f"I can {self.laser.does()},\nand I can {self.claw.does()},\nand I can {self.smartphone.does()}"

```

在 `Robot` 类中，`self.laser`, `self.claw`, 和 `self.smartphone` 都是 `Laser`, `Claw`, 和 `SmartPhone` 类的实例。因此，当调用它们的 `does` 方法时，Python 需要知道要将调用绑定到哪个实例上，这就是 `self` 参数的作用。

### 代码运行示例

完整代码运行示例如下：

```python
python复制代码
class Laser:
    def does(self) -> str:
        return "disintegrate"

class Claw:
    def does(self) -> str:
        return "crush"

class SmartPhone:
    def does(self) -> str:
        return "ring"

class Robot:
    def __init__(self) -> None:
        self.laser = Laser()
        self.claw = Claw()
        self.smartphone = SmartPhone()

    def does(self) -> str:
        return f"I can {self.laser.does()},\nand I can {self.claw.does()},\nand I can {self.smartphone.does()}"

robot = Robot()
print(robot.does())

```

输出：

```
plaintext复制代码
I can disintegrate,
and I can crush,
and I can ring

```

这展示了正确使用 `self` 参数的方法和原因。

4o
