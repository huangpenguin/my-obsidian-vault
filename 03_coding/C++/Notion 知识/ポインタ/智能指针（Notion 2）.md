---
title: "智能指针"
publish: false
tags: ["C++"]
---
# 智能指针

### `std::unique_ptr`

`std::unique_ptr` 是一个独占所有权的智能指针。一个对象只能由一个 `std::unique_ptr` 拥有。

```cpp
#include <iostream>#include <memory>int main() {
    // 使用 std::make_unique 创建一个 std::unique_ptr
    std::unique_ptr<std::string> str = std::make_unique<std::string>("abc");

    // 使用智能指针
    std::cout << *str << std::endl;

    // std::unique_ptr 会在超出作用域时自动释放内存
    return 0;
}
```

### `std::shared_ptr`

`std::shared_ptr` 是一个共享所有权的智能指针。多个 `std::shared_ptr` 可以共享同一个对象。当最后一个 `std::shared_ptr` 被销毁时，对象会被自动删除。

```cpp
#include <iostream>#include <memory>int main() {
    // 使用 std::make_shared 创建一个 std::shared_ptr
    std::shared_ptr<std::string> str = std::make_shared<std::string>("abc");

    // 使用智能指针
    std::cout << *str << std::endl;

    // 另一个 std::shared_ptr 指向同一个对象
    std::shared_ptr<std::string> str2 = str;

    std::cout << *str2 << std::endl;

    // std::shared_ptr 会在超出作用域时自动释放内存
    return 0;
}
```

### `std::weak_ptr`

`std::weak_ptr` 是一种弱引用智能指针，不会影响对象的生命周期。它通常与 `std::shared_ptr` 配合使用，用于解决循环引用的问题。

```cpp
#include <iostream>#include <memory>int main() {
    // 创建一个 std::shared_ptr
    std::shared_ptr<std::string> str = std::make_shared<std::string>("abc");

    // 创建一个 std::weak_ptr 引用相同的对象
    std::weak_ptr<std::string> weakStr = str;

    // 使用 lock 方法创建一个 std::shared_ptr 从 std::weak_ptr
    if (auto sharedStr = weakStr.lock()) {
        std::cout << *sharedStr << std::endl;
    } else {
        std::cout << "对象已被删除" << std::endl;
    }

    // std::shared_ptr 会在超出作用域时自动释放内存
    return 0;
}
```
