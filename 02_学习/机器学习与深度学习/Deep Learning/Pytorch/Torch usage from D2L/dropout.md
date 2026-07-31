---
title: "dropout"
publish: false
tags: ["机器学习"]
---
# dropout

---

```python
#model define

dropout1,dropout2=0.2,0.5

class Net(nn.module):
	def __init__(self,num_inputs,num_outputs,num_hiddens1.num_hiddens2,
	             is_training=True):
	    super(Net,self).__init__()
	    self.num_inputs=num_inputs
	    self.training=is_training
	    self.lin1=nn.Linear(num_imputs,num_hiddens1)
	    self.lin2=nn.Linear(num_hiddens1,num_hiddens2)
	    self.lin3=nn.Linear(num_hiddens2,num_outsputs)
	    self.relu=nn.Relu()
	def forward():
		H1 = self.relu(self.lin1(X.reshape((-1,self.num_inputs))))
		if self.training == True:
			H1=dropout_layer(H1,dropout1)
		H2=self.relu(self.lin2(H1))
		if self.training == True:
			H2=dropout_layer(H2,dropout2)
		out=self.lin3(H2)
		return out
		
net=Net(num_inputs,num_outputs,num_hidddens1,num_hiddens2)
```

```python
#model define using api

import torch
import torch.nn as nn

dropout1, dropout2 = 0.2, 0.5

class Net(nn.Module):  # 注意要用 nn.Module（大写 M）
    def __init__(self, num_inputs, num_outputs, num_hiddens1, num_hiddens2, is_training=True):
        super(Net, self).__init__()
        self.num_inputs = num_inputs
        self.training = is_training

        # 定义三层的线性层
        self.lin1 = nn.Linear(num_inputs, num_hiddens1)
        self.lin2 = nn.Linear(num_hiddens1, num_hiddens2)
        self.lin3 = nn.Linear(num_hiddens2, num_outputs)

        # 激活函数
        self.relu = nn.ReLU()
        
        # Dropout 层
        self.dropout1 = nn.Dropout(dropout1)
        self.dropout2 = nn.Dropout(dropout2)

    def forward(self, X):
        # 输入经过第一层线性变换和 ReLU 激活函数
        H1 = self.relu(self.lin1(X.reshape((-1, self.num_inputs))))
        if self.training:  # 如果是训练模式，使用 Dropout
            H1 = self.dropout1(H1)
        
        # 第二层线性变换和 ReLU 激活函数
        H2 = self.relu(self.lin2(H1))
        if self.training:  # 如果是训练模式，使用 Dropout
            H2 = self.dropout2(H2)
        
        # 输出层，不需要激活函数，因为它是最后一层
        output = self.lin3(H2)
        return output

```
