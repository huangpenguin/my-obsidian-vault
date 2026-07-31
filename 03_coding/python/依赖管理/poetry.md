---
title: "poetry"
publish: false
tags: ["Python"]
---
# poetry

<aside>
💡

poetry安装的虚拟环境会在当前文件夹下的哪

</aside>

### **初始化/配置 Poetry 项目**

### **如果是新项目：**

```
mkdir myproject && cd myproject
poetry init  # 交互式生成 pyproject.toml
```

### **如果是已有项目（如你的 backend）：**

确保项目根目录有 **`pyproject.toml`**，然后运行：

```
poetry install  # 根据 pyproject.toml 安装所有依赖
```

---

### **3. 指定 Python 版本**

在 **`pyproject.toml`** 中明确 Python 版本约束：

```
[tool.poetry.dependencies]
python = "^3.9"  # 例如限定 3.9.x
```

Poetry 会自动检测并使用系统中匹配的 Python 版本。若未安装对应版本，会报错提示。

---

### **4. Python 解释器路径**

### **Poetry 使用的 Python 路径：**

- 运行以下命令查看虚拟环境的 Python 路径：复制下载
    
    bash
    
    ```
    poetry run which python
    ```
    
    - **Linux/macOS 输出示例**：
        
        **`/home/username/.cache/pypoetry/virtualenvs/myproject-abcdefg-py3.9/bin/python`**
        
    - **Windows 输出示例**：
        
        **`C:\Users\username\AppData\Local\pypoetry\Cache\virtualenvs\myproject-abcdefg-py3.9\Scripts\python.exe`**
        

---

### **5. 虚拟环境位置**

Poetry 默认将虚拟环境存储在以下路径：

- **Linux/macOS**:
    
    **`~/.cache/pypoetry/virtualenvs/<项目名>-<随机哈希>-py<版本>`**
    
- **Windows**:
    
    **`%APPDATA%\Local\pypoetry\Cache\virtualenvs\<项目名>-<随机哈希>-py<版本>`**
    

### **自定义虚拟环境路径：**

在 **`poetry config`** 中修改：

bash

复制

下载

```
poetry config virtualenvs.in-project true  # 虚拟环境会生成在项目目录下的 `.venv` 文件夹中
```

之后新建的虚拟环境路径为：**`<项目根目录>/.venv`**

---

### **6. 关键操作命令**

| **功能** | **命令** |
| --- | --- |
| 安装依赖 | **`poetry add <包名>`**（如 **`poetry add flask@^2.0`**） |
| 进入虚拟环境 | **`poetry shell`** |
| 退出虚拟环境 | **`exit`** 或 **`Ctrl+D`** |
| 安装全部依赖 | **`poetry install`** |
| 更新依赖 | **`poetry update`** |

---

### **7. 与 Docker 开发的对比**

- **Poetry 本地开发**：
    - 适合快速迭代，直接调试代码。
    - 需手动管理服务依赖（如数据库、前端等）。
- **Docker 开发**：
    - 环境隔离彻底，但可能需要频繁重建镜像。

建议：**本地用 Poetry 开发核心逻辑，用 `docker-compose` 启动辅助服务（如数据库）**。
