---
title: "何时使用 torch.no_grad()"
publish: false
tags: ["机器学习"]
---
# 何时使用 torch.no_grad()

- **推理或评估**：在模型训练完毕后进行推理或评估时，通常使用 `torch.no_grad()`，因为这些步骤不需要更新参数。
- **计算损失**：在监测训练过程中的损失时，使用 `torch.no_grad()` 也是一种良好的实践，确保不会无意间积累不必要的梯度信息。

```python
for epoch in range(num_epochs):
    for X, y in data_iter(batch_size, features, labels):
        l = loss(net(X, w, b), y)  # X和y的小批量损失
        # 因为l形状是(batch_size,1)，而不是一个标量。l中的所有元素被加到一起，
        # 并以此计算关于[w,b]的梯度
        l.sum().backward()
        sgd([w, b], lr, batch_size)  # 使用参数的梯度更新参数
    with torch.no_grad():
        train_l = loss(net(features, w, b), labels)
        print(f'epoch {epoch + 1}, loss {float(train_l.mean()):f}')
```
