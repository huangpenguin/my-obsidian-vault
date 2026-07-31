---
title: "怎么把 Docker 镜像推送到 Azure？（手把手命令）"
publish: false
tags: ["项目实践"]
---
# 怎么把 Docker 镜像推送到 Azure？（手把手命令）

既然 Agent 不能帮你敲终端命令，你需要自己打开终端（Terminal / CMD）执行这些步骤。
假设你的项目文件夹叫 `chatbot-project`，且你已经安装了 Docker Desktop 和 Azure CLI (`az`)。

**第一阶段：登录与准备**

1. **登录 Azure：**Bash
    
    `az login
    # 浏览器会弹窗让你登录，登录成功后终端会显示你的订阅信息`
    
2. **找到你的仓库地址 (ACR)：**
你需要问管理员或者在 Azure Portal 搜 "Container registries"，找到那个名字，比如叫 `mycompanyacr`。它的完整地址通常是 `mycompanyacr.azurecr.io`。
3. **登录仓库：**Bash
    
    `az acr login --name mycompanyacr
    # 成功会显示 "Login Succeeded"`
    

**第二阶段：打包与推送**
4.  **构建镜像 (Build)：**
在你的项目根目录下（有 Dockerfile 的地方）运行：
`bash # 注意：要把 mycompanyacr 换成你真实的仓库名 # v1 是版本号，下次更新可以是 v2 docker build -t mycompanyacr.azurecr.io/interviewer-bot:v1 .` 
5.  **推送到云端 (Push)：**`bash docker push mycompanyacr.azurecr.io/interviewer-bot:v1`

**第三阶段：部署 (Deploy)**
这一步通常在 **Azure Portal** 网页上操作最简单（因为你要配置 Secrets）：

1. 进入 **Container Apps** 页面。
2. 找到你的 App，点击左侧菜单的 **"Containers"**。
3. 点击 **"Edit and deploy"**。
4. 把镜像地址选为你刚才 push 上去的 `mycompanyacr.azurecr.io/interviewer-bot:v1`。
5. 保存，Azure 就会开始更新了。
