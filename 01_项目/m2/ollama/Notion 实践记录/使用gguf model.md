---
title: "使用gguf model"
publish: false
tags: ["LLM","项目实践"]
---
# 使用gguf model

## 用 Ollama 自定义模型功能加载 Hugging Face 模型

Ollama 支持通过自定义 `Modelfile` 把 Hugging Face 上的模型拉下来并包装为 Ollama 模型。

### ✳️ 步骤如下：

### 1. **找到模型的 GGUF 格式版本**

Ollama **只能加载 GGUF 格式** 的量化模型，而 Hugging Face 上的模型很多是原始 PyTorch 或 safetensors 格式。

👉 去 **TheBloke** 的 Hugging Face 页面搜索是否有这个模型的 **Qwen-32B GGUF 版本**，例如你要找：

```

ELYZA-Thinking-1.0-Qwen-32B-GGUF

```

如果没有 GGUF 版本，无法直接用于 Ollama，你可以考虑使用 `text-generation-webui` 或 `vllm` 部署该模型（见方法二）。

### 2. **写一个自定义的 Modelfile**

如果你找到了 GGUF 模型，可以创建如下结构：

```bash
elyza-qwen-32b/
├── Modelfile
└── model.gguf  # 下载好的模型文件

```

`Modelfile` 示例：

```
FROM llama.cpp
PARAMETER num_ctx 32768
PARAMETER temperature 0.7
PARAMETER top_k 40
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.1

```

将 `model.gguf` 放在同目录，然后构建模型：

```bash
ollama create elyza-qwen-32b -f Modelfile
```

启动测试：

```bash
ollama run elyza-qwen-32b
```
