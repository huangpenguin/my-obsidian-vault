---
title: "map"
publish: false
tags: ["C++"]
---
# map

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    // 创建一个存储 <int, string> 键值对的 map
    std::map<int, std::string> intToStringMap = {
        {1, "one"},
        {2, "two"},
        {3, "three"}
    };

    // 创建一个存储 <string, double> 键值对的 map
    std::map<std::string, double> stringToDoubleMap = {
        {"pi", 3.14},
        {"e", 2.71},
        {"sqrt2", 1.41}
    };

    // 访问和操作 map 中的键值对
    std::cout << "Int to String Map:" << std::endl;
    for (const auto& pair : intToStringMap) {
        std::cout << pair.first << " -> " << pair.second << std::endl;
    }

    std::cout << "String to Double Map:" << std::endl;
    for (const auto& pair : stringToDoubleMap) {
        std::cout << pair.first << " -> " << pair.second << std::endl;
    }

    return 0;
}

```
