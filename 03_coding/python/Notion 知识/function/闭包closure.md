---
title: "闭包closure"
publish: false
tags: ["Python"]
---
# 闭包closure

闭包（Closure）是指在函数内部定义的函数，可以访问其外部函数的变量和参数，即使外部函数已经返回。闭包的重要特性是能够“记住”创建它时的环境，因此它在函数式编程和其他编程范式中非常有用。

### 闭包的特性

1. **保持状态：** 闭包可以记住并访问它所在的作用域中的变量，即使在这个作用域已经结束的情况下。
2. **函数嵌套：** 闭包是一种嵌套函数，它定义在另一个函数内部。
3. **函数作为返回值：** 闭包通常在外部函数中定义，并作为返回值返回。

### 闭包的例子

```python
def outer_function(x):
    def inner_function(y):
        return x + y
    return inner_function

add_five = outer_function(5)  # outer_function 返回的是 inner_function,此时具有记忆效应的inner_function就是closure
print(add_five(10))  # 然后再去调用这个记忆好的closure

###总体上感觉有点像记忆T细胞
```

在这个例子中：

- `outer_function` 是一个外部函数，接受一个参数 `x`。
- `inner_function` 是定义在 `outer_function` 内部的一个内部函数，它接受一个参数 `y`。
- `outer_function` 返回 `inner_function`。
- `add_five` 是一个闭包，它捕获了 `outer_function` 的参数 `x`，即 `5`，并将其保存在自己的环境中。
- 调用 `add_five(10)` 时，闭包会使用保存在其环境中的 `x` 的值 `5`，并将其与参数 `y` 的值 `10` 相加，得到结果 `15`。

### 闭包的应用场景

1. **数据隐藏和封装：** 闭包可以用于创建私有变量，从而实现数据隐藏和封装。
    
    ```python
    def make_counter():
        count = 0
        def counter():
            nonlocal count
            count += 1
            return count
        return counter
    
    counter1 = make_counter()
    print(counter1())  # 输出: 1
    print(counter1())  # 输出: 2
    
    counter2 = make_counter()
    print(counter2())  # 输出: 1
    
    ```
    
2. **回调函数：** 闭包常用于回调函数，尤其是在异步编程或事件驱动编程中。
    
    ```python
    def make_multiplier(x):
        def multiplier(y):
            return x * y
        return multiplier
    
    double = make_multiplier(2)
    print(double(5))  # 输出: 10
    
    triple = make_multiplier(3)
    print(triple(5))  # 输出: 15
    
    ```
    
3. **函数式编程：** 闭包是函数式编程的重要组成部分，允许创建高阶函数和柯里化（Currying）。

### 总结

闭包是强大而灵活的编程工具，通过捕获和记住其定义环境中的变量，使得函数能够在其外部函数作用域结束后仍然访问这些变量。这种特性使得闭包在许多编程场景中都非常有用，包括数据隐藏、回调函数和函数式编程等。
