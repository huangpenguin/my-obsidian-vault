---
title: "智能指针-以viego为例"
publish: false
tags: ["C++"]
---
# 智能指针-以viego为例

常见的智能指针包括 **`std::unique_ptr`**、**`std::shared_ptr`** 和 **`std::weak_ptr`**。它们分别提供了不同的内存管理策略：

- **`std::unique_ptr`**：独占所有权的智能指针，只能有一个指针指向同一块内存，当 **`std::unique_ptr`** 被销毁时，它所指向的内存会被释放。
- **`std::shared_ptr`**：共享所有权的智能指针，可以有多个指针指向同一块内存，内存会在所有指向它的 **`std::shared_ptr`** 都被销毁时才会被释放。
- **`std::weak_ptr`**：弱引用智能指针，不会增加引用计数，需要配合 **`std::shared_ptr`** 使用，用于避免循环引用问题。

在示例代码中，我使用了 **`std::unique_ptr`** 来管理技能对象的内存。通过 **`std::make_unique`** 函数创建智能指针，可以确保资源在使用完毕后自动释放，从而避免内存泄漏问题。

```cpp
#include <iostream>
#include <string>
#include <memory>

// 技能基类
class Skill {
public:
    virtual void use() const = 0;
};

// 具体技能类
class Skill_A : public Skill {
public:
    void use() const override {
        std::cout << "Skill_A!!!!" << std::endl;
    }
};

// 具体技能类
class Skill_B : public Skill {
public:
    void use() const override {
        std::cout << "Skill_B!!!!" << std::endl;
    }
};

// 英雄类
class Hero {
    std::unique_ptr<Skill> skill_q;//member_variable

public:
    Hero() : skill_q(std::make_unique<Skill_A>()) {}//construction
    
    void attack() {
        if (/* 王の支配OK */) {
            skill_q = std::make_unique<Skill_B>();
        }
    }

    void useSkill() const {
        skill_q->use();
    }
};

int main() {
    Hero hero;
    hero.useSkill(); 

    hero.attack();
    hero.useSkill(); 

    return 0;
}

```
