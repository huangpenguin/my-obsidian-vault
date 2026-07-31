---
title: "docker知识"
publish: false
tags: ["OCR","项目实践"]
---
# docker知识

```markdown
# Docker 镜像与容器说明 + 日常使用

## 一、镜像和容器的「永久性」详解

### 1. 镜像（Image）

- **是什么**：只读模板，相当于「系统快照」：基础系统 + Paddle + 你装好的依赖，一层层叠在一起。
- **是否永久**：
  - **是**。一旦 `docker build` 或 `docker pull` 成功，镜像会保存在本机（或镜像仓库），**不会因为关机关机或删容器而消失**。
  - 除非你主动执行 `docker rmi <镜像名>` 删除镜像，否则会一直存在。
- **可复用**：同一个镜像可以反复用来创建多个容器。

### 2. 容器（Container）

- **是什么**：由镜像「运行」出来的一个实例，相当于一台正在跑的「小虚拟机」。你在容器里改文件、装包、写数据，都是发生在容器这一层。
- **是否永久**：
  - **默认不永久**。`docker run` 出来的容器，退出后容器本身还在，但：
    - 若用 `docker rm <容器名>` 删掉容器，则**容器内所有改动（包括你后来在容器里装的包、改的文件）都会一起消失**。
    - 只有**挂载到宿主机的目录**（例如 `-v $PWD:/paddle`）里的内容会永久保存在你本机的文件夹里。
  - 所以：**代码、数据、训练结果等应放在挂载目录（如 `/paddle`）里**，这样删了容器也不会丢。

### 3. 对照表

| 对象 | 存什么 | 删掉会怎样 | 典型命令 |
|------|--------|------------|----------|
| **镜像** | 系统 + Paddle + 本项目依赖（只读） | 需要该镜像时得重新 build/pull | `docker build` / `docker pull` / `docker rmi` |
| **容器** | 运行时状态、未挂载的改动 | 容器内新装/新改的东西都没了；挂载目录不受影响 | `docker run` / `docker rm` |
| **挂载目录**（如 `$PWD:/paddle`） | 你仓库里的代码、数据、output | 不会因删容器而丢，和本机目录一致 | 在 `docker run` 里用 `-v` |

### 4. 实际建议

- **镜像**：用本仓库的 Dockerfile 构建一次（或偶尔在依赖变更后重构建），长期保留，反复用。
- **容器**：  
  - 想保留「这次装了什么、改了什么」：用 `docker start/stop` 启停已有容器，不要 `docker rm`。  
  - 不介意每次重来：每次 `docker run` 新容器，代码和数据都在挂载的 `/paddle` 里，删容器只丢容器内的临时改动。
- **重要数据与结果**：一律放在挂载进容器的目录（如 `/paddle`，对应本机仓库目录），这样和本机一致、永久保存。

---

## 二、本项目的 Dockerfile 做了什么

- **基础镜像**：官方 Paddle 镜像（可选 CPU / GPU CUDA11.8 / GPU CUDA12.6），已含 PaddlePaddle。
- **在镜像里额外做的事**：  
  - 只复制 `pyproject.toml` 和 `uv.lock`。  
  - 用 `uv export` 导出依赖列表，再用 `pip install -r` 装到**系统 Python**（不创建 .venv）。  
  - 不包含 Paddle/PaddleOCR 的版本写死，它们来自基础镜像。
- **运行时**：通过 `-v $PWD:/paddle` 把本仓库挂到容器里的 `/paddle`，所以**代码和数据不塞进镜像**，而是用你本机的目录，便于更新代码、保留结果。

---

## 三、日常使用流程

### 第一次（或依赖变更后）：构建镜像

在**本机仓库根目录**执行（按你机器选 CPU 或 GPU）：

```bash
cd /mnt/home/huang/workspace_daihen

# GPU（CUDA 11.8，常用）
docker build -t ocr-train-workspace:latest .

# GPU（CUDA 12.6）
docker build --build-arg PADDLE_IMAGE=ccr-2vdh3abv-pub.cnc.bj.baidubce.com/paddlepaddle/paddle:3.0.0-gpu-cuda12.6-cudnn9.5-trt10.5 -t ocr-train-workspace:cuda12 .

