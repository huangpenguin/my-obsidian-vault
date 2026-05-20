
（_基于 `pjt-aks` 项目复盘总结_）

## 1. 极致的依赖与运行环境管理：引入 `uv`

摒弃了传统的 `pip`、`requirements.txt` 甚至 `Poetry`，全面拥抱了 Rust 编写的极速包管理器 **[uv](https://github.com/astral-sh/uv)**。

- **可复用做法：**
    
    - 使用 `uv sync` 替代 `pip install`，确保多环境下的依赖严格一致（基于 `uv.lock`）。
        
    - 使用 `uv run` 执行命令（如 `uv run pytest`、`uv run uvicorn`），无需手动激活虚拟环境，降低了团队协作的心智负担。
        
    - **价值：** 极大地提升了 CI/CD 流水线的构建速度和本地开发体验。
        

## 2. 严密的 CI/CD 流水线与自动化代码审查

项目在 `.github/workflows/` 下构建了极其规范的自动化流水线，并且引入了 AI 辅助。

- **可复用做法：**
    
    - **三道防线并行（CI）：**
        
        - **格式与 Lint：** 使用 [Ruff](https://docs.astral.sh/ruff/)（极速 Lint/Format 工具）代替 Flake8/Black（`uv run ruff check`）。
            
        - **静态类型检查：** 引入 [Pyright](https://microsoft.github.io/pyright/)，配合 `pyrightconfig.json` 保证 Python 类型的强约束，这在大型项目中对防止低级错误至关重要。
            
        - **单元测试与覆盖率：** 结合 `pytest` 和 `pytest-cov`，在 CI 中强制要求输出覆盖率报告（`--cov-report`）。
            
    - **AI 自动化 Code Review：** 接入 **[CodeRabbit](https://coderabbit.ai/)**（通过 `.coderabbit.yaml` 配置），在 PR 阶段自动用中文进行代码审查，提出性能优化、安全漏洞和规范约束的建议。
        
    - **价值：** 将代码质量把控前置到 PR 阶段，减少人工 Review 的基础工作量。
        

## 3. 驱动领域设计的目录架构 (DDD 雏形)

摒弃了将所有逻辑揉杂在 API 路由层的做法，采用了清晰的职责分离（Separation of Concerns）。

- **可复用做法：**
    
    - `app/api/`：**接口层**。只负责接收请求、参数校验、调用业务逻辑、返回响应。
        
    - `src/core/`：**核心业务逻辑层**（Service 层）。存放真正的处理算法和规则，与外部框架解耦，方便单独测试。
        
    - `src/schemas/`：**数据契约层**。集中管理所有 Pydantic Models 和请求/响应结构。
        
    - `src/clients/`：**外部依赖层**。将对外部系统（如数据库、第三方 API）的调用封装在这里。
        
    - **价值：** 无论以后是换 Web 框架，还是业务极速膨胀，代码结构都不会乱，测试也极其好写（因为业务逻辑与路由解耦）。
        

## 4. 规范的敏捷开发与分支策略 (GitHub Flow)

README 中详细定义了一套基于 Issue 的工作流，这对团队协作极具指导意义。

- **可复用做法：**
    
    - **强制绑定 Issue：** 开发新功能必须先有 Issue，分支命名严格遵循 `<type>/<issue-number>-<short-description>`（如 `feature/123-add-pdf-extraction`）。
        
    - **标准化 PR 模板：** 根目录下放置 `.github/PULL_REQUEST_TEMPLATE.md`，规范化提交说明。
        
    - **语义化 Commit：** Commit message 关联 Issue（例如 `Closes #123`），实现 PR 合并时自动关闭 Issue。
        
    - **价值：** 需求可溯源，开发进度一目了然，方便后期 Review 和回滚。
        

## 5. 全局配置与环境变量分离

良好的工程从不把配置硬编码在代码里。

- **可复用做法：**
    
    - 提供明确的 `.env.example`，新开发者一拉取代码就知道需要配置哪些环境变量。
        
    - 统一的 `project_setting.py` 文件，在此处加载并验证所有的环境配置，然后在其他模块中统一引用这个配置文件。
        
    - **价值：** 保护敏感信息，同时确保应用启动时就能校验缺失的配置，而不是运行到一半才崩溃。
        

## 6. 系统级的可观测性 (Observability)

虽然项目中用的是 Langfuse 针对 LLM 进行追踪，但其思想可以复用到传统后端。

- **可复用做法：**
    
    - 在核心链路上打 Tag（例如 Session ID、耗时、核心参数）。
        
    - 任何复杂的处理流（例如数据清洗管道、订单状态流转）都应该有日志或追踪面板，记录每一步的输入和输出。
        
    - **价值：** 线上出现 Bug 时，不再依赖盲猜，而是有据可查（Traceability）。