---
title: "drop_last"
publish: false
tags: ["OCR","项目实践"]
---
# drop_last

**📌 解释：**

在使用 `DataLoader` 时，如果 **数据集大小不能被 batch size 整除**，最后一个 batch 会比其他的“小一点”。

设想你有 105 个样本，batch size 是 32：

- 正常 batch：32, 32, 32
- 最后一个：**只有 9 个**

如果你设置：

```python
drop_last=True

```

最后这个不完整的 batch 会被 **丢弃（drop）**。

**🧠 为什么需要这个？**

某些模型（如分布式训练、BN层）可能要求每个 batch 尺寸一致，否则出错或效果不佳。
