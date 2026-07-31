---
title: "清除所有 Docker 数据后重装"
publish: false
tags: ["LLM","项目实践"]
---
# 清除所有 Docker 数据后重装

# 1. 停止 Docker 服务

sudo systemctl stop docker

# 2. 彻底清理 Docker 所有数据（强烈建议先备份你还想保留的东西）

sudo rm -rf /var/lib/docker

# 3. 重启 Docker 服务（会自动重新初始化 overlay2）

sudo systemctl start docker

# 4. 再次尝试重新运行 ollama 容器

sudo docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
