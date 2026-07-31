---
title: "vector"
publish: false
tags: ["C++"]
---
# vector

```cpp
vector<Type> v1;//v1暂时为空，默认初始化；
vector<Type> v2(v1);//v2包含v1所有元素的副本；
```

```cpp
#include <iostream>
#include <vector>
using namespace std;
class product
{
    int id{0};
    string name{"not available"};

    public:
    product (){};//NSDMI
    explicit product(int id,string name,int price)
    :id(id),name(name),price(price)
    {};
};
int main()
{
std::vector<product> p =
{
product(1, "smart phone"),
product{2, "tablet"},
};
p.push_back(product{});
p.push_back(product{});
}
```
