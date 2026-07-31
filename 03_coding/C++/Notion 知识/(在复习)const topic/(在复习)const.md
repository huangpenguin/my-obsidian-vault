---
title: "(在复习)const"
publish: false
tags: ["C++"]
---
# (在复习)const

## **`const int`** 和 **`#define`** 区别

**`const int`** 定义的常量具有作用域，只在定义它的块内可见。**`#define`** 定义的常量是全局的，它在定义点之后的整个文件中都有效，甚至在其他文件中也有效。

**`const int`** 定义的常量有类型，因此在编译时进行类型检查。当使用 **`const int`** 定义常量时，如果程序中使用了该常量的不正确类型，则会在编译时或运行时产生错误消息，并提供更好的调试信息。**`#define`** 定义的常量只是简单的文本替换，没有类型信息，因此不会进行类型检查，错误可能会很难调试。

## **const对象默认为文件局部变量**

非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。未被const修饰的变量不需要extern显式声明！而const常量需要显式声明extern，并且需要做初始化！因为常量在定义后就不能被修改，所以定义时必须初始化。

在 C 和 C++ 中，数组的名称本质上是一个常量指针，指向数组的第一个元素。由于它是一个常量指针，因此不能修改它以指向不同的位置。例如，**`a = p;`** 这样的赋值是不允许的。

[https://github.com/Light-City/CPlusPlusThings/tree/master/basic_content/const](https://github.com/Light-City/CPlusPlusThings/tree/master/basic_content/const)
