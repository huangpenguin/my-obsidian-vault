---
title: "docker权限管理(Entrypoint + gosu 大法)"
publish: false
tags: ["OCR","项目实践"]
---
# docker权限管理(Entrypoint + gosu 大法)

如果你去看 Redis、PostgreSQL 这些世界级开源软件的 Docker 源码，它们全是用这招。这招最硬核，哪怕外面的人乱挂载，它也能在内部自己摆平。

1. **容器启动时**：必须还是以 **Root** 身份进去（皇帝亲自出马）。
2. **执行脚本（Entrypoint）**：皇帝进去后的第一件事，是看一眼挂载进来的代码目录到底是谁的，然后用 `chown` 命令把目录权限强行改成内部普通用户的。
3. **退位让贤（gosu）**：权限改顺了之后，皇帝立刻用 `gosu`（一个轻量级权限切换工具）脱下龙袍，切换成普通用户去运行真正的业务代码（比如 `node app.js`）。
- **优势**：绝对不会翻车，极其优雅，但需要你会写简单的 Shell 脚本作为入口点。

---

### 1. 如果你用的是 Debian/Ubuntu 系列（比如 `node:18`、`python:3.10-slim`）

你需要安装的神器叫 **`gosu`**。

- **怎么装**：在你的 `Dockerfile` 里加一行：Dockerfile
    
    `RUN apt-get update && apt-get install -y gosu && rm -rf /var/lib/apt/lists/*`
    

### 2. 如果你用的是 Alpine 系列（比如 `node:18-alpine`、`nginx:alpine`）

因为 Alpine 极度精简，它的包管理器里没有 `gosu`，但是有一个它的完美双胞胎兄弟，叫 **`su-exec`**。

- **怎么装**：在你的 `Dockerfile` 里加一行：Dockerfile
    
    `RUN apk add --no-cache su-exec`
    

---

### 🧐 进阶拷问：为什么非要装这个？我用系统自带的 `su` 或者 `sudo` 不行吗？

这是面试高级容器工程师的一道**经典必考题**！三叔给你把底牌揭开：

如果你在 entrypoint 脚本里用 `su - myuser -c "node app.js"` 或者 `sudo` 来切换用户并启动应用，你的应用进程（Node）将**无法拿到 PID 1（一号进程）的身份**，PID 1 会被 `su` 命令霸占。

- **后果就是**：当你想关掉容器（执行 `docker stop`）时，Docker 发出的终止信号（SIGTERM）只会发给 PID 1（也就是 `su`）。你的 Node.js 应用根本收不到关机信号，它会被强行“拔电源”杀死（SIGKILL），导致数据丢失或文件损坏！
- **gosu / su-exec 的神仙操作**：它们在完成“降级脱下龙袍（Root 换成普通用户）”之后，会**自动原地爆炸**，把 PID 1 的皇位完美让给你的业务进程（`node app.js`）。这样你 `docker stop` 时，应用就能优雅地退出。

---

### 🛠️ “终极看门狗”的完整代码骨架（供你直接抄）

一旦装好了 `gosu`，你需要配合一个 `entrypoint.sh` 脚本来完成这套优雅的连招。逻辑非常固定：

**准备一个 `entrypoint.sh` 文件：**

Bash

`#!/bin/bash
# 此时还是 Root 身份（皇帝）

# 1. 把外面挂载进来的代码目录（/workspace），强行改成内部普通用户（myuser）的
chown -R myuser:myuser /workspace

# 2. 脱下龙袍（Root），换上普通衣服（myuser），并把皇位（PID 1）让给真正的命令
exec gosu myuser "$@"`

**在你的 `Dockerfile` 里这样写：**

Dockerfile

`FROM ubuntu:22.04

# 1. 装 gosu
RUN apt-get update && apt-get install -y gosu

# 2. 建一个内部普通用户 myuser
RUN useradd -m myuser

# 3. 把看门狗脚本扔进去，给执行权限
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

WORKDIR /workspace

# 4. 设置入口点
ENTRYPOINT ["/entrypoint.sh"]

# 5. 默认执行的命令（会作为 "$@" 传给 entrypoint.sh）
CMD ["node", "app.js"]`

**三叔总结：**
用这招，你开发团队里的其他兄弟不管在外面怎么瞎搞、用什么乱七八糟的 UID 挂载文件进容器，这个看门狗脚本都会在启动的瞬间把权限全部“洗”成容器认识的样子，绝对不会再报没有权限的错！
