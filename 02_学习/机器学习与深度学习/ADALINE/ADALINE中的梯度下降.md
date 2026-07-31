---
title: "ADALINE中的梯度下降"
publish: false
tags: ["机器学习"]
---
# ADALINE中的梯度下降

### 梯度下降的数学原理

梯度下降的基本思想是通过计算损失函数相对于模型参数的导数（即梯度），并沿着负梯度方向更新参数，以最小化损失函数。在 ADALINE 中，损失函数通常是均方误差 (MSE)，定义如下：

$$
\text{MSE} = \frac{1}{2} \sum_{i=1}^{m} (t_i - y_i)^2
$$

其中：

- m 是训练样本的数量。
- ti 是第 i 个样本的真实输出（目标输出）。
- yi 是第 i 个样本的预测输出。

对于每个权重 wj，损失函数的梯度是：

$$
\frac{\partial \text{MSE}}{\partial w_j} = -\sum_{i=1}^{m} (t_i - y_i) x_{ij}
$$

而偏置的梯度是：

$$
\frac{\partial \text{MSE}}{\partial b} = -\sum_{i=1}^{m} (t_i - y_i)
$$

### 梯度下降在代码中的实现

在实际代码实现中，梯度下降算法体现在以下两个关键更新步骤中：

### 1. 误差计算

```python
errors = y - output
```

这里，`errors` 是一个向量，包含了每个训练样本的误差。它表示的是预测值 `output` 与实际标签 `y` 之间的差距 t−y。这正是梯度计算中的一部分。

### 2. 权重更新

```python
self.weights += self.learning_rate * X.T.dot(errors)
```

这一行代码实现了基于梯度的权重更新。下面解释这行代码是如何进行梯度下降的：

- `X.T.dot(errors)`：这是输入矩阵 XXX 的转置与误差向量 `errors` 的矩阵乘法。假设 X 的维度是 m×n（即 m 个样本和 n 个特征），而 `errors` 是 m×1 的列向量。矩阵乘法结果是一个 n×1 的向量，每个元素表示所有样本在特定特征上的误差累积，即：

$$
\text{grad}_w = X^T \cdot (t - y)
$$

这里的 `grad_w` 表示每个特征对损失的导数（即梯度），这是一个 n 维的向量。

- `self.learning_rate * X.T.dot(errors)`：乘以学习率 η 控制更新步长，避免权重变化过快或过慢。
- `self.weights += ...`：这一行代码将梯度乘以学习率后加到当前权重上，这是梯度下降的核心步骤。它沿着负梯度方向调整权重，以减小误差。

### 3. 偏置更新

```python
self.bias += self.learning_rate * errors.sum()
```

- `errors.sum()`：计算所有误差的总和，即
- $\sum_{i=1}^{m} (t_i - y_i)$，这是偏置的梯度。
- `self.learning_rate * errors.sum()`：通过学习率调整步长。
- `self.bias += ...`：更新偏置，减小误差。

### 总结

虽然在代码中没有显式地计算梯度，但是权重和偏置的更新实际上已经包含了梯度下降的步骤：

1. `errors = y - output` 计算了误差，它反映了损失函数相对于输出的梯度。
2. `self.weights += self.learning_rate * X.T.dot(errors)` 使用误差来调整权重，这一矩阵运算相当于计算了损失函数相对于每个权重的梯度。
3. `self.bias += self.learning_rate * errors.sum()` 使用误差的总和来调整偏置，这相当于计算了损失函数相对于偏置的梯度。
