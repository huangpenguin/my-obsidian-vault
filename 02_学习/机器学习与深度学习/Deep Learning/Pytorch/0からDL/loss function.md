---
title: "loss function"
publish: false
tags: ["机器学习"]
---
# loss function

## MSE

```python
def mean_squared_error(y, t):
    return 0.5 * np.sum((y-t)**2)
```

此时y是神经网络输出值，**MSE** 一般需要目标标签 `t` 是 one-hot 编码的向量

---

## Cross Entropy Error(Mini-batch)

一般t真实标签都是onehot，所有用这个而不是下面那段代码会比较多

```python
def cross_entropy_error(y, t):
    if y.ndim == 1:#when there is no batch 
		    t = t.reshape(1, t.size)
		    y = y.reshape(1, y.size)
		batch_size = y.shape[0]
		return -np.sum(t * np.log(y + 1e-7)) / batch_size
```

此时t一般是one-hot,也就是说只有预测值对应的yt会被取对数

## Cross Entropy Error(Mini-batch) when t is label(one number)

```python
def cross_entropy_error(y, t):
		if y.ndim == 1:
				t = t.reshape(1, t.size)
				y = y.reshape(1, y.size)
		batch_size = y.shape[0]
		return -np.sum(np.log(y[np.arange(batch_size), t] + 1e-7)) / batch_size
		
'''
example
t:[2,7,0,9,4]
np.arange(batch_size):[0,1,2,3,4]
y[np.arange(batch_size), t]: the network output for certain true label

'''
```
