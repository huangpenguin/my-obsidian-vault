---
title: "preprocess"
publish: false
tags: ["机器学习"]
---
# preprocess

```python
train_data = pd.read_csv(download('kaggle_house_train'))
test_data = pd.read_csv(download('kaggle_house_test'))
print(train_data.shape)
print(test_data.shape)
```

这段代码加载了训练集和测试集，并打印它们的形状。这是检查数据是否成功加载的第一步。

### 2. 打印数据的一部分

```python
print(train_data.iloc[0:4, [0, 1, 2, 3, -3, -2, -1]]) # 第二维是特征
```

打印训练集中的前四行的某些特征，以便对数据有一个初步的了解。

### 3. 连接训练集和测试集

```python
# 处理掉不需要的列(ID)
all_features = pd.concat((train_data.iloc[:, 1:-1], test_data.iloc[:, 1:])) # 行合并
```

这里的 `pd.concat` 函数将训练集和测试集的特征（去掉 ID 和目标变量 `SalePrice`）纵向合并。这样可以在后续处理时对两个数据集进行统一的特征工程。
