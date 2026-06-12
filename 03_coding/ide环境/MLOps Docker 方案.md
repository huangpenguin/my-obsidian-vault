## 检查服务器挂载状况
由于gpu服务器使用的是autofs的形式，随用随连，只能通过下面方式查看到
```
(base) ➜  ~ mount | grep nfs
nfsd on /proc/fs/nfsd type nfsd (rw,relatime)
/etc/auto.nfs on /mnt type autofs (rw,relatime,fd=6,pgrp=3495,timeout=300,minproto=5,maxproto=5,indirect,pipe_ino=68827)
(base) ➜  ~ cat /etc/auto.nfs
data  -fstype=nfs,rw,hard,intr,timeo=300,nfsvers=3  192.168.3.14:/mnt/storage/data
home  -fstype=nfs,rw,soft,_netdev,nfsvers=4        192.168.3.14:/mnt/storage/home
```

对应的docker启动窗口应该是
```
docker run -d --gpus all \
  --name my-fast-train \
  -v $(pwd):/workspace \
  -v /mnt/data/data_folder:/data \
  my-image:latest python train.py
```

`-v /mnt/data/data_folder:/data`：当 Docker 尝试去读取宿主机的 `/mnt/data/data_folder` 时，会**瞬间触发**宿主机的 Autofs 机制，把远程的 `data1` 数据流源源不断地送进容器内的 `/data` 目录。这样，你在写深度学习代码（`train.py`）时，加载数据集的路径直接写死的相对路径或绝对路径 `/data` 即可.

构建 MLOps 模板项目（核心文件配置）

假设你已经在 GitLab 上建立了一个名为 mlops-template 的项目，并且已经成功拿到了项目级的 Runner Token，且在两台服务器上注册好了亮起绿灯的 Runner（标签为 gpu）。

  

现在，我们用本地的 Cursor 打开这个空项目，需要在这个项目根目录下创建 4 个核心文件。

  

文件 1：.devcontainer/devcontainer.json

作用：让你的本地 Mac Cursor 能够一键通过 SSH 钻进服务器的 Docker 容器里，实现“模式 B”的丝滑 AI 开发。

  

内容：在根目录下新建 .devcontainer/devcontainer.json：

  

JSON

{

"name": "Lab Deep Learning Environment",

"build": {

"dockerfile": "../Dockerfile"

},

"runArgs": [

"--gpus", "all",

"--shm-size", "16g" // 非常重要：防止 PyTorch DataLoader 多线程报错 (Shared Memory OOM)

],

"mounts": [

// 核心数据外挂：把服务器的 NFS 挂载到容器内部的 /data 目录

"source=/mnt/nfs_data,target=/data,type=bind,consistency=cached"

],

"customizations": {

"vscode": {

"extensions": [

"ms-python.python",

"ms-toolsai.jupyter"

]

}

}

}

文件 2：Dockerfile

作用：定义你开发和训练时统一的容器环境（根据前面的讨论，我们直接 FROM 官方稳定源，不需要自己从头造轮子）。

  

内容：在根目录下新建 Dockerfile：

  

Dockerfile

# 1. 选用 PyTorch 官方提供的含 CUDA 环境的稳定镜像

FROM pytorch/pytorch:2.2.1-cuda12.1-cudnn8-devel

  

# 2. 设置时区和中文字符集

ENV TZ=Asia/Shanghai

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

  

# 3. 设置工作目录

WORKDIR /workspace

  

# 4. 配置国内 Pip 源加速下载（这里我在京都适当修改）

RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

  

# 5. 安装一些实验室高频通用的基础包（不要把特定项目的包写在这）

RUN pip install --no-cache-dir \

wandb \

mlflow \

tensorboard \

scikit-learn \

pandas

  

# 如果需要用 Claude Code，可以在这里预装 Node.js 环境（可选）

# RUN apt-get update && apt-get install -y nodejs npm

文件 3：.gitlab-ci.yml (终极调度器)

作用：当你在 Mac 上 git push 后，云端 GitLab 会指挥服务器 Runner 闪电般启动训练容器，然后立刻宣告 CI/CD 成功退出，不挂起进程。

  

