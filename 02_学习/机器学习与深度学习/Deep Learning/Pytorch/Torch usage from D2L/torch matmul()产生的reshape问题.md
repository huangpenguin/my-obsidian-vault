---
title: "torch.matmul()产生的reshape问题"
publish: false
tags: ["机器学习"]
---
# torch.matmul()产生的reshape问题

### 如下代码中y_hat是(10,1)，而直接创建的一维tensor则为(10,)，故需要进行reshape

```
def squared_loss(y_hat, y):  #@save
    """均方损失"""
    return (y_hat - y.reshape(y_hat.shape)) ** 2 / 2
```

### 常见的还有tensor和计算结果矩阵之间计算也要把tensor转过去
