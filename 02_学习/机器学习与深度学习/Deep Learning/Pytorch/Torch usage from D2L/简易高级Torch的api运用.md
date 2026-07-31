---
title: "简易高级Torch的api运用"
publish: false
tags: ["机器学习"]
---
# 简易高级Torch的api运用

```python
import numpy as np
import torch
from torch.utils import data
from d2l import torch as d2l

true_w = torch.tensor([2, -3.4])
true_b = 4.2
features, labels = d2l.synthetic_data(true_w, true_b, 1000)

def load_array(data_arrays, batch_size, is_train=True):  #@save
    """构造一个PyTorch数据迭代器"""
    dataset = data.TensorDataset(*data_arrays)#张量解包
    return data.DataLoader(dataset, batch_size, shuffle=is_train)

batch_size=10
data_iter=load_array((features,labels),batch_size)
```

`TensorDataset` 是一个简单的数据集封装类，用于将特征和标签等张量组合成一个数据集对象。

`DataLoader` 是 PyTorch 中的一个数据加载器，它的作用是：

- **批量加载数据**：根据指定的 `batch_size`，将数据集分成多个小批量，方便模型逐批进行训练或推理。
- **打乱数据顺序**：如果设置 `shuffle=True`，则在每个 epoch 开始时打乱数据顺序，以提高模型的泛化能力。
- **并行加载数据**：通过设置 `num_workers` 参数，`DataLoader` 可以利用多线程并行加载数据，提升数据加载的效率。

```python
from torch import nn

#define model
net=nn.Sequential(nn.Linear(2,1))

#intialize parameter
net[0].weight.data.normal_(0,0.01)
net[0].bias.data.fill_(0)

#define loss and optim
loss=nn.MSELoss()
trainer=torch.optim.SGD(net.parameters(),lr=0.03)

#train
num_epochs=3
for epoch in range(num_epochs):
	for X,y in data_iter:
		l=loss(net(X),y)
		trainer.zero_grad()
		l.backward()
		trainer.step()
	l=loss(net(features),labels)#evaluate
```

更为高级的版本

```python
import torch
from torch import nn
import torch.optim as optim

# 设置随机种子以确保结果可重复
torch.manual_seed(42)

# 检查是否可以使用 GPU
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 定义模型类
class LinearRegressionModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(2, 1)  # 定义线性层，输入特征为2，输出特征为1

    def forward(self, x):
        return self.linear(x)

# 实例化模型并移动到计算设备上
net = LinearRegressionModel().to(device)

# 初始化参数
with torch.no_grad():
    net.linear.weight.normal_(0, 0.01)
    net.linear.bias.fill_(0)

# 定义损失函数和优化器
loss_fn = nn.MSELoss()
optimizer = optim.SGD(net.parameters(), lr=0.03)

# 定义训练参数
num_epochs = 3
batch_size = 10
data_iter = DataLoader(dataset, batch_size=batch_size, shuffle=True)  # 使用 DataLoader

# 训练过程
for epoch in range(num_epochs):
    net.train()  # 进入训练模式
    for X, y in data_iter:
        X, y = X.to(device), y.to(device)  # 将数据移到设备上
        pred = net(X)  # 前向传播
        loss = loss_fn(pred, y)  # 计算损失

        optimizer.zero_grad()  # 梯度清零
        loss.backward()  # 反向传播
        optimizer.step()  # 更新参数

    # 评估模型
    net.eval()  # 进入评估模式
    with torch.no_grad():  # 禁用梯度计算
        full_pred = net(features.to(device))
        train_loss = loss_fn(full_pred, labels.to(device))

    print(f'Epoch {epoch + 1}, Loss: {train_loss.item():.6f}')

```

在 PyTorch 中，`model.train()` 和 `model.eval()` 用于切换模型的**训练模式**和**评估模式**。这两种模式会影响模型中某些层的行为，尤其是 `Dropout` 和 `Batch Normalization` 层。以下是详细解释：

### 1. `model.train()` —— 训练模式

- **作用**：将模型切换到训练模式，意味着网络的某些层会以训练时的方式进行操作。
- **影响的层**：
    - **Dropout**：在训练模式下，`Dropout` 层会按照设定的概率随机“丢弃”一些神经元。这是一种正则化方法，用于防止过拟合。
    - **Batch Normalization**：在训练模式下，`Batch Normalization` 会使用当前 mini-batch 的均值和方差进行归一化操作，并不断更新其内部的运行均值和方差。这使得网络能够更好地适应当前的训练数据分布。

### 2. `model.eval()` —— 评估模式

- **作用**：将模型切换到评估模式，意味着网络在推理或评估时会以固定的方式进行操作。
- **影响的层**：
    - **Dropout**：在评估模式下，`Dropout` 层会被禁用，也就是说所有神经元都会被激活，不会有随机丢弃的操作。
    - **Batch Normalization**：在评估模式下，`Batch Normalization` 会使用训练过程中累计得到的整体均值和方差进行归一化，而不是使用当前 mini-batch 的统计信息。这可以确保模型在评估时表现稳定，避免由于小样本批次波动而导致的不一致性。

### 3. 为什么需要切换模式？

- **训练阶段**：`model.train()` 确保了 `Dropout` 和 `Batch Normalization` 按照训练时的设定来工作，从而达到模型正则化的效果，防止过拟合。
- **评估阶段**：`model.eval()` 确保模型在推理或评估时的行为与训练时一致，但避免使用训练中具有随机性的操作（如 Dropout），保证结果的稳定和可预测性。
