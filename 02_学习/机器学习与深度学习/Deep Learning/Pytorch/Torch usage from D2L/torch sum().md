---
title: "torch.sum()"
publish: false
tags: ["机器学习"]
---
# torch.sum()

```python
#默认是求和所有元素
import torch

tensor = torch.tensor([[1, 2, 3],[4, 5, 6]])

# 对第一个维度求和（竖直方向）
result_dim0 = torch.sum(tensor, dim=0)
print(result_dim0)  # 输出：tensor([5, 7, 9])

# 对第二个维度求和（水平方向）
result_dim1 = torch.sum(tensor, dim=1)
print(result_dim1)  # 输出：tensor([ 6, 15])
```

---

### 保持原结构可以使用keep_dim

```python
X = torch.tensor([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
X.sum(0, keepdim=True)#[[5,7,9]] shape:(1,3)
X.sum(0)#[5,7,9]  shape:(3,)
```
