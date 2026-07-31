---
title: "#LogSumExp"
publish: false
tags: ["机器学习"]
---
# #LogSumExp

### 1. **Softmax 函数的本质**：

Softmax 函数用来将神经网络输出的分数（未规范化的预测）转换成概率。公式是：

y^j=exp⁡(oj)∑kexp⁡(ok)\hat{y}_j = \frac{\exp(o_j)}{\sum_k \exp(o_k)}y^j=∑kexp(ok)exp(oj)

这里，`o_j` 是网络的输出，而 `\hat{y}_j` 是转换后的概率值。

### 2. **数值稳定性问题：上溢**：

在计算 Softmax 时，如果 `o_k` 中的某些数特别大，计算 `exp(o_k)` 会变得非常大，超过计算机能处理的最大值，导致所谓的 **上溢**（overflow）。这时分子或者分母会变成无穷大（`inf`），计算结果会出错，得到的是 `inf`、`nan`（不是数字）等。

### 3. **解决方法：减去最大值**：

为了避免这种上溢问题，可以在计算 Softmax 之前，把所有 `o_k` 减去最大值 `max(o_k)`。这不会改变 Softmax 的结果，但可以避免计算过大的指数。

换句话说，我们通过：

exp⁡(oj−max⁡(ok))∑kexp⁡(ok−max⁡(ok))\frac{\exp(o_j - \max(o_k))}{\sum_k \exp(o_k - \max(o_k))}

∑kexp(ok−max(ok))exp(oj−max(ok))

来代替原来的公式。这样，即使 `o_k` 很大，减去最大值后计算出的指数也不会太大。

### 4. **数值稳定性问题：下溢**：

虽然我们避免了上溢问题，但有时 `o_j - \max(o_k)` 可能是一个很大的负数，这样 `exp(o_j - \max(o_k))` 会接近 0，导致 **下溢**（underflow）。这会让结果接近 0，最后导致 `log(0)` 的计算变为 `-inf`，引发数值问题。

### 5. **结合 Softmax 和交叉熵来解决问题**：

在计算交叉熵损失时，我们最终要用到 `log(Softmax)`。聪明的方法是将这两个步骤合并，从而避免上述问题。通过一些数学转换，公式变为：

log⁡(y^j)=oj−max⁡(ok)−log⁡(∑kexp⁡(ok−max⁡(ok)))\log(\hat{y}_j) = o_j - \max(o_k) - \log\left(\sum_k \exp(o_k - \max(o_k))\right)

log(y^j)=oj−max(ok)−log(k∑exp(ok−max(ok)))

这样，我们直接使用 `o_j - \max(o_k)` 而不需要计算 `exp()` 和 `log()` 这些容易出现问题的操作，避免了数值溢出的问题。

### 6. **LogSumExp 技巧**：

这种合并 Softmax 和交叉熵的做法类似于一个叫做 "LogSumExp" 的技巧，它在避免数值问题的同时提高了计算的稳定性和效率。
