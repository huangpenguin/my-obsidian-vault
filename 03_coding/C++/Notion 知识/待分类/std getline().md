---
title: "std::getline()"
publish: false
tags: ["C++"]
---
# std::getline()

语法:

std::getline(std::istream& is, std::string& str);

用例:

```cpp
std::getline(std::cin, input);
```

### 工作原理

1. **等待用户输入**: 当 `std::getline(std::cin, input)` 被调用时，程序会等待用户输入。
2. **读取整行文本**: 用户输入后，按下 Enter 键（表示行的结束），`std::getline` 会从输入流中读取整行文本，直到遇到换行符（`'\n'`）。
3. **去除换行符**: 换行符不会被包含在 `input` 中，它仅用来标识行的结束。
4. **存储到字符串**: `input` 变量将包含用户输入的内容，不包括最后的换行符。

### `std::getline` 的优点

- **完整行读取**: 可以读取包括空格在内的整行文本。这在处理用户输入时非常有用，因为 `std::cin >>` 操作符会忽略空格和换行符。
- **灵活性**: 可以从任何输入流中读取，如 `std::ifstream` 读取文件内容，`std::istringstream` 读取字符串流等。
