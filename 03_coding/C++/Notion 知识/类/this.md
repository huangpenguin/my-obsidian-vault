---
title: "this"
publish: false
tags: ["C++"]
---
# this

```cpp
class A
{
int value = 0;
public:
void set(int value);
int get() const;
};
void A::set(int value)
{
value = value; // 两边都是引数
this->value = value; //这样才把引数value的值赋给了メンバー
}
int main()
{
A a;
a.set(42);
std::cout << a.get() << std::endl;
}
```

this不可自己定义、修改值，因为是默认生成的
