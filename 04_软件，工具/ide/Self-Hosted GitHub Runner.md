## 搭建 GitLab 云端控制中枢

本阶段分为 **云端配置** 和 **服务器端配置** 两部分。

### 步骤 1：在 GitLab 官网上获取注册凭证（Token）

我们需要去 GitLab 拿到一个“接头暗号”，这样服务器才能合法的连接上你的项目。

1. 打开 [GitLab 官网](https://gitlab.com/)，进入或者新建一个你实验室的**项目（Repository）**或**群组（Group）**。
    
2. 在左侧边栏中，依次点击：**Settings（设置）** -> **CI/CD**。
    
3. 展开 **Runners** 这一项。
    
4. 找到 **Project runners** 区域，点击 **New project runner** 按钮。
    
5. 在创建页面：
    
    - **Platform**（平台）：选择 `Linux`。
        
    - **Tags**（标签）：给这个 Runner 加上标签。例如：服务器 A 加 `server-a`, `gpu`；服务器 B 加 `server-b`, `gpu`。
        
    - 点击 **Create runner**。
        
6. **关键信息**：页面上会弹出一行极重要的命令，格式通常是 `gitlab-runner register --url https://gitlab.com --token GLRT-xxxxxxxxx`。把这个 **Token（令牌）** 复制下来，准备去服务器上用。
    

### 步骤 2：在两台服务器上部署并注册 GitLab Runner

我们之前提到过，Runner 最好直接通过 Docker 容器的方式常驻在服务器后台，这样最干净、最容易维护。

**请分别在服务器 A 和服务器 B 上执行以下命令**：

#### 1. 启动常驻的 Runner 容器

这行命令会在服务器后台启动一个 GitLab Runner 的官方容器。

Bash

```
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

> **笔记原理解析**：`-v /var/run/docker.sock...` 这个挂载非常核心。它的意思是把服务器宿主机的 Docker 进程借给 Runner 容器用，这样 Runner 才能在后续的 CI/CD 中，在服务器上创建并运行其他的深度学习容器。把**宿主机（你的服务器）**的 `/srv/gitlab-runner/config` 目录，映射到**容器内**的 `/etc/gitlab-runner` 目录。

#### 2. 执行注册（将服务器与云端绑定）

运行以下命令进入刚刚启动的容器内部，触发注册流程：

Bash

```
docker exec -it gitlab-runner gitlab-runner register
```

接下来终端会进入**交互式提问**，请根据以下提示填写：

1. **Enter the GitLab instance URL**: 直接回车（默认就是 `https://gitlab.com/`）。
    
2. **Enter the registration token**: 粘贴你刚刚在步骤 1 中复制的那个 `GLRT-xxxxxxxxx` 令牌。
    
3. **Enter a name for the runner**: 给它起个名字，比如 `lab-server-a`。
    
4. **Enter an executor**: **非常关键，输入 `docker`**。
    
5. **Enter the default Docker image**: 输入 `ubuntu:22.04`（这是当你的 CI/CD 脚本没指定镜像时，默认使用的垫底镜像）。

gitlab-runner register  --url https://gitlab.com  --token glrt-_yK5tj65DbPSEPaSBwTJ3GM6MQpvOjEKcDoxY200b2IKdDozCnU6Z3l3bnoc.01.1o0ae63mo
    

**成功标志**：完成提问后，刷新你的 GitLab 网页端 Runner 页面，你会看到刚刚配置的服务器名字亮起了**绿灯**，这意味着云端中枢已经成功跟你的 GPU 服务器握手了！

### 步骤 3：配置 Runner 允许使用显卡

由于我们的 Runner 是通过 Docker Executor 启动其他容器来跑任务的，默认情况下它启动的子容器是看不到 GPU 的。我们需要修改一下 Runner 的配置文件。

**在服务器终端执行以下命令修改配置**：

1. 用编辑器打开 Runner 的配置文件：
    
    Bash
    
    ```
     sudo nano /srv/gitlab-runner/config/config.toml
    ```
    
2. 找到 `[runners.docker]` 这一行，在它的下方，把 `gpus = ""` 修改为：
    
    Ini, TOML
    
    ```
    gpus = "all"
    ```
    
    _(如果找不到这一行，直接在 `[runners.docker]` 这一段落里手动添加一行 `gpus = "all"` 即可)_
    
3. 保存并退出（Nano 编辑器按 `Ctrl+O` 回车保存，`Ctrl+X` 退出）。
    
4. 重启服务器上的 Runner 容器让配置生效：
    
    Bash
    
    ```
    docker restart gitlab-runner
    ```
    

至此，**云端控制中枢与两台服务器的管道已经彻底铺设完毕**。云端已经可以对两台服务器下发任何 Docker 相关的指令，并且这些指令启动的容器都能全量调用 GPU。

现在，我们可以进入最兴奋的阶段：**编写项目的 CI/CD 脚本（`.gitlab-ci.yml`），来实现你设想的“闪电下发、后台静默训练、W&B/MLflow 监控”机制了**。



---
![image.png](https://raw.githubusercontent.com/huangpenguin/note-images/main/img/%7By%7D/%7Bm%7D/20260524223630056.png)
## 1. GitLab CI 是什么？

你可以把它理解成：

> 你 push 代码 → GitLab 自动按 `.gitlab-ci.yml` 里的“任务清单”去执行一系列脚本。

一次 push 会生成一条 Pipeline（流水线），里面有一个或多个 Job（任务）。

你的仓库里，一次 pipeline 大致是这样：

---

## 2. 几个核心概念

### Pipeline

一次 CI 运行的总流程。在 GitLab 页面 CI/CD → Pipelines 里能看到。

### Stage（阶段）

Job 的分组，按顺序执行：

- 先跑完 `quality` 阶段的所有 job
- 再跑 `deploy` 阶段

### Job（任务）

真正干活的单元。每个 job 就是一段 shell 脚本。

例如：

- `quality:ruff-check`：跑 Ruff 检查
- `run_training`：在 GPU 服务器上启动 Docker 训练容器

### Runner（执行器）

真正执行 job 的机器/程序。GitLab 本身不跑你的代码，它把 job 分配给 Runner。

Runner 分两类：

|类型|谁提供|典型用途|
|---|---|---|
|GitLab Shared Runner|GitLab 云|跑 lint、测试（有 `image:` 的 job）|
|Self-hosted Runner|你自己的 GPU 服务器|跑训练、访问 `/mnt/nfs_data`、用本地 Docker|

### `image:` 是什么？

告诉 Runner：在一个 Docker 容器里跑这个 job。

你的 quality job 用了：

quality.ymlLines 8-8

image: ghcr.io/astral-sh/uv:python3.12-bookworm-slim

意思是：在带 `uv` 的 Python 容器里执行 `ruff`、`pyright`。这种 job 通常交给 GitLab 共享 Runner 就行，不需要你的 GPU 服务器。

### `script:` 是什么？

Job 里真正要执行的命令，就是普通 shell 脚本。

### `rules:` 是什么？

这个 job 什么时候出现、什么时候自动跑、什么时候只能手动点。

你的 `run_training` 规则是：

train.ymlLines 8-13

rules:

- if: '$CI_COMMIT_BRANCH == "main" && $CI_COMMIT_MESSAGE =~ /\[run train\]/'

when: on_success

- if: $CI_COMMIT_BRANCH == "main"

when: manual

allow_failure: true

含义：

1. 在 `main` 分支，且 commit message 含 `[run train]` → 自动跑
2. 在 `main` 分支，其他情况 → 手动 Play，且失败不阻塞 pipeline（`allow_failure: true`）

### `include:` 是什么？

把 CI 配置拆成多个文件，根 `.gitlab-ci.yml` 负责“总调度”：

.gitlab-ci.ymlLines 22-24

include:

- local: .gitlab/ci/quality.yml

- local: .gitlab/ci/train.yml

### `extends:` 是什么？

复用配置，避免重复写。  
`.quality_base` 是“模板 job”（名字以 `.` 开头，不会单独执行），其他 job `extends` 它。

---

## 3. 你的两类 Job 为什么不一样？

### Quality jobs（不需要 GPU Runner tag）

quality.ymlLines 27-40

quality:ruff-check:

extends: .quality_base

script:

- uv run ruff check .

...

特点：

- 有 `image:` → 在容器里跑
- 没有 `tags:` → 任意可用的共享 Runner 都能接
- 只做代码检查，不需要 GPU、不需要你的 NFS

### `run_training`（必须要 tag）

train.ymlLines 1-4

run_training:

stage: deploy

tags:

- gpu

特点：

- 没有 `image:`（不是让 GitLab 开容器跑 CI，而是在 Runner 所在机器上直接执行 `docker build` / `docker run`）
- 必须跑在 你自己的 GPU 服务器 上
- 需要访问本地 Docker、`/mnt/nfs_data` 等

所以这类 job 必须指定 tag，告诉 GitLab：“请把这个任务交给贴了这个标签的 Runner”。

---

## 4. Runner tag 还要不要？

要，而且非常重要。

`tags` 不是可有可无的装饰，而是 job 和 Runner 的匹配条件：

# Job 说：我只接受带 gpu-server 标签的 Runner

tags:

- gpu-server

注册 Runner 时你也要填同样的 tag：

gitlab-runner register

# ...

# Tags: gpu-server

匹配关系：

|Job 里的 tags|Runner 注册时的 tags|结果|
|---|---|---|
|`gpu-server`|`gpu-server`|✅ 能跑|
|`gpu`|`gpu-server`|❌ 永远 pending|
|不写 tags|共享 Runner|❌ GPU 服务器接不到（或跑错机器）|

你写的：

tags:

- gpu-server

完全正确，只要你在 GPU 服务器注册 Runner 时用的也是 `gpu-server`。

当前仓库里写的是 `gpu`，你实际注册的是 `gpu-server` 的话，就要改成一致，否则 job 会一直 pending（等待 Runner）。

---

## 5. 为什么 quality 不用 tag，training 要用？

可以记这个简单规则：

|需求|是否需要 tag|
|---|---|
|普通 lint / 测试 / 构建|通常不需要，用共享 Runner + `image:`|
|必须用某台物理机（GPU、NFS、本地 Docker）|必须 用 tag 指定|

你的 `run_training` 脚本里要执行：

docker build ...

docker run --gpus all ...

-v /mnt/nfs_data:/data

这些只能在 你的 GPU 服务器 上做，所以必须 tag。

---

## 6. 你配置里几个容易混淆的点

### `needs: []`

表示 不等待 quality job 完成，deploy 阶段可以独立调度。  
训练派发更快，但意味着 `[run train]` 自动触发时也不会等 lint 通过。

### `resource_group: gpu-training`

同一项目里，训练 job 互斥，避免两个训练同时抢 GPU。

### `when: manual`

Job 会出现在 pipeline 里，但要你手动点 Play 才执行。

### `allow_failure: true`

这个 job 失败了，pipeline 整体仍可能显示 success（适合“可选的手动训练”）。

---

## 7. 一次完整流程（结合你的项目）

1. 你 `git push` 到 GitLab `main`
2. GitLab 读取 `.gitlab-ci.yml` + include 的文件
3. 创建 pipeline，包含：
    - `quality:ruff-check`
    - `quality:ruff-format`
    - `quality:pyright`
    - `run_training`（manual 或 `[run train]` 触发）
4. quality 三个 job → GitLab 共享 Runner 在 Docker 容器里跑
5. `run_training` → 只有 tag 为 `gpu`（或你改成 `gpu-server`）的 Runner 才会接
6. Runner 在 GPU 服务器上执行 `docker build` + `docker run -d ...`
7. CI job 结束；训练在后台容器里继续跑，你要用 `docker logs -f ...` 看结果

---

## 8. 你应该怎么设 tag？

Job 和 Runner 必须完全一致。 二选一即可：

方案 A：继续用 `gpu`

- Runner 注册 tag：`gpu`
- CI 里：`tags: [gpu]`（当前仓库就是这样）

方案 B：改成 `gpu-server`（你提到的）

- Runner 注册 tag：`gpu-server`
- CI 里改成：
    
    tags:
    
    - gpu-server
    

多台 GPU 服务器时，可以用更细 tag，例如：

- `gpu-a100-1`
- `gpu-4090-2`

然后在 job 里指定你想跑在哪台：

tags:

- gpu-a100-1

---

## 9. 一句话总结

- CI 语法：`.gitlab-ci.yml` 定义 pipeline → stage → job → script
- Quality job：云 Runner + Docker 镜像，一般 不需要 tag
- Training job：必须跑在你的 GPU 服务器上，必须要 tag
- `gpu-server` vs `gpu`：名字无所谓，Job 和 Runner 注册时必须相同