---
title: "Include Guard"
publish: false
tags: ["C++"]
---
# Include Guard

### 这个过程的工作原理如下：

1. **首次包含**:
    - 当编译器第一次处理 `calculator.h` 时，`CALCULATOR_H` 未定义，所以 `#ifndef CALCULATOR_H` 的条件为真。
    - `#define CALCULATOR_H` 被执行，宏 `CALCULATOR_H` 被定义。
    - 编译器继续处理和编译 `#ifndef` 和 `#endif` 之间的内容。
2. **重复包含**:
    - 如果在同一编译单元中再次包含 `calculator.h`，`CALCULATOR_H` 已经被定义过了。
    - 由于 `#ifndef CALCULATOR_H` 条件为假，编译器会跳过 `#ifndef` 和 `#endif` 之间的所有内容。
    - 这样，避免了重复定义的问题。

```cpp
#ifndef CALCULATOR_H  // 如果未定义 CALCULATOR_H
#define CALCULATOR_H  // 定义 CALCULATOR_H

#include <string>
#include <iostream>

//abcdefg如果定义过就不会经过这里

#endif // 结束 CALCULATOR_H 检查

```
