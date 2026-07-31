---
title: "logit函数"
publish: false
tags: ["机器学习"]
---
# logit函数

Logit 是一种函数，常用于描述 Logistic 回归中的关系。Logit 函数实际上是 Sigmoid 函数的逆函数，它将一个概率值映射到实数空间。

### 1. **Logit 函数**

Logit 函数定义为：

$$
\text{logit}(p) = \log\left(\frac{p}{1-p}\right)
$$

其中：

- p 是某个事件发生的概率，0<p<1。
    
    0<p<10 < p < 1
    
- log 是自然对数函数。

Logit 函数的输出可以为任意实数（从 −∞-\infty−∞ 到 +∞+\infty+∞），这意味着 Logit 函数将概率空间 (0,1)(0, 1)(0,1) 映射到了整个实数空间 (−∞,+∞)(-\infty, +\infty)(−∞,+∞)。

### 2. **Logit 与 Logistic 回归的联系**

在 Logistic 回归中，假设函数 hθ(x)h_\theta(x)hθ​(x) 是使用 Sigmoid 函数定义的：

$$
\theta(x) = \frac{1}{1 + e^{-\theta^T x}}
$$

这表示给定特征 xxx 的情况下，样本属于正类（标签为 1）的概率。

为了理解 Logit 函数与 Logistic 回归的联系，我们可以对上面的 Sigmoid 函数取对数，并通过 Logit 函数得到以下关系：

首先，考虑 Sigmoid 函数输出 hθ(x)h_\theta(x)hθ​(x) 对应的对数几率（Odds Ratio）：

hθ(x)1−hθ(x)=11+e−θTx1−11+e−θTx=eθTx\frac{h_\theta(x)}{1 - h_\theta(x)} = \frac{\frac{1}{1 + e^{-\theta^T x}}}{1 - \frac{1}{1 + e^{-\theta^T x}}} = e^{\theta^T x}

1−hθ​(x)hθ​(x)​=1−1+e−θTx1​1+e−θTx1​​=eθTx

取自然对数（Logit 函数）：

log⁡(hθ(x)1−hθ(x))=θTx\log\left(\frac{h_\theta(x)}{1 - h_\theta(x)}\right) = \theta^T x

log(1−hθ​(x)hθ​(x)​)=θTx

因此，Logit 函数可以解释为 Logistic 回归模型中线性部分 θTx\theta^T xθTx 的输出。

### 3. **总结**

- **Logit 函数** 是 Sigmoid 函数的逆函数，它将概率值 p 转换为对数几率，即 log(1−pp​)。
    
    pp
    
    log⁡(p1−p)\log\left(\frac{p}{1-p}\right)
    
- 在 Logistic 回归中，Logit 函数的输出对应于模型的线性部分 θTx，这表明给定特征 x 后的对数几率。
    
    θTx\theta^T x
    
    xx
    
- Logistic 回归的模型通过将对数几率 θTx 通过 Sigmoid 函数映射回概率空间，实现二分类问题的概率预测。
    
    θTx\theta^T x
    

Logit 函数帮助我们理解 Logistic 回归的内部机制，特别是如何将线性回归的输出转换为概率值，并用于分类。
