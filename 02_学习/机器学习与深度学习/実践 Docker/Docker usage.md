---
title: "Docker usage"
publish: false
tags: ["机器学习"]
---
# Docker usage

### 使用 Docker 跑 PyTorch 代码的简单步骤：

1. **安装 Docker**：
首先需要在本地安装 Docker。可以根据官方文档来安装：Docker 官方安装指南。
2. **创建 Dockerfile**：
在项目的根目录下创建一个 `Dockerfile`，里面定义你的环境配置。以下是一个简单的 PyTorch 环境的示例
    
    ```
    # 基于官方的 PyTorch 镜像
    FROM pytorch/pytorch:latest
    
    # 设置工作目录
    WORKDIR /app
    
    # 复制本地代码到容器内
    COPY . /app
    
    # 安装项目依赖
    RUN pip install -r requirements.txt
    
    # 暴露默认端口
    EXPOSE 8888
    
    # 运行程序
    CMD ["python", "your_script.py"]
    
    ```
    
3. **构建镜像**：
在终端运行以下命令来构建镜像：
    
    ```bash
    bash
    复制代码
    docker build -t pytorch-app .
    
    ```
    
4. **运行容器**：
使用以下命令启动容器：
    
    ```bash
    bash
    复制代码
    docker run --rm -it --gpus all pytorch-app
    
    ```
    
    这个命令会自动启用 GPU（前提是你机器有安装 NVIDIA Docker 插件）。
    

### Docker 优点：

- **环境一致性**：不管在哪台机器上，容器内的环境总是一致的。
- **免去反复配置**：你只需要配置一次 Dockerfile 。
- **可移植性**：你的代码和依赖都封装在一起，可以轻松共享给别人。

如果你有 GPU 需求，记得安装并配置好 `nvidia-docker`，确保容器可以利用 GPU 进行计算。

这也避免了反复在本地安装和卸载依赖的麻烦，特别是像 PyTorch 这种依赖库众多的框架。
