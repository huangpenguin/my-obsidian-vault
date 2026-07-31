---
title: "scikit-learn"
publish: false
tags: ["机器学习"]
---
# scikit-learn

- 大多数转换器期望数据存储在二维阵列

- **二维数组的结构**：Scikit-learn 的转换器期望的数据为形如 `(n_samples, n_features)` 的二维数组，其中：
    - `n_samples` 表示样本的数量，即行数。
    - `n_features` 表示每个样本的特征数量，即列数。
- **数据格式**：常见的数据格式包括 NumPy 数组 (`numpy.ndarray`)、Pandas 数据框 (`pandas.DataFrame`)，以及 Scipy 稀疏矩阵 (`scipy.sparse`)。这些格式的共同点是能够以二维结构存储数据。

### 举例

假设你有一个包含 3 个样本和 2 个特征的数据集，数据可以用如下的二维数组表示：

```python
import numpy as np

# 3 samples, 2 features
X = np.array([[1, 2],
              [3, 4],
              [5, 6]])

print(X.shape)  # 输出 (3, 2)

```

这里，`X` 是一个形状为 `(3, 2)` 的二维数组。

### 常见问题

1. **单个特征的处理**：如果你的数据集只有一个特征，Scikit-learn 仍然要求数据为二维格式，例如 `(n_samples, 1)`。如果提供一维数组 `X = [1, 2, 3]`，会导致错误，正确的做法是将其转化为二维数组：
    
    ```python
    X = np.array([1, 2, 3]).reshape(-1, 1)#转换为形状为 (3, 1) 的二维数组
    X[:,np.newaxis]#另一个类似效果的函数
    
    ```
    
2. **单个样本的处理**：如果只有一个样本但有多个特征，形状应为 `(1, n_features)`。

### 其他情况

虽然大多数 Scikit-learn 的转换器期望数据为二维数组，但某些特殊情况可能接受一维数组（如一些目标变量 `y` 的表示），或者需要更高维度的数据（如某些时间序列或图像处理任务）。然而，对于典型的机器学习任务，二维数组的表示是标准和普遍的。
