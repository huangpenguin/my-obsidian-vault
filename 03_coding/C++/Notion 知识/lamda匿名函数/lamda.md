---
title: "lamda"
publish: false
tags: ["C++"]
---
# lamda

```cpp
///对比一下是否使用lamda表达式,首先是不使用

#include <iostream>
#include <vector>
#include <numeric> // accumulate 函数所在的头文件

// 定义一个二元函数，用于计算两个整数的和
int add(int a, int b) {
    return a + b;
}

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    
    // 使用普通函数作为累加操作的二元函数
    int sum = std::accumulate(numbers.begin(), numbers.end(), 0, add);

    std::cout << "Sum: " << sum << std::endl;

    return 0;
}

///使用
int main()
{
std::vector<int> numbers = {1, 2, 3, 4, 5};
int sum = std::accumulate(numbers.begin(), numbers.end(), 0, [](int a, int b) { return a + b; });
}
```
