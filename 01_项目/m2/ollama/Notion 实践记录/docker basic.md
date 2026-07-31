---
title: "docker basic"
publish: false
tags: ["LLM","项目实践"]
---
# docker basic

### **基本操作（逐个进行）**

```bash
# 停止指定容器
docker stop <容器ID或名称>

# 删除指定容器
docker rm <容器ID或名称>
```

---

### 🚀 **批量操作（高效清理）**

```bash

# 停止所有运行中的容器
docker stop $(docker ps -q)

# 删除所有容器（包括已停止）
docker rm $(docker ps -aq)

# 或者：使用 -f 强制停止并删除
docker rm -f $(docker ps -aq)

```

---

### 📌 **注意事项**

1. `q`：只显示容器 ID（quiet 模式）
2. `a`：包括已停止的容器
3. 数据持久化用户请注意：删除容器时若有挂载数据卷可能造成数据丢失！

```bash

# 删除容器及其挂载的卷
docker rm -fv <容器ID或名称>

```

---

### 🔍 **操作前的安全确认**

```bash
bash
复制编辑
docker ps         # 查看正在运行的容器
docker ps -a      # 查看所有容器（包括停止的）
docker ps -q      # 查看运行中容器的 ID
docker ps -aq     # 查看所有容器的 ID（用于批量处理）

```

---

### 🧰 **其他常用容器操作**

```bash
bash
复制编辑
docker start <容器名>        # 启动已停止的容器
docker restart <容器名>      # 重启容器
docker logs <容器名>         # 查看容器日志
docker exec -it <容器名> /bin/bash  # 进入运行中的容器交互式终端

```

---

### 🧹 **通过筛选状态清理容器**

```bash
bash
复制编辑
# 删除所有已退出的容器
docker rm -f $(docker ps -aq --filter "status=exited")

```

---

### 💾 **管理和查看数据卷**

```bash
bash
复制编辑
# 删除名为 ollama 的卷
sudo docker volume rm ollama

# 查看 ollama 卷的数据占用空间
sudo du -sh /var/lib/docker/volumes/ollama/_data

```

---

### 🛠️ **Docker 服务管理（系统级）**

```bash

# 重启 Docker 服务
sudo systemctl restart docker

# 启动 Docker 服务（开机不自动）
sudo systemctl start docker

# 设置 Docker 开机自启
sudo systemctl enable docker

```

---

### docker过大

找到了！问题就是 `/var/lib/docker` 占用了83GB空间！这就是磁盘满的根本原因。

## 立即清理Docker数据

### 1. 先检查Docker状态

```bash
# 查看当前Docker容器和镜像
sudo docker ps -a
sudo docker images

```

### 2. 彻底清理Docker数据

```bash
# 停止所有Docker服务
sudo systemctl stop docker

# 删除整个Docker数据目录（这会删除所有容器、镜像、网络等）
sudo rm -rf /var/lib/docker

# 重启Docker服务
sudo systemctl start docker

# 检查Docker状态
sudo systemctl status docker
```

### 3. 验证清理效果

```bash
# 检查磁盘空间
df -h

# 检查Docker目录大小
sudo du -sh /var/lib/docker

```

### 4. 重新创建Ollama容器

```bash
# 重新拉取并运行Ollama容器
sudo docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:latest

# 验证容器运行
sudo docker ps

```

## 为什么Docker占用这么多空间？

从你的模型列表看，你试图运行的都是超大型模型：

- `qwen2.5vl:32b` - 32B参数模型
- `qwen2.5vl:32b-q8_0` - 32B参数模型（8位量化）
- `llama3.1:70b` - 70B参数模型
- `llama3.3:70b` - 70B参数模型

这些模型每个都可能占用10-40GB空间，即使设置了删除，但下载过程中会产生大量临时文件。

## 建议的后续操作

1. **立即执行上面的Docker清理命令**
2. **修改模型列表使用更小的模型**：

```bash
# 创建一个小模型测试列表
cat > test_addition/model_list_small.txt << 'EOF'
gemma2:2b
qwen2.5:3b
llama3.2:3b
phi3.5:3.8b-mini-instruct-q4_0
EOF

```

1. **测试系统是否正常**：

```bash
# 使用小模型列表测试
uv run test_addition/batch_process_models_from_file.py -f test_addition/model_list_small.txt

```
