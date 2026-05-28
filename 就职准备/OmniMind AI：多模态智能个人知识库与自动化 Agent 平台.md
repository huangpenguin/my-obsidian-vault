
---
🏗️ 项目核心架构设计

这个项目将融合你提到的所有技术，且完全可以在**个人本地环境（零成本）**运行调试。

```
[前端: Next.js + TS] <---> [后端: FastAPI + Python] <---> [AI 层: Ollama / Chroma / YOLO]
```

---

🛠️ 任务分解与干中学路线图

我们把项目拆分为四个阶段，每个阶段对应简历上的一个“高亮技能点”。

第一阶段：搭建全栈骨架（走通前后端与TS类型安全）

- **你要做的**：
    1. 用 Next.js (App Router) + TypeScript 搭建前端，利用 Tailwind CSS 快速写个现代化的聊天和文件上传界面。
    2. 用 FastAPI 搭建后端，设计一套支持 **SSE (Server-Sent Events) 流式响应**的聊天接口。
- **干中学（简历吸睛点）**：
    - **tRPC / OpenAPI 自动生成**：前端 TypeScript 如何与后端 Python 实现**类型同步**？学会用 FastAPI 自动生成的 `openapi.json` 自动转换出前端的 TS 类型定义（如使用 `openapi-typescript`），避免前后端字段对不上的低级错误。
    - **流式传输（Streaming）**：掌握打字机效果的底层原理（不写 `await response.json()`，而是通过流式读取 `ReadableStream` 渲染前端）。

第二阶段：构建高性能 RAG（本地知识库系统）

- **你要做的**：
    1. 前端支持用户上传 PDF、Markdown 或图片。
    2. FastAPI 接收文件，调用本地 Python 库（如 `pypdf`）切分文本。
    3. 使用轻量级向量库 **Chroma**（本地嵌入式模式），调用本地 **Ollama** 的 Embedding 模型（如 `bge-m3`）将文本向量化并存入 Chroma。
- **干中学（简历吸睛点）**：
    - **异步文件处理**：FastAPI 中使用 `BackgroundTasks` 异步处理文件解析，避免大文件上传导致接口卡死。
    - **混合检索优化**：学习并实现基础的 RAG 优化，如 **Chunk 块大小调优**、结合 **BM25 传统搜索 + 向量搜索** 的重排（Rerank）机制。

第三阶段：多模态扩展（集成计算机视觉 CV）

- **你要做的**：
    1. 用户上传一张带有多目标的照片（如各种水果、车辆、商品）。
    2. FastAPI 后端调用本地 **Ultralytics YOLO** 模型或 **OpenCV** 进行实时目标检测/图像裁剪。
    3. 后端将检测出的物体标签和裁剪后的图像特征，作为 Prompt 的上下文传给大模型，让大模型生成一份“图像分析报告”返回给 Next.js 展现。
- **干中学（简历吸睛点）**：
    - **混合技术栈协同**：真正理解什么时候用传统 CV（OpenCV 裁剪/调色），什么时候用深度学习（YOLO 检测），以及如何把 CV 的输出转化为 LLM 能读懂的结构化文本。

第四阶段：可拓展的 Agent 工具调用（ReAct 架构）

- **你要做的**：
    1. 在 FastAPI 中手写一个简单的 **ReAct (Reason-Action) 执行器**（不依赖重型框架，纯手写保值率最高）。
    2. 为 Agent 注册两个本地工具：`fetch_weather`（查天气API）和 `query_database`（查本地知识库）。
    3. 让大模型根据用户输入，自主决定是否调用工具、调用什么工具，并将结果拼接后返回给前端。
- **干中学（简历吸睛点）**：
    - **大模型 Function Calling (工具绑定)**：掌握如何用 JSON Schema 描述 Python 函数，并将其传给大模型，解析大模型返回的 `arguments` 并用 Python 动态执行。

---