# CPU
docker build --build-arg PADDLE_IMAGE=ccr-2vdh3abv-pub.cnc.bj.baidubce.com/paddlepaddle/paddle:3.0.0 -t ocr-train-workspace:cpu .
```

构建完成后，镜像会一直在本机，除非你 `docker rmi` 删掉。

### 每次要用：启动容器并进到工作目录

```bash
cd /mnt/home/huang/workspace_daihen

# GPU
docker run --gpus all --name paddleocr -v $PWD:/paddle --shm-size=8G --network=host -it ocr-train-workspace:latest /bin/bash

# CPU
docker run --name paddleocr -v $PWD:/paddle --shm-size=8G --network=host -it ocr-train-workspace:cpu /bin/bash
```

- `--name paddleocr`：给容器起名，方便以后 `docker start paddleocr`。  
- `-v $PWD:/paddle`：当前仓库目录挂到容器内 `/paddle`，代码和数据都在这里，**永久在本机**。  
- 进入后当前目录应为 `/paddle`，直接跑脚本即可。

### 在容器里干活

```bash
# 已在 /paddle
python scripts/eval_model.py -c output/xxx/config.yaml -w output/xxx/best_accuracy/best_accuracy.pdparams
python scripts/early_stop_train.py -c configs/rec_base.yaml --patience 10
# 等等
```

结果写在 `output/` 等目录里，因为都在挂载的 `/paddle` 下，退出/删容器后本机也能看到。

### 退出与下次再用

- **退出容器**：`exit` 或 Ctrl+D。
- **下次想用同一容器**（保留容器内临时状态时）：  
  `docker start -ai paddleocr`  
  若提示容器名已存在且你想新建一个，可先 `docker rm paddleocr` 再重新 `docker run ...`。
- **不保留容器**：退出后 `docker rm paddleocr`，下次再 `docker run ...` 新建即可；代码和数据不受影响（在 `/paddle` 挂载里）。

### 依赖或 Dockerfile 改了

改过 `pyproject.toml` / `uv.lock` 或 Dockerfile 后，重新构建镜像即可：

```bash
docker build -t ocr-train-workspace:latest .
```

之后新开的容器都会用新镜像；旧容器仍可用，只是不会自动得到新依赖。

---

## 四、常用命令速查

| 目的 | 命令 |
|------|------|
| 构建镜像 | `docker build -t ocr-train-workspace:latest .` |
| 运行新容器并进入 | `docker run --gpus all -v $PWD:/paddle --shm-size=8G --network=host -it ocr-train-workspace:latest /bin/bash` |
| 列出本机镜像 | `docker images \| grep ocr-train` |
| 列出容器（含已退出的） | `docker ps -a` |
| 启动已有容器并进入 | `docker start -ai paddleocr` |
| 停止容器 | `docker stop paddleocr` |
| 删除容器 | `docker rm paddleocr` |
| 删除镜像 | `docker rmi ocr-train-workspace:latest` |

```

---

## docker的两种模式，开发和生产

### 1. 本地开发模式（也就是你理解的这种）

- **`docker build`（造厨房）：** 仅仅是安装好 Python、配置好各种环境变量、装好飞桨（Paddle）等依赖库。造出一个完美的“运行环境”。
- **`docker run -v`（挂载菜篮子）：** 启动容器时，把你电脑上的代码文件夹“挂载”进去。
- **为什么要这样：** 因为你在本地还要频繁修改代码！你改一行代码，容器里瞬间就能运行最新的代码。如果每次改代码都要重新 build 一次，你会疯掉的。

### 2. 生产上线模式（这是你需要补充的认知）

当你本地代码全部写完、测试通过，准备把这个 AI 模型部署到公司的服务器或者给客户用的时候，玩法就变了。

- **`docker build`（连厨房带菜一起打包）：** 这时候，我们会在 `Dockerfile` 里写一行极其关键的代码：`COPY . /app`。它的意思是，在构建镜像的那一瞬间，把你写好的代码**直接“焊死”到镜像的内部**。
- **`docker run`（直接开机）：** 到了客户的服务器上，只需要敲 `docker run` 直接启动就行了，**完全不需要再用 `v` 挂载代码**。
- **为什么要这样：** 因为到了线上环境，代码是绝对不能随便乱改的。把代码打包进镜像里，就能保证无论是部署在东京的服务器还是北京的服务器，运行的代码版本绝对一模一样，不会因为挂载错了本地文件而出 Bug。
