Shell executor 只是“让 GitLab Runner 在 GPU 宿主机上执行调度命令”，不代表你的训练环境要跑在宿主机 shell 里。训练仍然可以完全跑在 Docker 容器里。

你现在混淆的点主要是：Docker 环境 和 GitLab Runner executor 不是同一层东西。

## 三层概念

### 1. 你的开发 Docker

这是你平时写代码用的环境：

Cursor / VS Code

→ remote dev Docker / devcontainer

→ 你在里面写代码、调试、跑 notebook / train.py

这个 Docker 是“开发环境”。

### 2. GitLab Runner executor

这是 GitLab CI 任务被谁启动的问题：

GitLab Pipeline

→ GitLab Runner

→ executor 决定 job 命令在哪里执行

常见两种：

shell executor:

job script 直接在 GPU 服务器宿主机 shell 里执行

docker executor:

GitLab Runner 先启动一个 CI job 容器

job script 在这个 CI job 容器里执行

### 3. 训练 Docker

这是你真正跑深度学习代码的环境：

docker run --gpus all your-image python train.py

这个 Docker 是“训练环境”。

关键点是：

即使用 shell executor，训练也可以在 Docker 里跑。

---

## Shell executor 的真实含义


GitLab Runner 使用 shell executor

→ 在 GPU 宿主机执行几行 shell 命令

→ docker build ...

→ docker run --gpus all ...

→ 真正的 Python / PyTorch / CUDA 环境在容器里

也就是说：

宿主机 shell 只负责“启动 Docker”

训练环境仍然完全在 Docker 镜像里

所以你说“环境为了可控只在 Docker 中存在”，这和 shell executor 不冲突。

---

## 为什么我们之前推荐 shell executor？

因为你的 GPU CI 需求更像这样：

GitLab job

→ 到 GPU 服务器

→ 调用宿主机 docker

→ docker run --gpus all

→ 挂载 /mnt/data

→ 跑训练

这时 shell executor 很合适，因为它天然能访问 GPU 宿主机上的：

- `/usr/bin/docker`
- NVIDIA driver
- `/mnt/data`
- 本机 GPU
- 本机队列控制，例如 `concurrent = 1`

流程是：

GitLab

→ shell runner on GPU server

→ docker run --gpus all -v /mnt/data:/data your-image

→ python train.py

注意最后一层才是真正的训练。

---

## Docker executor 又是什么？

Docker executor 是另一种模式：

GitLab

→ docker runner on GPU server

→ GitLab Runner 自动启动一个 job container

→ 在这个 job container 里执行 script

也就是说 `.gitlab-ci.yml` 里会写：

train:

image: your-registry/your-dl-image:cu124

script:

- nvidia-smi

- python train.py

这里没有 `docker run`。

因为 job 本身已经在 Docker 容器里了。

---

## 两种模式最大的区别

### Shell executor 模式

job script 在宿主机 shell 执行

script 里手动 docker run

训练在你手动启动的训练容器里跑

示意：

train:

tags:

- linux

- shell

- main_gpu

script:

- docker build -t my-dl-job .

- docker run --rm --gpus all -v /mnt/data:/data my-dl-job python train.py

宿主机需要有：

- Docker
- NVIDIA driver
- NVIDIA Container Toolkit
- `/mnt/data`

但宿主机不需要有：

- PyTorch
- Python 依赖
- 你的 conda/venv

这些都在镜像里。

### Docker executor 模式

job script 已经在 CI job 容器里执行

script 里直接 python train.py

不要再 docker run

示意：

train:

image: registry.example.com/my-dl-image:cu124

tags:

- linux

- docker

- gpu

script:

- nvidia-smi

- python train.py

这要求 runner 本身配置好：

- Docker executor
- GPU pass-through
- NVIDIA Container Toolkit
- 可能还要配置 volumes，例如 `/mnt/data:/data`

---

## 你之前遇到的问题为什么会发生？

之前失败类似：

Preparing the "docker" executor

docker: command not found

原因是你用了 Docker executor，但是 job script 里面又想执行：

docker run ...

这就变成了：

GitLab Runner 先启动一个 job 容器

→ job 容器里面执行 docker run

也就是“容器里再调用 Docker”。

默认情况下，CI job 容器里面没有 Docker CLI，也不能直接控制宿主机 Docker daemon，所以会报：

docker: command not found

除非你额外搞：

- Docker-in-Docker
- 挂载 `/var/run/docker.sock`
- privileged runner
- NVIDIA runtime 透传
- 复杂的安全配置

这些可以做，但对你现在这个实验室 GPU 队列场景来说，复杂度明显更高。

---

## 所以到底还用不用 shell？

看你想要哪种 CI 形态。

### 情况 A：CI 里要 build 镜像，然后 docker run 训练

用 shell executor。

推荐：是

因为 job 需要直接调用宿主机 Docker：

docker build ...

docker run --gpus all ...

这就是我们现在 runbook 的设计。

### 情况 B：CI job 本身就是训练容器

可以不用 shell，用 Docker executor + GPU 配置。

可以：是

但需要额外配置 runner

这时 `.gitlab-ci.yml` 里一般不写 `docker run`，而是：

image: your-dl-image

script:

- python train.py

### 情况 C：你只是因为开发在 Docker 里，所以以为 CI 也必须 Docker executor

不一定。

开发在 Docker 里，只说明你的环境应该被镜像化。  
它不决定 GitLab Runner 必须用 Docker executor。

---

## 对你的 DL 项目，我更建议这样理解

你可以继续保持：

环境只存在于 Docker 镜像

同时使用：

shell executor

组合起来就是：

GitLab shell runner

→ docker build / docker pull

→ docker run --gpus all

→ 训练环境在容器里

这其实是很常见、也很清晰的 GPU CI 设计。

---

## 推荐选择

以你现在的实验室场景，我建议先用：

shell executor + docker run --gpus all

原因：

- 和现有 `lab-gpu-runbook` 一致
- 容易挂载 `/mnt/data`
- 容易控制每台 GPU 服务器排队
- 不需要在 Docker executor 里再套 Docker
- 环境依然完全由 Docker 镜像控制

你只需要记住一句：

shell executor 负责“在 GPU 服务器上启动训练容器”，不是负责“裸跑训练代码”。