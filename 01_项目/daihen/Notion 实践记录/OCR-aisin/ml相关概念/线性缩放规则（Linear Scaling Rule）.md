---
title: "线性缩放规则（Linear Scaling Rule）"
publish: false
tags: ["OCR","项目实践"]
---
# 线性缩放规则（Linear Scaling Rule）

**📌 解释：**

当你使用**更大的 batch size**（例如从 32 增加到 128）时，为了训练仍然稳定且有效，通常需要 **等比例增加学习率（learning rate）**，这就是“线性缩放规则”。

**💡 公式：**

```
lr_new = lr_base * (batch_size_new / batch_size_base)
```

**📘 举例：**

如果你原来设置是：

```
batch_size = 32
learning_rate = 0.001
```

当你将 `batch_size` 改为 `128`，按线性缩放规则：

```
lr = 0.001 * (128 / 32) = 0.004
```

**🔧 适用情况：**

- 通常用于多 GPU 训练（batch 总量会增大）
- 通常用于 ImageNet、NLP 大模型预训练等
