---
title: "const话题"
publish: false
tags: ["C++"]
---
# const话题

**`const`**修饰符在成员函数的参数列表后面表示该成员函数是一个“***常量成员函数***”（const member function）。这意味着在该成员函数中不能修改对象的成员变量，常量成员函数可以保证对象状态的不变性。

```cpp
float Rectangle::area() const {
    // height = 10; // 这样做会导致编译错误，因为函数被声明为const
    return height * width;
}

```

### **为什么要放在函数参数列表后面**

在C++中，**`const`**修饰符放在函数参数列表的后面，表示该函数是一个常量成员函数。这种语法在语言设计上是为了区分函数参数的**`const`**修饰和成员函数的**`const`**修饰。\

正因此，构造函数不能是 **`const`** 成员函数。构造函数的主要职责是初始化对象的成员变量，这需要对对象的状态进行修改。由于 **`const`** 成员函数承诺不修改对象的状态，所以构造函数不能声明为 **`const`**。

| const成员函数
 | 该函数不会修改调用它的对象。通过在函数声明后面添加const来声明 | const MyClass obj;
 |
| --- | --- | --- |
| 
 |  | void someFunction(const MyClass& obj) {
// 在这个函数中，obj是const，不能修改obj的状态
obj.someConstFunction();  // 可以调用const成员函数
// obj.someNonConstFunction();  // 错误，不能调用非const成员函数
}
 |
|  |  | const MyClass& getConstObj() {
static MyClass obj;
return obj;  // 返回一个const引用
}
 |
| const对象 | 对象本身是不可变的，不能调用其非const成员函数，也不能修改其状态。通过在对象类型前面添加const来声明 | class MyClass {
public:
int getValue() const {
return value;
}

void setValue(int val) {
value = val;
}

private:
int value;
};

int main() {
MyClass obj1;
const MyClass obj2;

obj1.setValue(10);  // 允许，obj1不是const
int val1 = obj1.getValue();  // 允许，getValue()是const成员函数

// obj2.setValue(20);  // 错误：不能在const对象上调用非const成员函数
int val2 = obj2.getValue();  // 允许，getValue()是const成员函数

return 0;
}
 |
