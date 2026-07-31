---
title: "PyPI api for uv"
publish: false
tags: ["Python"]
---
# Pypi api for uv

### **初始化项目**

### 创建项目目录：

```
mkdir myproject && cd myproject
```

### 初始化虚拟环境：

```
uv venv  # 默认在 .venv 目录创建虚拟环境
```

- 指定虚拟环境名称：`uv venv myenv`
- 使用系统 Python：`uv venv --system`

---

### **激活虚拟环境**

```
source .venv/bin/activate
```

### Windows：

```
.venv\Scripts\activate
```

---

### **4. 安装依赖**

### 安装单个包：

```
uv pip install requests
```

### 从 requirements.txt 安装：

```
uv pip install -r requirements.txt
```

### 生成并安装锁定依赖（类似 `pip-compile`）：

```
uv pip compile requirements.in -o requirements.txt  # 生成依赖
uv pip sync requirements.txt  # 安装依赖
```

---

### **5. 管理依赖**

### 添加开发依赖：

```
uv pip install pytest --group=dev
```

### 导出依赖：

```
uv pip freeze > requirements.txt
```

### 卸载包：

```
uv pip uninstall requests
```

---

### **6. 运行项目脚本**

```
uv run python main.py
```

---

### **7. 升级依赖**

### 升级所有包：

```
uv pip install --upgrade -r requirements.txt
```

### 升级单个包：

```
uv pip install --upgrade requests
```

---

### **8. 删除虚拟环境**

直接删除虚拟环境目录：

```
rm -rf .venv  # 或手动删除 myenv 目录
```

---

### **9. 常用命令示例**

### 示例 1：从零开始创建项目

```
uv venv
source .venv/bin/activate
uv pip install fastapi "uvicorn[standard]"
uv pip freeze > requirements.txt
```

### 示例 2：同步生产环境依赖

```
uv pip sync requirements.txt --strict
```

### 示例 3：安装开发依赖

```
uv pip install black flake8 --group=dev
```

---

---

### **11. 升级 uv 自身**

```
uv pip install --upgrade uv
```
