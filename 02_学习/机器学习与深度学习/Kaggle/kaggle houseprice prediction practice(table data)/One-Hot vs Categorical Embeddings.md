---
title: "One-Hot vs Categorical Embeddings"
publish: false
tags: ["机器学习"]
---
# One-Hot vs Categorical Embeddings

简单来说，当一个特征（某一列）的分类数太多时，使用one-hot会显著增大维度，需要避开使用。

---

**Categorical Embeddings**（分类嵌入）是一种用于将分类变量转换为连续数值向量的技术，广泛应用于深度学习模型，尤其是当处理高维分类特征时，如推荐系统和自然语言处理。相比于传统的 **one-hot encoding** 或 **Ordinal Encoding**，embedding 方法能够更有效地表示高基数分类变量，并且可以捕捉到类别之间的语义关系。

### 什么是 Categorical Embeddings？

Categorical Embeddings 是将每个类别映射为一个低维的连续向量，这个向量可以通过模型在训练过程中学习出来。它的核心思想是将分类变量的每一个类别用一个向量表示，而这些向量的值可以通过模型训练动态更新。每个类别向量不仅仅是独立的数值，还包含了类别之间的隐含语义信息。

### 如何工作？

1. **初始化嵌入矩阵**：对于每个分类特征，首先初始化一个嵌入矩阵。矩阵的行数为类别数量，列数为嵌入维度（这个维度一般远小于 one-hot encoding 的维度）。
2. **映射类别到嵌入空间**：每个类别通过嵌入矩阵转换为一个连续向量（embedding）。这些向量会在训练过程中根据目标变量进行调整。
3. **学习嵌入向量**：嵌入向量通过深度学习模型（如神经网络）进行训练。神经网络可以根据最终的损失函数对这些嵌入向量进行优化，使得这些向量能够更好地捕捉类别的特性和类别间的关系。

### Categorical Embeddings 的优势

1. **低维表示**：相比于独热编码的高维度表示，embedding 向量维度要小得多。这减少了内存消耗并加快了计算速度。
2. **捕捉类别之间的关系**：独热编码不会捕捉类别之间的关系，而 embedding 向量可以通过模型学习到类别之间的相似性或相关性。
3. **处理高基数分类变量**：对于有许多不同类别的特征（如邮政编码、产品 ID），嵌入表示能够更有效地处理这些高基数变量，而不增加过多维度。
4. **可迁移性**：经过训练的 embedding 向量可以用作其他任务的输入，甚至在不同模型之间共享。

### 示例

***假设我们有一个分类变量“城市”，该变量有 100 个不同的值。如果使用 one-hot encoding，则我们将得到一个 100 维的向量，其中只有一个位置为 1，其他位置为 0。而如果我们使用 embedding 表示，我们可能将“城市”映射到一个 5 维的向量空间，每个城市对应的向量是通过训练学习得来的。***

在代码中，这可以通过神经网络框架（如 TensorFlow 或 PyTorch）轻松实现。例如，使用 Keras 进行分类嵌入：

```python
python
复制代码
from keras.layers import Input, Embedding, Flatten, Dense
from keras.models import Model

# 假设'city'有100个不同的类别，将其映射到5维向量
input_city = Input(shape=(1,))
embedding_city = Embedding(input_dim=100, output_dim=5)(input_city)
flat_city = Flatten()(embedding_city)

# 可以将嵌入后的向量与其他特征合并，输入到深度学习模型中
# 然后再添加其他层...
dense_layer = Dense(10, activation='relu')(flat_city)
output = Dense(1, activation='linear')(dense_layer)

model = Model(inputs=[input_city], outputs=[output])
model.compile(optimizer='adam', loss='mse')

```

在这个例子中，`city` 的分类变量被映射到了一个 5 维的嵌入空间，并作为神经网络的输入参与训练。

### 何时使用 Categorical Embeddings？

1. **深度学习模型**：嵌入通常用于深度学习模型，尤其是神经网络，而不是像 XGBoost 或随机森林这样对独热编码敏感的传统机器学习模型。
2. **高基数分类变量**：当分类变量的取值非常多时，使用 embedding 是非常有效的，尤其是在推荐系统中。
3. **需要捕捉类别之间的关系**：如果你认为不同类别之间有某种潜在的关系或相似性，embedding 可以自动发现并利用这些信息。
