---
title: "多态（Polymorphism）ポリモーフィズム"
publish: false
tags: ["Python"]
---
# 多态（Polymorphism）ポリモーフィズム

在 Python 中，**多态**（Polymorphism）是面向对象编程（OOP）的一个重要概念。它允许对象以多种形式存在，并对相同的操作做出不同的响应。简单来说，多态是不同的类可以以相同的方式被操作，即使这些类实现了不同的行为。

### 多态的基本概念

1. **方法重写（Method Overriding）**：
子类可以重写父类中的方法，提供特定的实现。在使用时，我们可以调用子类的实例方法，但不需要知道它的具体实现，只需要知道它具有相同的方法签名（方法名称和参数）。
2. **接口一致性**：
不同的类可以实现相同的方法或接口，但这些方法的具体实现可能不同。这样，我们就可以用统一的接口来处理这些不同的对象。

### 代码示例

以下是一个简单的 Python 示例，展示了如何使用多态来处理不同的对象：

```python
#总结：多个类有同名方法，利用一个函数make_animal_speak来实现多态调用
class Animal:
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

def make_animal_speak(animal: Animal):
    print(animal.speak())

# 示例
dog = Dog()
cat = Cat()

make_animal_speak(dog)  # 输出: Woof!
make_animal_speak(cat)  # 输出: Meow!

###
make_animal_speak 是函数的名称。
animal: Animal 是函数的参数部分，其中 animal 是参数的名称，Animal 是类型提示（type hint）。
print(animal.speak()) 是函数体部分，表示函数执行时的操作。
###
```

### 解释

1. **定义父类和子类**：
    - `Animal` 是一个基类（父类），定义了一个方法 `speak`。这个方法在基类中没有具体实现（用 `pass` 表示）。
    - `Dog` 和 `Cat` 是 `Animal` 的子类（派生类），它们都重写了 `speak` 方法，以提供特定的实现。
2. **多态的实现**：
    - `make_animal_speak` 函数接受一个 `Animal` 类型的参数，可以是 `Dog` 或 `Cat` 的实例。
    - 当调用 `make_animal_speak(dog)` 时，它会输出 `Woof!`，因为 `dog` 是 `Dog` 类的实例，`Dog` 类重写了 `speak` 方法。
    - 当调用 `make_animal_speak(cat)` 时，它会输出 `Meow!`，因为 `cat` 是 `Cat` 类的实例，`Cat` 类重写了 `speak` 方法。

### 多态的优点

1. **灵活性**：
多态允许你编写更通用的代码，这些代码可以处理各种不同的对象，而不必关心这些对象的具体类型。
2. **可扩展性**：
新的类可以通过继承已有的基类并实现基类的方法来扩展系统，无需修改现有代码，只需要保证新的类遵循相同的接口。
3. **代码重用**：
多态可以使代码更具通用性和复用性，减少了重复代码的编写。

### 类型提示（Type Hint）

- `animal: Animal` 中的 `Animal` 是一种类型提示。类型提示用于指定函数参数或返回值的预期类型。在这里，它表示 `animal` 参数应当是 `Animal` 类型的对象。
- 类型提示并不会强制检查类型，而是提供给开发者和工具（如 IDE 或静态类型检查器）有关预期类型的信息。它有助于代码的可读性和自动化工具的支持。
