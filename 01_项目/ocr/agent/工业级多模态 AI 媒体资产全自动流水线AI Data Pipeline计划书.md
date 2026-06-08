1. 项目定位与核心解决痛点

1.1 项目定位

本项目是一个面向企业生产力的**非交互式、高吞吐量媒体资产全自动生产与审计流水线**。系统通过代码逻辑强编排多 Agent 角色，实现“海报/视频生成 ➡️ 视觉自动测试 ➡️ 反思重绘 ➡️ 结构化存储”的工业化闭环。

1.2 核心工程痛点解决

- **消除生成不确定性**：引入多模态 Vision Agent 作为质量把控（QA），确保不合格资产（如文字穿模、肢体扭曲）零入库。
- **平滑应对速率限制（Rate Limits）**：通过异步并发控制与指数退避机制，在不触发云厂商 429 错误的前提下榨干 QPS 吞吐量。
- **分布式容错与断点续传**：引入基于数据库的任务状态机。若服务器中途断网或崩溃，重启后可精准从失败点恢复，拒绝重复生成。
- **全链路内存流式处理**：媒体资产在生成、测试、传输全过程中**“绝不落盘”**，完全在内存中以二进制流（Bytes）形式流转，极大提升 I/O 效率并降低分布式容器的磁盘开销。

---

2. 完整技术栈选型（全链路）

|架构分层|核心技术选型|选型核心理由|
|---|---|---|
|**基础语言与环境**|`Python 3.11+`|享受更快的异步 `asyncio` 性能与强类型注解支持。|
|**Agent 框架**|`Pydantic AI`|2026年最前沿的类型安全 Agent 框架，提供原生 Pydantic 运行时校验，完美支持结构化输出。|
|**LLM/VLM 模型底座**|`Google GenAI SDK`  <br>(Gemini 2.5 Flash / Imagen 3)|**Imagen 3** 负责高精度文本生图；**Gemini 2.5 Flash** 具备极高的多模态 Vision 速度与超低 Token 成本，适合做 QA 审计。|
|**高并发与限流调度**|`asyncio` + `tenacity`|`asyncio.Semaphore` 限制物理并发上限；`tenacity` 优雅实现指数退避重试（Exponential Backoff）。|
|**任务状态机 (DB)**|`PostgreSQL` / `SQLite`  <br>+ `SQLAlchemy` (Async)|提供事务支持，通过状态字段（Pending/Processing/Success/Failed）支撑断点续传。|
|**资产云存储**|`boto3` / `oss2`|支持标准 S3 / 阿里云 OSS 协议，提供内存中 `BytesIO` 的流式长传（Streaming Upload）。|
|**可视化控制台**|`Streamlit`|几十分钟内为非技术运营人员构建出“Prompt 模板配置、CSV 上传、任务看板”的 Web 界面。|
|**监控与日志**|`Loguru` + `Prometheus`|`Loguru` 负责全链路带异步上下文的日志追踪；`Prometheus` 监控当前的 QPS 吞吐率与 AI 坏图率。|

---

3. 系统详细架构与多 Agent 编排流水线 (Pipeline)

系统采用**编排式多智能体（Orchestrated Multi-Agent）**架构。Agent 之间不进行无序交谈，而是由 Python 主程序作为“导演”，通过控制数据流与 Pydantic 结构化对象来驱动状态流转。

3.1 核心 Agent 角色定义

1. **资产生成器（Generator Engine）**：接受 Python 主程序派发的 Prompt 任务，调用 Imagen 3 API 生成图片二进制流（Image Bytes）。
2. **视觉审计智能体（Vision QA Agent）**：基于 `Pydantic AI` 构建。输入为图片二进制流，输出必须为严格的 Pydantic 模型（包含布尔值、评分、原因）。
3. **提示词自适应修正器（Prompt Refiner Agent）**：当 QA 失败时激活，结合原始 Prompt 和 QA 失败原因，生成用于重绘的修正提示词。

3.2 数据流转拓扑（明确的 Outline）

