---
title: "vector"
publish: false
tags: ["C++"]
---
# vector

```cpp
#include <iostream>
#include <vector>

int main() {
    // 创建一个存储整数的 vector
    std::vector<int> intVector = {1, 2, 3, 4, 5};

    // 创建一个存储双精度浮点数的 vector
    std::vector<double> doubleVector = {1.1, 2.2, 3.3, 4.4, 5.5};

    // 创建一个存储字符串的 vector
    std::vector<std::string> stringVector = {"apple", "banana", "cherry"};

    // 访问和操作 vector 中的元素
    std::cout << "Int Vector: ";
    for (auto num : intVector) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    std::cout << "Double Vector: ";
    for (auto num : doubleVector) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    std::cout << "String Vector: ";
    for (auto str : stringVector) {
        std::cout << str << " ";
    }
    std::cout << std::endl;

    return 0;
}

```
