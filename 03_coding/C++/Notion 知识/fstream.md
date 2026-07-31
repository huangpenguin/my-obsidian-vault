---
title: "fstream"
publish: false
tags: ["C++"]
---
# fstream

将当前的源码文件打开并打印

```cpp
#include <fstream>
#include <iostream>
int main()
{
std::ifstream in{__FILE__};
std::string line;
while (!in.eof())
{
std::getline(in, line);
std::cout << line << std::endl;
}
}
```

从键盘获取文件名并访问

```cpp
#include <iostream>
#include <fstream>
#include <string>
int main()
{
std::string line;
std::cout << "> "; // プロンプト
std::getline(std::cin, line);
std::ifstream in{line};
std::getline(in, line);
std::cout << line << std::endl;
}
```

block读入

```cpp
#include <fstream>
int main()
{
std::ifstream in{"input.bin", std::ios::binary};
char buffer[100];
in.read(buffer, sizeof(buffer));
}
```