```
[运营通过 Streamlit 提交批次任务]
          │
          ▼
[解析数字/参数，批量写入 DB 状态为 PENDING]
          │
          ▼
[扫描器捞取 PENDING，推入 Asyncio 任务队列] ◄──────┐ (若未达最大重试次数)
          │                                      │
          ├──► [信号量 Semaphore 控频限流]        │
          │               │                      │
          │               ▼                      │
          │    [Generator 调用 Imagen 3 生图]    │
          │               │                      │
          │               ▼                      │
          │    [Vision QA Agent 多模态视觉测试]   │
          │               │                      │
          │               ▼                      │
          │    [解析 PAI 返回的结构化测试报告]     │
          │               │                      │
          │               ├─── (测试失败) ───────┴──► [Refiner 修改 Prompt]
          │               │
          │          (测试成功)
          │               │
          │               ▼
          │    [内存流式上传至 S3/OSS]
          │               │
          │               ▼
          └──► [更新 DB 状态为 SUCCESS，释放信号量]
```

---

4. 详细分步实施计划（5大阶段）

阶段一：数据层设计与断点续传状态机搭建 (Data Layer)

- **步骤 1**：搭建数据库表 `task_batches`（批次表）与 `media_tasks`（单张任务表）。
    - _`media_tasks` 关键核心字段_：`id`, `batch_id`, `inject_number` (绑定的随机数), `status` (PENDING/PROCESSING/SUCCESS/FAILED), `current_retry_count`, `oss_url`, `qa_score`, `qa_failed_reason`。
- **步骤 2**：编写异步数据访问层（DAO）。实现“断点扫描函数”：每次启动时，通过 SQL 捞出所有 `status IN ('PENDING', 'FAILED') AND current_retry_count < 3` 的记录。

阶段二：异步并发控制与限流网关建设 (Core Infra)

- **步骤 1**：基于 `asyncio.Semaphore(QPS_LIMIT)` 封装核心调度器，确保任何时刻向谷歌服务器发送的请求数不超过安全阈值。
- **步骤 2**：集成 `tenacity` 重试组件。精细化配置错误捕获：
    - 若捕获到 `HTTP 429 / Rate Limit Exceeded` 异常 ➡️ 触发 `wait_random_exponential`（指数退避，如等待 1s, 2s, 4s, 8s...），并不计入任务自身的业务重试次数。
    - 若捕获到网络超时 ➡️ 触发常规重试。

阶段三：强类型多 Agent 闭环开发 (PAI Integration)

- **步骤 1**：利用 `Pydantic` 声明严格的组件契约 `QAReportSchema`。定义 `is_passed: bool`、`visual_score: int`、`defect_type: Literal['blur', 'text_error', 'artifact', 'none']`。
- **步骤 2**：使用 `Pydantic AI` 初始化 `qa_agent`。利用其内置的重试机制（`max_json_retry`），确保当大模型返回的 JSON 损坏时，框架能自动拦截并让大模型修正，保证 Python 主程序拿到的永远是合法的结构化对象。
- **步骤 3**：实现编排逻辑（导演代码）。
    
    python
    
    ```
    if qa_report.is_passed == False:
        # 调度 Refiner Agent 扩写提示词，任务重试计数 +1，重新推入队列
    else:
        # 进入阶段四
    ```
    
    请谨慎使用此类代码。
    

阶段四：无盘化流式媒体存储管道 (Storage Pipeline)

- **步骤 1**：配置 `google-genai` SDK，使 Imagen 3 返回原始图像的 `bytes` 字节，杜绝使用 `image.save('temp.jpg')` 等本地磁盘 I/O 操作。
- **步骤 2**：使用 `io.BytesIO(image_bytes)` 将内存中的数据封装为标准的流对象。
- **步骤 3**：调用云存储 SDK 的异步/线程池流式上传接口（Streaming Upload），将资产直接推送至云端，并立即获取外链 URL。

阶段五：Streamlit 后端看板与可观测性落地 (UI & Monitor)

- **步骤 1**：用 Streamlit 编写前端页面。
    - _左侧配置栏_：输入 Prompt 模板、调节测试严苛度阈值、配置并发数。
    - _中央控制区_：支持运营上传包含数字列表的 CSV 文件。一键点击触发 `asyncio.run()` 异步批处理核心。
- **步骤 2**：利用 Streamlit 的 `st.dataframe` 或进度条，绑定数据库状态，每隔 3 秒实现一次前端任务进度的动态刷新。
- **步骤 3**：可观测性埋点。使用 `Loguru` 的 `logger.contextualize(task_id=...)`，确保高并发时，控制台打印的每一条日志都能精准对应到是哪一个数字、哪张图正在被处理或被拒绝。

---

5. 项目可拓展性设计 (Scalability Blueprint)

作为 AI 应用工程师，在架构设计之初就必须预留未来的拓展接口：

