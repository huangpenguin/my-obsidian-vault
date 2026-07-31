---
title: "rethrow"
publish: false
tags: ["C++"]
---
# rethrow

### rethrow的工作原理

当 `throw;` 在 `catch` 块中被执行时，它会重新抛出 `try` 块中捕获到的当前异常。此操作不会改变异常的类型和内容，而是将异常传递给上一级的异常处理器（如果有的话）。如果上一级没有合适的处理器，程序将终止。

```jsx
#include <iostream>
#include <fstream>

void processFile(const std::string& filename) {
    std::ifstream file;
    try {
        file.open(filename);
        if (!file.is_open()) {
            throw std::runtime_error("File could not be opened");
        }
        // 文件操作...
    } catch (const std::runtime_error& e) {
        std::cerr << "Error processing file: " << e.what() << std::endl;
        throw; // 重新抛出异常以便在更高层次处理
    }
}

int main() {
    try {
        processFile("nonexistent_file.txt");
    } catch (const std::exception& e) {
        std::cerr << "Caught in main: " << e.what() << std::endl;
    }
    return 0;
}

```
