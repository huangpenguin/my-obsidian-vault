---
title: "用法"
publish: false
tags: ["待整理"]
---
# 用法

```cpp

#include <set>using namespace std;

set<int> s;  // 声明一个整数类型的集合
s.insert(10);  // 向集合中插入元素
s.erase(10);  // 从集合中删除元素
bool exists = s.count(10);  // 检查元素是否存在
```

**查找元素**

- `find(val)`：查找值为 `val` 的元素的迭代器。如果找到，返回元素的迭代器；如果未找到，则返回集合末尾的迭代器 `end()`。

```cpp
auto it = s.find(10);
if (it != s.end()) {
    // 找到了元素 10
}
```

- `count(val)`：返回集合中值为 `val` 的元素的数量（因为集合中元素唯一，所以返回值只能是 0 或 1）。

```cpp
int count = s.count(10);  // 检查元素 10 的存在性
```

- **迭代器操作**
    - `begin()` 和 `end()`：返回指向集合起始和末尾的迭代器，用于遍历集合中的元素。
    
    ```cpp
    for (auto it = s.begin(); it != s.end(); ++it) {
        // 使用迭代器 it 访问集合中的元素
    }
    ```
    
- **其他操作**
    - `empty()`：检查集合是否为空，返回 `true` 或 `false`。
    
    ```cpp
    if (s.empty()) {
        // 集合为空
    ```
    
    - `size()`：返回集合中元素的数量。
    
    ```cpp
    int size = s.size();  // 获取集合中元素的数量
    ```
    
- **查找范围操作**
    - `lower_bound(val)`：返回第一个大于或等于 `val` 的元素的迭代器。
    
    ```cpp
    auto it = s.lower_bound(5);  // 返回第一个大于或等于 5 的元素的迭代器
    ```
    
    - `upper_bound(val)`：返回第一个大于 `val` 的元素的迭代器。
    
    ```cpp
    auto it = s.upper_bound(5);  // 返回第一个大于 5 的元素的迭代器
    ```
    
    - `equal_range(val)`：返回一个范围，其中包含所有键值等于 `val` 的元素的迭代器。这是 `lower_bound(val)` 和 `upper_bound(val)` 的结合使用。
    
    ```cpp
    auto range = s.equal_range(5);
    // range.first 是第一个大于或等于 5 的元素的迭代器
    // range.second 是第一个大于 5 的元素的迭代器
    ```
