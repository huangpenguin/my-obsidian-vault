---
title: "std::cout::setf 简介"
publish: false
tags: ["C++"]
---
# std::cout::setf 简介

`std::cout::setf` 是 `std::ostream` 类（标准输出流 `std::cout` 的基类）中的一个成员函数，用于设置流的格式标志（format flags）。这些标志控制输出的格式，例如字段宽度、对齐方式、数值的进制等。

### 常见的格式标志

- **std::ios::left** 和 **std::ios::right**：控制字段内内容的左对齐或右对齐。
- **std::ios::dec**，**std::ios::oct** 和 **std::ios::hex**：控制整数的进制（十进制、八进制或十六进制）。
- **std::ios::fixed** 和 **std::ios::scientific**：控制浮点数的表示方式（定点或科学记数法）。
- **std::ios::showbase**：显示数值的基数前缀（如十六进制的 `0x`）。
- **std::ios::showpoint**：显示浮点数的小数点和后续的零。
- **std::ios::uppercase**：显示大写的十六进制数字和科学记数法中的 `E`。

### `setf` 方法的签名

```cpp
std::ios_base& setf(std::ios_base::fmtflags fmtfl);
std::ios_base& setf(std::ios_base::fmtflags fmtfl, std::ios_base::fmtflags mask);
```

- 第一个重载版本设置给定的格式标志，并保留之前的设置。
- 第二个重载版本将流的格式标志设置为 `fmtfl & mask`，而 `mask` 指定哪些标志会被影响。

### `std::cout::setf` 影响的输出

当使用 `std::cout::setf` 设置一个格式标志时，这个标志会影响所有后续的输出操作，直到：

- 再次使用 `setf` 方法修改或清除这些标志。
- 使用 `std::cout::unsetf` 方法清除特定的标志。
- 使用 `std::ios::flags` 方法来重置或更改流的所有格式标志。

### 具体示例

下面的代码展示了 `std::cout::setf` 的效果及其对多个输出操作的影响：

```cpp

#include <iostream>#include <iomanip>int main() {
   // 设置数字显示为十六进制，并且在前面加上 0x
    std::cout.setf(std::ios::hex, std::ios::basefield);   // 设置进制为十六进制
    std::cout.setf(std::ios::showbase);                   // 显示基数前缀

    std::cout << 255 << std::endl;   // 输出：0xff
    std::cout << 100 << std::endl;   // 输出：0x64

    // 清除进制和基数前缀的设置，返回到默认的十进制显示
    std::cout.unsetf(std::ios::basefield);                // 清除进制设置
    std::cout.unsetf(std::ios::showbase);                 // 清除基数前缀设置

    std::cout << 255 << std::endl;   // 输出：255
    std::cout << 100 << std::endl;   // 输出：100

    // 设置字段宽度和左对齐
    std::cout.width(10);
    std::cout.setf(std::ios::left);

    std::cout << 123 << "!" << std::endl;  // 输出：123       !
    std::cout << 456 << "!" << std::endl;  // 输出：456       !

    // 清除左对齐设置，返回到默认的右对齐
    std::cout.unsetf(std::ios::left);
    std::cout.width(10);

    std::cout << 789 << "!" << std::endl;  // 输出：       789!
    std::cout << 101 << "!" << std::endl;  // 输出：       101!

    return 0;
}
```

### 基数前缀的意义：

- **`0x`**：表示十六进制数。例如，`0x1A3` 表示十六进制数 `1A3`。
- **`0`**：单独的前缀 `0` 用来表示八进制数。例如，`075` 表示八进制数 `75`。
- **没有前缀**：默认表示十进制数。例如，`123` 就是十进制数 `123`。

。

- 

### 总结

- `std::cout::setf` 设置的格式标志会影响所有后续的输出，直到标志被清除或更改。
- 这些设置对 `std::cout` 流是全局的，影响多个输出操作，直到它们被显式地改变。
- 可以使用 `unsetf` 或 `flags` 方法来清除或更改这些格式设置。
- `std::cout.width`和`std::cout.precision`和`std::cout.fill是一次性的`
