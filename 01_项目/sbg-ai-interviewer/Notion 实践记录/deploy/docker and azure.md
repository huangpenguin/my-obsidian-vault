---
title: "docker and azure"
publish: false
tags: ["项目实践"]
---
# docker and azure

### 镜像（Image） vs. 容器（Container）：到底在 Build 啥？

这是最经典的问题。请看这个比喻：

- **镜像 (Image)**：相当于**“游戏安装包”**（或者是快餐店的**标准食谱**）。它是只读的、静态的文件。你执行 `docker build` 得到的就是**镜像**。它还没跑起来，不占内存，只是躺在硬盘里。
- **容器 (Container)**：相当于**“运行中的游戏”**（或者是按照食谱**刚炒出来的那盘菜**）。它是镜像的运行实例。当你执行 `docker run` 时，系统会把镜像解压并跑起来，这时它才是一个真正的进程。

> **结论**：`build` 的是**镜像**。你把镜像传到 Azure，Azure 会根据这个镜像帮你启动一个或多个**容器**。
> 

### 第一站：你的电脑（工厂）

- **原材料：** `Dockerfile` + 源代码（Python 文件）。
- **动作：** `docker build`。
- **产物：** **镜像 (Docker Image)**。
    - *解释：* 这一步是在你本地完成的。就像你在工厂里把零件组装好，打包成了一个**“密封的快递箱”**。这个箱子里装好了 Python、uv、你的代码，一切都准备就绪。

### 第二站：Azure Container Registry (ACR)（物流仓库）

- **动作：** `docker push`。
- **过程：** 你把那个“密封的快递箱”（镜像）上传到 Azure 的仓库里存着。
- **重点：** Azure 的 **Container Apps (运行平台)** 此时还不知道你要干嘛，东西只是存到了仓库里。
    - *回答你的问题：* 所以我们“丢”上去的，是打包好的**镜像**。

### 第三站：Azure Container Apps (ACA)（零售店/运行平台）

- **动作：** 部署 (Deploy)。
- **过程：**
    1. 你告诉 Azure Container Apps：“喂，去仓库（ACR）里把版本号为 `v1` 的那个快递箱拿出来。”
    2. Azure 从仓库**拉取 (Pull)** 那个镜像。
    3. Azure **拆箱运行 (Run)**。
- **产物：** **容器 (Container)**。
    - *解释：* 只有当镜像跑起来了，它才叫“容器”。

---

### 总结流程图

代码段

`graph LR
    A[你的电脑] -- docker build --> B(Docker 镜像 Image)
    B -- docker push --> C[Azure 仓库 ACR]
    C -- 只有这一步是在云端拉取 --> D[Azure 运行平台 ACA]
    D -- 跑起来变成 --> E(Docker 容器 Container)`

### 你的实际操作步骤（复习）

1. **本地：** `docker build -t myapp:v1 .`
    - (把代码和 Dockerfile 变成**镜像**)
2. **上传：** `docker push myregistry.azurecr.io/myapp:v1`
    - (把**镜像**丢进 Azure 仓库)
3. **告知：** 在 Azure Portal 的 Container Apps 里设置 -> Image: `myapp:v1`
    - (告诉 Azure 去仓库拿这个镜像跑)
