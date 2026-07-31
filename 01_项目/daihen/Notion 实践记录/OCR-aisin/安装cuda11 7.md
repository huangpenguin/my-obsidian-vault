---
title: "安装cuda11.7"
publish: false
tags: ["OCR","项目实践"]
---
# 安装cuda11.7

## 如果你要使用 CUDA 11.7，需要做两件事：

---

### ✅ Step 1：**系统层安装 CUDA 11.7 的运行时支持**

你可以安装 CUDA 11.7 的 runtime（不需要 full toolkit），用以下命令（Ubuntu 系统）：

```bash
bash
复制编辑
sudo apt update
sudo apt install libcudnn8 libcudnn8-dev libnccl2 libnccl-dev
sudo apt install nvidia-cuda-toolkit  # 可选，如果你希望有 `nvcc`

```

或者你也可以直接用 NVIDIA 官方安装方法安装 CUDA 11.7 runtime DEB 包：

参考官方文档：

[https://developer.nvidia.com/cuda-11-7-0-download-archive](https://developer.nvidia.com/cuda-11-7-0-download-archive)

---

### ✅ Step 2：**Conda 环境中安装 Paddle 11.7 对应版本**

使用以下命令：

```bash

pip install paddlepaddle-gpu==2.4.2.post117 -f https://www.paddlepaddle.org.cn/whl/mkl/stable.html

```

这是 PaddlePaddle 对应 **CUDA 11.7** 的稳定版本，官方推荐这样装。

---

## 🧪 验证是否正确

然后在虚拟环境中运行：

```bash
bash
复制编辑
python -c "import paddle; print(paddle.is_compiled_with_cuda()); print(paddle.device.cuda.device_count())"

```

---

## 🔁 可选：切换系统 CUDA 默认版本（仅当你有多个 CUDA 安装）

你可以用以下方法切换默认 CUDA 链接：

```bash
bash
复制编辑
sudo update-alternatives --install /usr/local/cuda cuda /usr/local/cuda-11.7 110
sudo update-alternatives --config cuda

```

选择 `cuda-11.7` 所在目录即可。

如果你不确定是否已安装 CUDA 11.7 runtime，可以执行：

```bash
ls /usr/local/ | grep cuda
nvcc --version

```
