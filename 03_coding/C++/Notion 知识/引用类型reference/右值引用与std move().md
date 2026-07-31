---
title: "右值引用与std::move()"
publish: false
tags: ["C++"]
---
# 右值引用与std::move()

`std::move` 是一个标准库函数，它将其参数显式地转换为右值引用。它本身不做任何实际的移动操作，只是类型转换。通过这种转换，可以调用类的移动构造函数或移动赋值运算符，而不是拷贝构造函数或拷贝赋值运算符

```cpp
#include <iostream>
#include <memory>

class A {
public:
    A() { std::cout << "コンストラクター" << std::endl; }
    ~A() { std::cout << "デストラクター" << std::endl; }
};

std::unique_ptr<A> allocate() {
    std::cout << "allocate()" << std::endl;
    std::unique_ptr<A> ptr{ new A{} };
    return std::move(ptr); // 转移所有权
}

int main() {
    {
        std::unique_ptr<A> ptr; // 空的 std::unique_ptr，初始化为 nullptr
        std::cout << "関数呼び出しの前" << std::endl;
        ptr = allocate(); // 接收 allocate 返回的所有权
        std::cout << "関数呼び出しのあと" << std::endl;
    } // ptr 作用域结束，自动释放资源
    std::cout << "スコープのあと" << std::endl;
}

```

```cpp
#include <iostream>
#include <memory>

class Resource {
public:
    Resource() {
        std::cout << "Resource acquired\n";
    }
    ~Resource() {
        std::cout << "Resource destroyed\n";
    }
    void sayHi() const {
        std::cout << "Hi from Resource\n";
    }
};

void transferOwnership(std::unique_ptr<Resource> src, std::unique_ptr<Resource>& dst) {
    dst = std::move(src);  // 通过 std::move 转移所有权
}

int main() {
    std::unique_ptr<Resource> res1 = std::make_unique<Resource>();
    std::cout << "res1 owns the Resource\n";

    std::unique_ptr<Resource> res2;
    std::cout << "res2 does not own any Resource\n";

    // 转移所有权
    transferOwnership(std::move(res1), res2);

    if (!res1) {
        std::cout << "res1 no longer owns the Resource\n";
    }

    if (res2) {
        std::cout << "res2 now owns the Resource\n";
        res2->sayHi();
    }

    return 0;
}

```

- **创建 `Resource` 对象**：
    - `std::unique_ptr<Resource> res1 = std::make_unique<Resource>();` 创建一个 `Resource` 对象，并让 `res1` 拥有它。此时输出 "Resource acquired"。
- **初始化 `res2`**：
    - `std::unique_ptr<Resource> res2;` 创建一个空的 `std::unique_ptr`，初始化为 `nullptr`。此时 `res2` 不拥有任何资源。
- **转移所有权**：
    - 调用 `transferOwnership(std::move(res1), res2);` 将 `res1` 的所有权转移给 `res2`。
    - 在 `transferOwnership` 函数内部，使用 `dst = std::move(src);` 将 `src` 的所有权转移给 `dst`。
    - 由于使用了 `std::move`，`src` 被转换为右值引用，从而调用 `std::unique_ptr` 的移动赋值运算符，将资源的所有权从 `src` 转移到 `dst`。此时 `res1` 不再拥有资源，而 `res2` 获得了资源的所有权。
- **检查所有权转移结果**：
    - 通过 `if (!res1)` 判断 `res1` 是否为空，如果为空，则说明 `res1` 不再拥有资源，输出 "res1 no longer owns the Resource"。
    - 通过 `if (res2)` 判断 `res2` 是否拥有资源，如果拥有，则输出 "res2 now owns the Resource" 并调用 `res2->sayHi()`，输出 "Hi from Resource"。
- **资源释放**：
    - 当 `res2` 超出其作用域时，其析构函数被调用，自动释放 `Resource` 对象，输出 "Resource destroyed"。
