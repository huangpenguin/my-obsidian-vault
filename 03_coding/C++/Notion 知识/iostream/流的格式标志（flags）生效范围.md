---
title: "流的格式标志（flags）生效范围"
publish: false
tags: ["C++"]
---
# 流的格式标志（flags）生效范围

在 C++ 中，当你使用 `std::ios_base::setf` 设置流的格式标志（flags）时，这些标志会一直生效，直到它们被显式地修改或清除。因此，理解如何获取、检查和重置这些标志是非常重要的。

### 获取和检查当前的标志

C++ 提供了几种方法来获取和检查当前的流格式标志：

1. **获取所有标志**：可以使用 `std::ios_base::flags()` 方法获取所有的格式标志。
2. **检查特定标志**：可以通过逻辑与操作（`&`）检查特定的标志是否被设置。

### 示例代码

以下是如何获取和检查 `std::cout` 的格式标志的示例：

```cpp
cpp复制代码
#include <iostream>#include <iomanip>int main() {
    // 获取并保存初始的流标志
    std::ios_base::fmtflags initial_flags = std::cout.flags();

    // 设置一些标志
    std::cout.setf(std::ios::hex, std::ios::basefield);    // 设置十六进制
    std::cout.setf(std::ios::left, std::ios::adjustfield); // 设置左对齐
    std::cout.setf(std::ios::showbase);                    // 显示基数前缀

    // 输出当前的标志
    std::ios_base::fmtflags current_flags = std::cout.flags();

    // 检查特定的标志
    if (current_flags & std::ios::hex) {
        std::cout << "Hex flag is set." << std::endl;
    }
    if (current_flags & std::ios::left) {
        std::cout << "Left align flag is set." << std::endl;
    }
    if (current_flags & std::ios::showbase) {
        std::cout << "Show base flag is set." << std::endl;
    }

    // 恢复初始的流标志
    std::cout.flags(initial_flags);

    return 0;
}

```

### 输出解释

- **`initial_flags`**：保存初始状态的格式标志。
- **`std::cout.setf(...)`**：设置了十六进制、左对齐和显示基数前缀的标志。
- **`current_flags`**：获取当前的标志状态。
- **检查特定标志**：使用逻辑与操作检查特定的标志是否被设置。
- **恢复初始标志**：将流的格式标志恢复为初始状态。

### 自动重置标志

一些操作（如 `std::ostream::width`）会在使用后自动重置，但大多数标志设置（如 `setf`）不会自动重置，除非你显式地重置或修改它们。这就是为什么了解如何获取和管理标志状态很重要。

### 手动恢复标志

当你需要恢复到某个特定的标志状态时，可以使用 `std::ios_base::flags()` 方法进行恢复。

### 恢复标志的示例

```cpp
cpp复制代码
#include <iostream>#include <iomanip>int main() {
    // 保存初始的流标志
    std::ios_base::fmtflags initial_flags = std::cout.flags();

    // 设置十六进制和显示基数前缀
    std::cout.setf(std::ios::hex, std::ios::basefield);
    std::cout.setf(std::ios::showbase);

    std::cout << 255 << std::endl; // 输出: "0xff"

    // 恢复初始的流标志
    std::cout.flags(initial_flags);

    std::cout << 255 << std::endl; // 输出: "255"（恢复到十进制）

    return 0;
}

```

在这个示例中：

- `initial_flags` 保存了初始的标志状态。
- 我们设置了十六进制和显示基数前缀的标志。
- 然后，通过 `std::cout.flags(initial_flags)` 恢复到初始状态，使得后续的输出恢复到默认的十进制格式。

### `mask` 参数的作用总结

`mask` 参数在 `setf` 方法中用于精确控制哪些标志会被修改。通过指定 `mask`，可以选择性地只更改特定的标志，而不影响其他的标志。例如：

```cpp
cpp复制代码
std::cout.setf(std::ios::hex, std::ios::basefield); // 只修改 basefield 相关的标志
std::cout.setf(std::ios::left, std::ios::adjustfield); // 只修改 adjustfield 相关的标志

```

在这种情况下：

- `std::ios::basefield` 包括 `std::ios::dec`、`std::ios::oct` 和 `std::ios::hex`，表示与数值进制相关的标志。
- `std::ios::adjustfield` 包括 `std::ios::left`、`std::ios::right` 和 `std::ios::internal`，表示与对齐方式相关的标志。

### 示例代码：修改多个标志

下面是一个例子，演示了如何使用 `mask` 参数来修改多个标志而不影响其他标志：

```cpp
cpp复制代码
#include <iostream>#include <iomanip>int main() {
    // 获取并保存初始的流标志
    std::ios_base::fmtflags initial_flags = std::cout.flags();

    // 设置十六进制和显示基数前缀，且仅修改 basefield
    std::cout.setf(std::ios::hex | std::ios::showbase, std::ios::basefield | std::ios::showbase);

    // 输出十六进制值
    std::cout << 255 << std::endl; // 输出: "0xff"

    // 设置左对齐，且仅修改 adjustfield
    std::cout.setf(std::ios::left, std::ios::adjustfield);

    // 设置宽度并输出
    std::cout.width(10);
    std::cout << 255 << std::endl; // 输出: "0xff      "（左对齐）

    // 恢复初始的流标志
    std::cout.flags(initial_flags);

    // 输出十进制值
    std::cout << 255 << std::endl; // 输出: "255"

    return 0;
}

```

在这个例子中：

- 使用 `std::ios::basefield | std::ios::showbase` 作为 `mask`，仅修改与进制相关的标志和基数前缀的标志。
- 使用 `std::ios::adjustfield` 作为 `mask`，仅修改与对齐方式相关的标志。
- 这样可以确保其他的标志（如对齐方式）不会被不小心修改。

```cpp
#include <iostream>
int main()
{
std::cout.setf(std::ios::right | std::ios::oct, std::ios::basefield);
std::cout.width(8);
std::cout.fill('-');
std::cout << 1234 << std::endl;
std::cout.width(8);
std::cout.fill('-');
std::cout << 1000 << std::endl;
}
```
