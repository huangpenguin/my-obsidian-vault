---
title: "docker指令和维护"
publish: false
tags: ["OCR","项目实践"]
---
# docker指令和维护

**1. 首次构建镜像（仓库根目录）**

```markdown
cd /mnt/home/huang/workspace_daihen

# GPU（CUDA 11.8）
docker build -t ocr-train-workspace:latest .

# 或 CPU
docker build --build-arg PADDLE_IMAGE=ccr-2vdh3abv-pub.cnc.bj.baidubce.com/paddlepaddle/paddle:3.0.0 -t ocr-train-workspace:cpu .
```

**2. 运行容器并进入（每次要用时）**

```markdown
cd /mnt/home/huang/workspace_daihen

# GPU
docker run --gpus all --name paddleocr -v $PWD:/paddle --shm-size=8G --network=host -it ocr-train-workspace:latest /bin/bash

# CPU
docker run --name paddleocr -v $PWD:/paddle --shm-size=8G --network=host -it ocr-train-workspace:cpu /bin/bash
```

**3. 退出与下次进入同一容器**

```
# 退出
exit

# 下次复用同一容器
docker start -ai paddleocr
```

**4. 不再需要该容器时**

```markdown
docker stop paddleocr
docker rm paddleocr
```

```markdown
注意如果在container挂载的文件夹下由container默认的root用户创建的文件
如果回到本地还要再使用的话记得赋予本地用户权限
sudo chown -R huang:huang /mnt/home/huang/workspace_daihen/StyleText
```

---

四、后续维护
1. 增加新依赖（要进镜像、进 uv.lock）
在本机仓库根目录（当前物理目录）执行：
cd /mnt/home/huang/workspace_daihen
uv add 包名
docker build -t ocr-train-workspace:latest .
之后新开的容器（或 docker run 新容器）都会带新包；已有容器不会自动更新，需要新建容器或重建镜像后再用。
2. 容器里 /paddle 就是挂载的本机目录，下次进容器（或新开容器）直接看到最新代码，无需 rebuild 镜像。
3. 更新 Paddle 基础镜像版本
修改 Dockerfile 第一行 ARG PADDLE_IMAGE=... 或构建时传参
