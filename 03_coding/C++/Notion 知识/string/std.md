---
title: "std"
publish: false
tags: ["C++"]
---
# std

### `std::find` 和 `std::find_if` 的区别

1. **`std::find`**：
    - **用法**：`std::find(first, last, value)` 用于在范围 `[first, last)` 内查找与 `value` 相等的第一个元素。
    - **查找条件**：查找的条件是元素等于 `value`。
    - **返回值**：返回一个指向找到的元素的迭代器，如果未找到，则返回 `last`。
2. **`std::find_if`**：
    - **用法**：`std::find_if(first, last, predicate)` 用于在范围 `[first, last)` 内查找第一个使谓词 `predicate` 返回 `true` 的元素。
    - **查找条件**：查找的条件由 `predicate` 函数指定，可以是任意的逻辑条件。
    - **返回值**：返回一个指向找到的元素的迭代器，如果未找到，则返回 `last`。

```cpp
auto iter = std::find_if(v.begin(), v.end(), [](int v) { return v == 3; });
```

### 代码中不同结果的原因

尽管两个查找函数在你的示例中都找到了目标元素，但它们的查找条件是不同的：

- **`std::find(v.begin(), v.end(), 2)`**：
    - 查找范围 `[v.begin(), v.end())` 中第一个等于 `2` 的元素。
    - 找到了 `2` 这个元素，并返回指向它的迭代器。
    - 输出 `iter = 2`。
- **`std::find_if(v.begin(), v.end(), [](int v) { return v == 3; })`**：
    - 查找范围 `[v.begin(), v.end())` 中第一个满足 `lambda` 表达式 `v == 3` 的元素。
    - 找到了 `3` 这个元素，并返回指向它的迭代器。
    - 输出 `iter = 3`。