📈 如何在面试中“包装”这个项目？

转职面试时，面试官最看重的是**解决工程问题的能力**，而不是“调包”。你可以这样描述你的亮点：

1. **全栈类型安全（Type Safety）**：
    - _面试台词_：“我利用 FastAPI 的 Pydantic 模型作为 Single Source of Truth，通过自动化工具将其映射为 Next.js 端 TypeScript 的强类型接口，这减少了 90% 以上因前后端字段变更导致的运行时错误。”
2. **异步与流式体验优化（Performance）**：
    - _面试台词_：“针对大模型的长文本生成，我在 FastAPI 端设计了异步生成器（Async Generator），通过 SSE（Server-Sent Events）将 Token 实时推送到 Next.js 前端，将用户感知到的首字延迟（TTFT）从 5秒 降低到了 200ms 以内。”
3. **异构数据处理能力（AI Engineering）**：
    - _面试台词_：“项目集成了传统的文本 RAG 与 CV 目标检测。利用 YOLO 对用户上传的图片进行预处理和特征提取，将其转化为语义标签输入给 LLM，实现了跨模态的智能问答。”

---

🏗️ OmniMind AI 项目开发流程与架构部署指南

1. 整体系统架构图 (Architecture)

所有代码和前端服务在本地/Docker中隔离，通过高性能网络互联。由于 V100 属于较早期的架构（Volta），我们采用 **Ollama/vLLM 容器化** 来自动处理 CUDA 环境，避免版本冲突。

```
[ 本地 Mac / Cursor ] 
       │ (SSH 远程开发 / Git Push)
       ▼
[ 代码托管集群: GitHub / GitLab ]
       │ (Webhook 触发)
       ▼
[ 远程 V100 服务器 (Ubuntu) ] ── (运行 GitLab Runner)
       │
       ├──► [ Docker 运行环境 (算力层) ]
       │     ├── Container 1: vLLM / Ollama (分配 GPU 0, 1 -> 跑大模型/Embedding)
       │     └── Container 2: Ultralytics YOLO (分配 GPU 2 -> 跑视觉检测)
       │
       └──► [ Docker 运行环境 (业务层) ]
             ├── Container 3: FastAPI 后端 (Port 8000 + ChromaDB 本地嵌入)
             └── Container 4: Next.js 前端 (Port 3000 -> 接收客户端请求)
```

---

2. 环境初始化准备 (Server Setup)

在服务器（Ubuntu）上，首先需要配置好 GPU 容器运行时，这是让 Docker 能够调用 4 张 V100 显卡的基石。

2.1 安装 NVIDIA Container Toolkit

在服务器上执行，让 Docker 获得 GPU 加速能力：

bash

```
# 添加官方源
curl -fsSL https://github.io | sudo gpgcheck=false gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://github.io | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/types.d/nvidia-container-toolkit.list

# 安装并重启 Docker
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

请谨慎使用此类代码。

2.2 启动算力层基础容器 (Ollama)

使用前两张 V100 显卡（GPU 0, 1）来运行大模型：

bash

```
docker run -d --gpus '"device=0,1"' \
  -v ollama:/root/.ollama \
  -p 11434:11434 \
  --name ai-ollama \
  --restart always \
  ollama/ollama

