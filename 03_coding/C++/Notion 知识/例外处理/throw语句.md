---
title: "throw语句"
publish: false
tags: ["C++"]
---
# throw语句

在 C++ 中，当 `throw` 语句被执行时，抛出的异常会立即终止当前作用域的执行，并跳转到最近的 `catch` 块进行异常处理。因此，在 `throw` 语句之后的代码不会被执行。这种行为确保了程序在异常抛出时能够快速地离开异常发生的上下文，进入到异常处理阶段。

### 详细解

```cpp
#include <iostream>void throwException() {
    std::cout << "Throwing exception..." << std::endl;
    throw std::runtime_error("An error occurred"); // 立即跳转到 catch 块
    std::cout << "This line will not be executed." << std::endl;
}

int main() {
    try {
        std::cout << "Inside try block." << std::endl;
        throwException();
        std::cout << "This line will also not be executed." << std::endl;
    } catch (const std::runtime_error& e) {
        std::cout << "Caught exception: " << e.what() << std::endl;
    }
    std::cout << "Execution continues after the catch block." << std::endl;
    return 0;
}
```
