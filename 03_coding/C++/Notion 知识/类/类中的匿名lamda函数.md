---
title: "类中的匿名lamda函数"
publish: false
tags: ["C++"]
---
# 类中的匿名lamda函数

成员函数中的lamda函数无法直接捕获到成员变量，所以需要通过捕获this指针来获取成员变量。

```cpp
#include <iostream>
#include <vector>
using namespace std;

class C
{
    int a{1};
  public:
  void show_a()
  {
    [this]()
    {cout<<this->a<<endl;}();//this是指针，这里只能用arrow
  }  
};
int main() 
{
 C c{};
 c.show_a();
}
```