1. **多模型交叉盲审（Cross-Validation）**：QA 层目前仅依赖 Gemini。未来可横向拓展为“评审团模式”：同时调用 Gemini 2.5 和 Claude 3.5 Sonnet，通过代码实现投票算法（如两个模型都投通过才算过），极大提升资产合规率。
2. **多媒体资产线纵向升级（Image to Video）**：Imagen 3 生成图后，可无缝对接视频生成模型 API。流水线的 QA Agent 只需升级为“视频抽帧检查器”，读取视频的头、中、尾三帧进行多模态 Vision 审计，即可将生图流水线无缝升级为**短视频矩阵大批量生产流水线**。

---

6. 使用目标人群与商业化边界

- **目标人群**：电商矩阵运营团队（批量生产带券码的海报）、新媒体矩阵号主（批量生产带随机数字或励志语录的短视频/封面）、游戏测试策划（大批量随机道具/Icon 美术资产打样审查）。
- **商业化优势**：直接手写这套 Pipeline 的核心在于**“省钱与高效”**。未来通过对接各大厂商的 **Batch API**，在夜间提交非实时大批处理任务，能为企业斩获高达 **50% 的大模型调用成本折扣**，这是任何低代码平台（Dify/Coze）在量产场景下无法企及的商业护城河。

---

7. 容器化部署方法 (Deployment Lifecycle)

项目采用**轻量级、无状态的容器化云原生部署**方案：

1. **基础镜像构建**：编写 `Dockerfile`。选用 `python:3.11-slim` 减小体积，仅安装 `google-genai`、`pydantic-ai`、`streamlit`、`sqlalchemy`、`tenacity` 核心依赖。
2. **本地/私有云常驻部署（Docker Compose）**：
    - 容器 A：`PostgreSQL` 负责持久化任务状态机。
    - 容器 B：你的 Python 流水线程序（内置 Streamlit 服务，对外暴露 8501 端口）。
3. **云原生 Serverless 弹性部署（强烈推荐，大厂标准架构）**：
    - 将整个流水线剥离 Streamlit，做成纯粹的**任务执行器（Worker）**。
    - 将其打包部署在 **阿里云函数计算（FC）** 或 **AWS Lambda** 上，通过消息队列（如 MQTT / RabbitMQ）进行事件驱动触发。
    - _优势_：当运营上传 10,000 个数字时，Serverless 瞬间拉起数十个容器并发吞吐，跑完之后容器自动销毁。无任务时不占用任何常驻服务器成本，实现极致的**按需计费（Pay-as-you-go）**。


没有s3之类的个人项目怎么办？
免费替代品——MinIO（本地私有化 S3）

如果你想练兵，提前体验标准的 S3 接口编程：

- **做法**：在本地电脑上下载并运行 **MinIO**（一个开源的、在本地运行的对象存储服务器）。它提供跟 AWS S3 **一模一样的 API 接口和 Python SDK（boto3）**。
- **价值**：你在本地连接 `localhost:9000` 进行开发。未来如果项目要上线 AWS，你**一行代码都不用改**，只需要把连接地址改成 AWS 的链接即可。

---
数据库方案呢？
**状态机数据库**：`SQLite` + `aiosqlite`（单文件数据库，无配置痛苦）

---
部署方案呢？
方案 B：Hugging Face Spaces（完全免费）

AI 圈最流行的个人作品展示舞台。

- **怎么用**：在 Hughinng Face 上建一个 Space，选 Streamlit SDK。把你本地的代码 push 进去。
- **优势**：完全免费，自带公网链接，且在 AI 社区里曝光度高，非常适合作为个人技术作品集（Portfolio）展示给未来的雇主。

---

你可以按照以下顺序**肉身零成本**地推进这个项目：

1. **第一步（当天可完成）**：新建一个 Python 项目，安装 `google-genai` 和 `pydantic-ai`，尝试用最简单的 `asyncio` 连写 3 个循环，把生成的图直接用 `open()` 存到本地 `outputs/` 文件夹。
2. **第二步（引入状态机）**：加入 `SQLAlchemy` + `SQLite`，实现任务的持久化。故意在程序跑到一半时按 `Ctrl+C` 强行终止，然后重启程序，看看它能不能**自动跳过已存在的图片**，只生剩下的图。
3. **第三步（界面化与外发）**：套上 `Streamlit` 壳子，在本地用 `streamlit run main.py` 预览。满意后一键发布到 `Streamlit Cloud`。