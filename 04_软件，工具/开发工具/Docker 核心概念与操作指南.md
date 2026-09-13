## 1. 为什么需要 Docker？

- **解决依赖冲突：** 防止 Python 版本、PyTorch、CUDA 或外部工具（如 ffmpeg）版本不匹配引发的错误。
    
- **隔离项目环境：** 不污染宿主机操作系统，支持在同一台机器上安全构建与删除不同的项目环境。
    
- **确保实验可复现：** 方便轻松复现他人发布的模型、论文源码或基准测试（Benchmark）环境。
    

---

## 2. 核心概念

- **镜像（Image）：** 环境的“模具”或模板，打包了运行程序所需的全部文件、依赖库及配置。
    
- **容器（Container）：** 基于镜像创建并运行的“成品”，是一个相互隔离的实际运行空间。
    

---

## 3. 常用基础命令

### 镜像获取与查看

- `docker pull <image_name>:<tag>` : 从 Docker Hub 等仓库下载指定镜像
    
- `docker images` : 查看已下载的本地镜像列表
    

### 容器运行与交互

- `docker run --rm <image_name>` : 创建并运行容器（容器退出后自动删除）
    
- `docker run -it --rm <image_name>` : 分配伪终端并保持标准输入打开，实现交互式操作
    
- `docker run -it --rm <image_name> bash` : 进入容器内部的 Linux CLI 终端环境
    

---

## 4. 常用运行选项

- **端口映射 (`-p`)**
    
    - 将宿主机的端口连接到容器内部端口。
        
    - 示例：`docker run -it --rm -p 8080:8000 python:3.10.20-bookworm bash`
        
- **GPU 挂载 (`--gpus`)**
    
    - 允许容器调用宿主机的 NVIDIA GPU 资源。
        
    - 示例：`docker run --rm --gpus all nvidia/cuda:13.3.1-base-ubuntu22.04 nvidia-smi`
        

---

## 5. Docker Compose 与数据卷共享（Bind Mount）

用于在容器销毁后保留数据，或实现宿主机与容器之间的文件实时同步。

YAML

```
# compose.yaml 基础配置示例
services:
  app:
    image: python:3.10.20-bookworm
    working_dir: /app
    volumes:
      - .:/app  # 将宿主机的当前目录挂载到容器的 /app 目录
```

- **运行命令：** `docker compose run --rm app bash`
    

---

## 6. 使用 Dockerfile 构建自定义镜像

通过 Dockerfile 记录并自动化构建自定义环境的步骤。

### Dockerfile 示例

Dockerfile

```
FROM python:3.12.14-slim-bookworm
WORKDIR /app

# 安装 Linux 系统级依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends ffmpeg && \
    rm -rf /var/lib/apt/lists/*

# 安装 Python 依赖包
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制源码并设置默认启动命令
COPY make_video.py .
CMD ["python", "make_video.py"]
```

### 构建与执行

- `docker compose build` : 根据 Dockerfile 构建镜像
    
- `docker compose run --rm app` : 使用构建好的镜像运行容器