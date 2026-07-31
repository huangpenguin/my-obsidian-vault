---
title: "std::sort"
publish: false
tags: ["C++"]
---
# std::sort

`std::sort` 是 C++ 标准库中的一个算法函数，用于对指定范围内的元素进行排序。它是一个模板函数，可以适用于任何支持随机访问迭代器的容器，例如 `std::vector`、`std::array`、`std::deque` 等。

### `std::sort` 函数的完整模板定义

```cpp
cppCopy code
namespace std {
    template< class RandomIt >
    void sort( RandomIt first, RandomIt last );

    template< class RandomIt, class Compare >
    void sort( RandomIt first, RandomIt last, Compare comp );
}

```

- `RandomIt` 是一个模板参数，表示支持随机访问的迭代器类型。
- `Compare` 是另一个模板参数，表示一个比较函数或函数对象类型。

```cpp
// 高阶函数升序的例子
std::sort(users.begin(), users.end(), compareUsers);
bool compareUsers(const User& a, const User& b) {
    return a.name < b.name;//true就这么返回
}
//lambda 表达式
std::sort(users.begin(), users.end(), [](const User& a, const User& b) {
    return a.rating < b.rating;
});

```

### 比较函数为什么返回 `bool` 值

比较函数返回 `bool` 值是因为 `std::sort` 使用这个布尔值来决定元素的顺序。让我们看一个简单的排序过程：

1. **比较函数的调用**：
    - 当 `std::sort` 需要比较两个元素 `a` 和 `b` 时，它会调用比较函数 `comp(a, b)`。
2. **返回 `true` 或 `false`**：
    - 如果 `comp(a, b)` 返回 `true`，则 `a` 应该排在 `b` 之前。
    - 如果 `comp(a, b)` 返回 `false`，则 `a` 不应该排在 `b` 之前（`b` 可能排在 `a` 之前，或 `a` 和 `b` 的顺序保持不变）。
