---
title: "(打印实例)manipulate可以直接将格式控制插入打印的语句中去"
publish: false
tags: ["C++"]
---
# (打印实例)manipulate可以直接将格式控制插入打印的语句中去

```cpp

//マニピュレータを使用した整数表示
#include <iostream>
#include <iomanip>
int main()
{
std::cout << std::right << std::oct
<< std::setw(8) << std::setfill(’0’)
<< 1234 << std::endl;
}
```

```cpp
#include <iostream>
#include <iomanip>
int main()
{

//　メンバー関数を使用する方法
std::cout.setf(std::ios::hex, std::ios::basefield);
std::cout.setf(std::ios::left, std::ios::adjustfield);
std::cout.width(16);
std::cout.fill(’=’);
std::cout << 0xdeadbeef << std::endl;

// マニピュレータを使用する方法
std::cout << std::hex << std::left
<< std::setw(16) << std::setfill(’=’)
<< 0xdeadbeef << std::endl;
}
```
