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
