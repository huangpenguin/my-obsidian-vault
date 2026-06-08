全栈技术栈汇总（技术选型）

- **核心逻辑与 Agent 框架**：`Python 3.11+` + `Pydantic AI`（提供强类型、动态工具调用与自动重试机制）
- **模型接入**：`google-genai` SDK（调用 Gemini 2.5 Flash 用于多模态测试，Imagen 3 用于图像生成）
- **并发与调度引擎**：`Asyncio`（异步网络请求） + `Loguru`（工业级日志记录，用于追踪哪张图卡在了哪个 Agent）
- **数据层**：`SQLAlchemy` + `PostgreSQL`（记录任务状态、随机数种子、QA 评分，实现断点续传）
- **资产存储**：`boto3`（标准 S3 协议，自动将过审的图片/视频上传至 AWS S3 或阿里云 OSS）
- **轻量可视化前端**：`Streamlit` 或 `Gradio`（用于非技术人员配置 Prompt 模板、查看任务进度和 QA 坏图率）

2. 系统架构与拓展性设计

- **当前目标（V1.0）**：`固定Prompt + 随机数 ➡️ Imagen 3 ➡️ Gemini 视觉QA测试 ➡️ 合格后上传OSS`。
- **未来可拓展性（V2.0+）**：
    - **横向拓展（多模型混合测试）**：测试 Agent 不仅用 Gemini，还可以同时调用 Claude 3.5 Sonnet 进行双重交叉盲审（Cross-validation），只有两个 AI 都通过才算合格。
    - **纵向拓展（多媒体流）**：从生图拓展到**生视频**（如接入 Veo 或 Sora API），测试 Agent 演进为“视频抽帧测试 Agent”，专门检查视频连贯性和闪烁度。
    - **自适应反思机制**：系统根据历史被拒（Failed）的图片特征，自动总结出“Imagen 3 的避坑词（Negative Prompts）”，动态注入后续的生成中。

4. 部署方法（如何交付与上线）

为了保持项目的轻量与工业化，建议采用**容器化云原生部署**：

- **开发阶段**：直接使用 `python main.py` 在本地或本地 Streamlit 界面运行。
- **生产部署**：
    1. 编写 `Dockerfile` 将整个 Python 管道打包成一个镜像。
    2. **Serverless 部署**（推荐）：部署在 AWS Lambda 或 阿里云 FC（函数计算）中，配置定时触发器（Cron Job）或队列触发。因为你的系统是管道式的，非常契合 Serverless 按需计费、弹性扩容的特点。
    3. **常驻部署**：使用 `Docker Compose` 在一台轻量云服务器（VPS）上运行，后台用 `Supervisor` 或 `Docker` 保持进程常驻，通过 Streamlit 提供 Web 访问端点。

---
一、 核心工程优化点精细化设计

1. 高并发与速率限制（Rate Limits）管理

谷歌 API 通常有两重限制：**QPS（每秒请求数）** 和 **RPM/TPM（每分钟请求数/每分钟 Token 数）**。

- **核心方案**：使用 Python `asyncio` 配合 **`asyncio.Semaphore`（信号量控制并发数）** + **指数退避重试（Exponential Backoff）**。
- **最佳实践**：不要自己手写复杂的令牌桶算法，推荐使用成熟的 `tenacity` 库，它能完美处理“遇到 429 报错（Rate Limit）时自动等待并重试”的逻辑。
- 
2. 输入输出数据管理

- **输入端管理**：放弃让运营看代码。使用 **`Streamlit`** 快速搭建一个 Web 界面。运营在网页上输入 Prompt 模版（如 `"一张写着数字 {number} 的未来感海报"`），并上传一个包含数千个随机数字的 CSV 文件，点击“开始任务”。
- **输出端管理（流式上传）**：图片在内存中生成后，**绝不落盘（不写本地硬盘）**。直接将二进制字节流通过内存缓冲区（`io.BytesIO`）推送到 **阿里云 OSS / AWS S3**，获取外链 URL 后，将 `任务ID | 随机数 | Prompt | OSS_URL | 生成时间 | 状态` 存入数据库。

3. 异常处理与自动断点续传（通过数据库状态机实现）

要做到“第 888 张图断网，重启后自动从 889 开始”，必须引入**任务状态机（Task State Machine）**。

- **数据库设计**：创建一张 `image_tasks` 表。
    - 字段：`id` (自增), `task_batch_id` (批次ID), `random_number` (数字), `status` (状态: `PENDING`/`PROCESSING`/`SUCCESS`/`FAILED`), `oss_url` (链接)。
- **断点续传逻辑**：
    1. 运营上传 5000 个数字时，系统一次性在数据库生成 5000 条 `PENDING` 记录。
    2. 程序启动或意外崩溃重启时，代码第一步去数据库执行：`SELECT * FROM image_tasks WHERE task_batch_id = 'xxx' AND status IN ('PENDING', 'FAILED')`。
    3. 这样每次只捞取**未成功**的任务，天然实现断点续传。

---

二、 多 Agent 调度：如何控制它们之间的协作？

> **提问回复**：是的，**完全可以通过传递数据、Output 加代码逻辑来控制，这种模式被称为“编排式多智能体（Orchestrated Multi-Agent）”**。

在生产环境中，**不要**让 Agent 之间像微信聊天一样自由、无序地对话（那种模式极难控制且非常烧 Token）。作为 AI 应用工程师，你应该用**确定性的代码逻辑（如状态机、if-else、或有向无环图 DAG）**来充当“导演”，控制数据的流转。

3 角色 Agent 调度流程图解：

`[输入数据]` ➡️ **Generator Agent** (生成图) ➡️ `[图片二进制]` ➡️ **QA Agent** (看图审计) ➡️ `[Pydantic 结构化报告]` ➡️ **代码逻辑控制(导演)**

- **如果 QA 报告 `is_passed == True`** ➡️ 触发 **Storage Agent / 存储函数** ➡️ 存入数据库 ➡️ 结束。
- **如果 QA 报告 `is_passed == False`** ➡️ 代码控制将 `错误原因` 重新打包 ➡️ 丢回给 **Generator Agent** 重新生成（限制最多循环 3 次）。

这正是 **Pydantic AI** 这个框架的核心设计哲学。它不希望你搞空洞的“Agent 自主聊天”，而是希望你用普通的 Python 代码（函数、循环）去精准控制 Agent 的输入和输出。

---

工业级多模态 AI 媒体资产流水线（AI Data Pipeline）实施计划书

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

---