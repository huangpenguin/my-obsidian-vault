---
title: "shuffle"
publish: false
tags: ["OCR","项目实践"]
---
# shuffle

**📌 解释：**

是否在每个 epoch 开始前，对数据进行**打乱（shuffle）**。

**💡 默认行为：**

- `shuffle=True`：每次训练都随机打乱数据顺序 → 更好泛化
- `shuffle=False`：数据固定顺序 → 可重复性好，但容易过拟合

**📘 示例：**

```python

dataloader = DataLoader(dataset, batch_size=32, shuffle=True)

```

适合训练阶段用，**测试/验证阶段必须设为 `False`**！
