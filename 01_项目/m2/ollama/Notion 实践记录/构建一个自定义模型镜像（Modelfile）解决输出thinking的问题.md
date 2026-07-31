---
title: "构建一个自定义模型镜像（Modelfile）解决输出thinking的问题"
publish: false
tags: ["LLM","项目实践"]
---
# 构建一个自定义模型镜像（Modelfile）解决输出thinking的问题

## ✅ **推荐做法：构建一个自定义模型镜像（Modelfile）**

你的环境是通过 Docker 启动的 `ollama` 容器，因此推荐方法如下：

---

### ✅ Step 1: 在主机上准备 Modelfile

在 GCP VM 的某个目录里新建：

```bash
bash
复制编辑
mkdir ~/my-qwen-model && cd ~/my-qwen-model
nano Modelfile

```

内容如下：

```
Dockerfile
复制编辑
FROM qwen:7b

# 当遇到 <think> 字符串时就停止输出
PARAMETER stop "<think>"

# 模型默认行为
PARAMETER temperature 0.3
PARAMETER num_ctx 4096

# 系统指令，简化回答风格
SYSTEM "あなたは簡潔に答える日本語AIアシスタントです。説明・分析や<think>のような思考プロセスは出力しないでください。"

```

---

### ✅ Step 2: 进入运行中的 `ollama` 容器或主机运行构建命令

你现在用的是 Docker 启动 Ollama 的方式，所以有两种方法：

---

### ✅ 方法一：进入容器内部构建（推荐）

```bash
bash
复制编辑
sudo docker exec -it ollama bash

```

然后在容器内创建 `/root/my-qwen-model/Modelfile` 并运行：

```bash
bash
复制编辑
ollama create my-qwen -f /root/my-qwen-model/Modelfile

```

构建成功后，你可以：

```bash
bash
复制编辑
ollama run my-qwen

```

或者你的 Python 代码中将模型名改成 `"my-qwen"`。

---

### ✅ 方法二：把文件挂载进容器构建

也可以在主机构建后用 `-v` 挂载目录进去：

```bash
bash
复制编辑
sudo docker run -it --rm \
  --gpus=all \
  -v $(pwd):/root/model \
  -v ollama:/root/.ollama \
  -p 11436:11436 \
  ollama/ollama:latest \
  ollama create my-qwen -f /root/model/Modelfile

```

---

### ✅ Step 3: 修改你的调用逻辑（Python 脚本）

在 `batch_process_models_from_file.py` 中，把模型名由 `qwen:7b` 改为你自定义的：

```python
python
复制编辑
MODEL_NAME = "my-qwen"

```

如果你是按模型列表读取的，只要列表中写成 `my-qwen` 即可。