内容：在根目录下新建 .gitlab-ci.yml：

  

YAML

stages:

- deploy

  

run_training:

stage: deploy

tags:

- gpu-server # 确保这个标签与你在服务器注册 Runner 时填写的标签一致

rules:

# 规则 1：如果你在 commit message 里写了 [run train]，才会自动触发

- if: '$CI_COMMIT_MESSAGE =~ /\[run train\]/'

when: always

# 规则 2：其他所有 push 默认只生成任务卡片，需要你去 GitLab 网页上手动点 "Play" 按钮才执行

- when: manual

script:

# 1. 提取当前的分支名，用来作为动态容器的名字（支持多人或多分支并发，互不干扰）

- CONTAINER_NAME="train-${CI_PROJECT_NAME}-${CI_COMMIT_REF_SLUG}"

# 2. 如果上一次该分支的训练容器还在跑，先安全停止并强制删除旧容器

- echo "Cleaning up old container..."

- docker stop $CONTAINER_NAME || true

- docker rm $CONTAINER_NAME || true

# 3. 在服务器后台静默启动训练 (-d 参数)

- echo "Starting new training container..."

- docker run -d \

--gpus all \

--name $CONTAINER_NAME \

--shm-size 16g \

-v $(pwd):/workspace \

-v /mnt/nfs_data:/data \

pytorch/pytorch:2.2.1-cuda12.1-cudnn8-devel \

python /workspace/train.py

# 4. 打印一行日志，通知你容器在后台启动成功了

- echo "Training started successfully! You can check logs using 'docker logs -f ${CONTAINER_NAME}'"

文件 4：train.py (极简测试脚本)

作用：用来验证整个流水线、GPU 挂载、以及 W&B 是否打通。

  

内容：在根目录下新建 train.py：

  

Python

import torch

import time

# import wandb # 等你配好 W&B Token 后取消注释

  

print("=== 实验室 MLOps 自动化测试系统 ===")

print(f"1. CUDA 是否可用: {torch.cuda.is_available()}")

if torch.cuda.is_available():

print(f"2. 当前使用的 GPU 设备: {torch.cuda.get_device_name(0)}")

  

print("3. 开始模拟漫长的模型炼丹过程...")

# 这里可以加入你的万能 init-ai.sh 注入的 W&B 启动代码

# wandb.init(project="mlops-test")

  

for epoch in range(1, 6):

print(f"Epoch [{epoch}/5] - 正在训练中... Loss: {0.5 / epoch:.4f}")

time.sleep(3) # 模拟每轮训练耗时

  

print("=== 训练圆满结束！ ===")

  
  

大致重构蓝图：

my-lab-ultimate-scaffold/ # 实验室终极脚手架项目根目录

│

├── .cursor/ # 🧠 AI 规则大脑

│ └── （现有的结构）

│

├── .devcontainer/ # 💻 本地 Mac 开发容器配置

│ └── devcontainer.json # 挂载 GPU、挂载 NFS 数据集、集成 Cursor

│

├── .github/ # 🦊 如果你有时也用 GitHub

│ └── workflows/

│ └── lint.yml # GitHub Actions 自动运行 Ruff 检查

│

├── .gitignore # 过滤掉不需要提交的垃圾文件

├── Dockerfile # 📦 实验室统一的深度学习核心基础镜像

├── .gitlab-ci.yml # 🚀 GitLab 闪电调度器（后台静默训练执行脚本）

├── pyproject.toml # 🎨 Ruff 质检官的规则配置文件

├── init-ai.sh # 💉 万能注入脚本（用于魔改第三方项目时一键拉取上述配置）

├── train.py # 🧪 冒烟测试脚本（用于验证一切是否通畅）

└── README.md # 📖 实验室 Infra 使用说明书

------------------------------------------------------
因为重定向了仓库所以如果出问题可以更新地址
```
understand that the URL will change from `huang.pengbin/ai-coding-rules` to `jil_atr/ai-coding-rules`
# 查看当前的远程仓库地址
git remote -v

# 将其修改为 GitLab 转移后的新地址（在 GitLab 网页上复制新的克隆链接）
git remote set-url origin git@gitlab.com:your-group/mlops-template.git
```