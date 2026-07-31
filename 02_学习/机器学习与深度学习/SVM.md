---
title: "SVM"
publish: false
tags: ["机器学习"]
---
# SVM

支持向量机（Support Vector Machine, SVM）是一种用于分类和回归分析的监督学习模型。在分类问题中，SVM 的主要思想是找到一个能够将不同类别的数据点分开并且间隔最大的超平面。SVM 通过一个称为支持向量的子集来定义边界，这些支持向量是距离分割超平面最近的数据点。

### 1. 基本概念

假设有一个数据集 $\{(\mathbf{x}_i, y_i)\}_{i=1}^n$，其中 xi∈Rd 是 d 维特征空间中的输入向量，yi∈{−1,1} 是对应的类别标签。SVM 试图找到一个超平面将这两类数据分开。这个超平面可以表示为：

$$
\mathbf{w} \cdot \mathbf{x} - b = 0
$$

其中 $\mathbf{w} \in \mathbb{R}$ 是法向量，决定了超平面的方向，b 是偏置项。

### 2. 分类间隔

分类间隔（margin）定义为距离超平面最近的点的距离。在几何上，间隔可以表示为：

$$
\text{margin} = \frac{2}{\|\mathbf{w}\|}
$$

为了最大化间隔，SVM 寻求最小化 ∥w∥，同时确保所有数据点被正确分类。对于每个样本点 (xi,yi)，我们有以下约束条件：

$$
y_i (\mathbf{w} \cdot \mathbf{x}_i - b) \geq 1, \quad \forall i
$$

### 3. 优化问题

SVM 的目标是最大化间隔，即最小化 ∥w∥。因此可以将其表述为以下优化问题：

$$
\min_{\mathbf{w}, b} \frac{1}{2} \|\mathbf{w}\|^2
$$

$$
\text{subject to } y_i (\mathbf{w} \cdot \mathbf{x}_i - b) \geq 1, \quad \forall i
$$

### 4. 拉格朗日对偶问题

为了求解这个优化问题，引入拉格朗日乘子 αi≥0，形成拉格朗日函数：

$$
L(\mathbf{w}, b, \alpha) = \frac{1}{2} \|\mathbf{w}\|^2 - \sum_{i=1}^n \alpha_i \left[y_i (\mathbf{w} \cdot \mathbf{x}_i - b) - 1\right]
$$

通过对 w 和 b 求偏导并令其为零，我们可以得到对偶问题：

$$
\max_{\alpha} \sum_{i=1}^n \alpha_i - \frac{1}{2} \sum_{i=1}^n \sum_{j=1}^n \alpha_i \alpha_j y_i y_j \mathbf{x}_i \cdot \mathbf{x}_j
$$

$$
\text{subject to } \sum_{i=1}^n \alpha_i y_i = 0, \quad \alpha_i \geq 0, \quad \forall i
$$

### 5. 核方法

当数据集是***非线性可分***时，可以使用核方法（Kernel Method）将数据映射到一个高维空间，使其线性可分。核函数 K(xi,xj)计算在高维空间中对应点的内积，从而避免显式计算高维映射。常用的核函数有：

- 线性核：$K(\mathbf{x}_i, \mathbf{x}_j) = \mathbf{x}_i \cdot \mathbf{x}_j$
- 多项式核：$K(\mathbf{x}_i, \mathbf{x}_j) = (\mathbf{x}_i \cdot \mathbf{x}_j + c)^d$
- 高斯径向基核（RBF 核）：$K(\mathbf{x}_i, \mathbf{x}_j) = \exp\left(-\frac{\|\mathbf{x}_i - \mathbf{x}_j\|^2}{2\sigma^2}\right)$

### 6. 理解公式

1. **超平面方程 w⋅x−b=0**：这是 SVM 分类的基础，用于分隔不同类别的数据点。法向量 w 的方向和大小决定了超平面的方向和距离原点的远近。
2. **分类间隔公式 \frac{2}{\|\mathbf{w}\|}**：这个公式表示两类数据点之间的“空白区域”的大小。SVM 通过最大化间隔来提高分类的鲁棒性和泛化能力。
3. **拉格朗日函数和对偶问题**：这些公式用于将原始的约束优化问题转换为对偶问题，便于用现有的优化算法求解。拉格朗日乘子 αi\alpha_iαi​ 反映了每个数据点对最终分类决策的影响。
4. **核函数 K(xi,xj)**：核方法允许 SVM 在高维空间中寻找线性可分的超平面，解决非线性分类问题。核函数的选择和参数调优对模型性能有直接影响。

通过理解这些公式和推导过程，可以深入掌握 SVM 的原理以及在不同场景下的应用。
