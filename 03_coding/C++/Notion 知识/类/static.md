---
title: "static"
publish: false
tags: ["C++"]
---
# static

static变量的lifetime与普通变量不同，这导致初始化只会执行一次

```cpp
int count()
{
static i = 1;//第二次访问该函数就不会初始化了，沿用前一次的调用剩下来的结果
i++;
}
```
