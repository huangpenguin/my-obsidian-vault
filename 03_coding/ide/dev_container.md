### 验证 Cursor 与远程容器的无缝连接

既然你想确认 Cursor 能不能很好地使用服务器上的 Docker，我们现在就花 3 分钟做个实验，让你亲眼看看 Dev Container 的魔力。请按照以下步骤实操并记录：

**第一步：用 Cursor 连上服务器**

1. 在 Mac 上打开 Cursor。
    
2. 安装扩展：在左侧扩展栏搜索并安装 **Remote - SSH** 和 **Dev Containers** 这两个官方插件。
    
3. 按 `Cmd + Shift + P` 呼出命令面板，输入 `Remote-SSH: Connect to Host`，输入你的服务器 SSH 地址（比如 `ubuntu@你的服务器IP`），连上你的服务器。
    

**第二步：在服务器上建一个测试目录**

1. 连接成功后，在 Cursor 底部的终端里敲击命令新建一个测试文件夹：
    
    Bash
    
    ```
    mkdir ~/test-dev-container && cd ~/test-dev-container
    ```
    
2. 在 Cursor 左侧资源管理器中，点击“打开文件夹”，选择刚刚创建的 `~/test-dev-container`。
    

**第三步：一键生成 Dev Container 配置**

1. 再次按 `Cmd + Shift + P`，输入 `Dev Containers: Add Dev Container Configuration Files` 并回车。
    
2. 依次选择：`Modify your active workspace` -> `Python 3` -> `默认版本 (比如 3.12)` -> 都不选其他附加功能，直接点 OK。
    
3. 这时你的目录下会多出一个 `.devcontainer/devcontainer.json` 文件。
    

**第四步：加入 GPU 支持并启动**

1. 打开那个 `devcontainer.json`，在花括号 `{}` 里面加上这一行配置，让容器能挂载 GPU：
    
    JSON
    
    ```
    "runArgs": ["--gpus", "all"],
    ```
    
2. 保存文件。Cursor 右下角会自动弹出一个提示框：**"Reopen in Container"**（在容器中重新打开）。点击它！ _(如果没有弹框，按 `Cmd + Shift + P` 输入 `Dev Containers: Rebuild and Reopen in Container`)_
    

**第五步：见证奇迹** 等待一两分钟（Cursor 正在服务器上拉取 Python 镜像并把环境塞进去）。当左下角绿色的状态栏显示 Dev Container 连上后：

1. 打开 Cursor 底部的新终端。
    
2. 输入 `nvidia-smi` 并回车。你能看到显卡信息。
    
3. 输入 `python --version` 并回车。你能看到干净的 Python 环境。
    
4. **最重要的是**：你虽然是在 Mac 的界面上操作，但你现在敲的每一个字符、运行的每一行代码，都被完全包裹在这个服务器的独立 Docker 容器里了，绝对不会污染服务器本身的系统！