# 进入容器下载你需要的模型（Qwen2.5 14B 非常适合 V100 双卡并跑）
docker exec -it ai-ollama ollama run qwen2.5:14b
docker exec -it ai-ollama ollama pull bge-m3 # 下载 RAG 用的 Embedding 模型
```

请谨慎使用此类代码。

---

3. Cursor 驱动的“干中学”开发流程

通过 Cursor 的 AI 辅助功能，你可以快速写出没有历史包袱的干净代码。

3.1 仓库初始化与本地配置

1. 在 GitHub/GitLab 创建一个 Mono-Repo（单仓多项目）命名为 `omnimind-ai`。
2. 目录结构推荐：
    
    text
    
    ```
    omnimind-ai/
    ├── frontend/        # Next.js + TypeScript 项目
    ├── backend/         # FastAPI + Python 项目 (内含 ChromaDB 存储)
    ├── .gitlab-ci.yml   # GitLab CI/CD 配置文件
    └── docker-compose.yml
    ```
    
    请谨慎使用此类代码。
    

3.2 使用 Cursor 远程编码工作流

1. 打开 Mac 的 Cursor，点击左下角的 **绿色图标 (Remote-SSH)**。
2. 输入服务器的 SSH 连接信息，直接打开服务器上的 `omnimind-ai` 文件夹。
3. **Cursor Composer (⌘ + I 或 Ctrl + I)** 核心玩法语录：
    - _“在 backend 目录下帮我用 FastAPI 初始化一个支持 CORS 的多模块项目，要求使用 Pydantic v2，并预留一个 /api/chat 的 SSE 流式路由。”_
    - _“在 frontend 目录下用 Next.js 14 App Router 和 TypeScript 写一个 Chat 界面，支持 Markdown 渲染和大模型流式打字机效果。”_

---

4. 生产级 Docker 编排配置

在项目根目录下编写 `docker-compose.yml`，将全栈业务服务联动在一起。

yaml

```
version: '3.8'

services:
  backend:
    build: ./backend
    container_name: omnimind-backend
    restart: always
    ports:
      - "8000:8000"
    volumes:
      - ./backend/chroma_data:/app/chroma_data # 持久化向量数据库
    environment:
      - OLLAMA_BASE_URL=http://docker.internal # 关联宿主机的 Ollama
    extra_hosts:
      - "host.docker.internal:host-gateway"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ['2'] # 分配第三张 V100 专门处理视觉或后端推理任务

  frontend:
    build: ./frontend
    container_name: omnimind-frontend
    restart: always
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://YOUR_SERVER_IP:8000
    depends_on:
      - backend
```

请谨慎使用此类代码。

---

5. 企业级 GitLab CI/CD 自动化部署

既然你已经配置好了 GitLab Runner，我们直接编写 `.gitlab-ci.yml`。这里的策略是：**代码一推，Runner 在服务器本地直接执行 Docker Compose 重新编译并拉起服务，实现秒级热更新。**

yaml

```
stages:
  - deploy

variables:
  DOCKER_HOST: unix:///var/run/docker.sock

deploy_to_server:
  stage: deploy
  tags:
    - your-runner-tag # 替换成你部署在服务器上的 GitLab Runner 的 Tag
  only:
    - main # 仅在 main 分支提交时触发
  script:
    - echo "开始自动化构建与部署..."
    # 1. 停止并移除旧的业务容器（保持 Ollama 算力容器不动，避免重复下载模型）
    - docker compose down --remove-orphans
    # 2. 重新编译并后台启动业务容器
    - docker compose up --build -d
    # 3. 清理无用的 Docker 镜像残余
    - docker image prune -f
    - echo "项目 OmniMind AI 部署成功！"
```

请谨慎使用此类代码。

---

💡 个人转职提分：后续升级的“小心机”

当你把这套敏捷开发和部署流程跑通后，你的简历和面试说辞将直逼大厂高级工程师。你可以逐步尝试以下**进阶策略**：

1. **多卡负载均衡 (Model Sharding)**：如果在后续开发中，你发现 14B 模型不够聪明，你可以利用 vLLM 替代 Ollama，在配置中写入 `--tensor-parallel-size 4`，将一个 **70B 的超大模型横跨 4 张 V100** 运行。在面试中聊到“多卡张量并行调优”，含金量极高。
2. **GitLab CI 缓存优化**：在前端编译（`npm run build`）时，利用 GitLab CI 的 `cache` 功能缓存 `node_modules` 和 `.next/cache`，将部署时间从 5 分钟压缩到 30 秒内。