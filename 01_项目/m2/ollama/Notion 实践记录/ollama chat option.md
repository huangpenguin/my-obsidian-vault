---
title: "ollama.chat option"
publish: false
tags: ["LLM","项目实践"]
---
# ollama.chat option

| `num_predict` | 最大生成 token 数，防止一次生成太多 |
| --- | --- |
| `temperature` | 控制输出随机性 |
| `top_k` | 限制选择前 k 个词 |
| `top_p` | 只选取累计概率为 top_p 的词 |
| `repeat_penalty` | 惩罚重复词 |
| `batch_size` | 一次生成时使用的 token 数量（小一点更省显存） |
