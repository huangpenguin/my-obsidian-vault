---
title: "override"
publish: false
tags: ["C++"]
---
# override

```cpp
#include <iostream>
#include <string>
using namespace std;

class Base//抽象类
{
public:
virtual void name() const;
virtual void most_name() const = 0;//纯粹假想函数
};
void Base::name() const
{
    cout << '1' << endl;
}
class d1: public Base//抽象类
{ 
public:
    void name() const override;
};
void d1::name()const{cout << '2' << endl;}

class d2 : public d1
{
public:
void most_name() const override;
};
void d2::most_name() const { cout << '3' << endl;}

int main()
{
d1 d; //エラー。抽象クラスのインスタンスを作ることができない
d2 d; // 
d.name();//最接近的父类，即d1的name方法
d.most_name();
}
```

1.假想関数的指定需要声明virtual

2.对于只有纯粹假想函数的抽象类不可以直接实例化，同时哪怕其中之一是纯粹假想函数，这也意味着无法实例化，因为有未知的部分

3.override了就必须写函数实现
