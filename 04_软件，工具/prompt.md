---
tags:
  - 技术/通用
---
>[!code]- first prompt
>Analyze this codebase to generate or update `.github/copilot-instructions.md` for guiding AI coding agents. Focus on discovering the essential knowledge that would help an AI agents be immediately productive in this codebase. Consider aspects like: - The "big picture" architecture that requires reading multiple files to understand - major components, service boundaries, data flows, and the "why" behind structural decisions - Critical developer workflows (builds, tests, debugging) especially commands that aren't obvious from file inspection alone - Project-specific conventions and patterns that differ from common practices - Integration points, external dependencies, and cross-component communication patterns Source existing AI conventions from `**/{.github/copilot-instructions.md,AGENT.md,AGENTS.md,CLAUDE.md,.cursorrules,.windsurfrules,.clinerules,.cursor/rules/**,.windsurf/rules/**,.clinerules/**,README.md}` (do one glob search). Guidelines (read more at https://aka.ms/vscode-instructions-docs): - If `.github/copilot-instructions.md` exists, merge intelligently - preserve valuable content while updating outdated sections - Write concise, actionable instructions (~20-50 lines) using markdown structure - Include specific examples from the codebase when describing patterns - Avoid generic advice ("write tests", "handle errors") - focus on THIS project's specific approaches - Document only discoverable patterns, not aspirational practices - Reference key files/directories that exemplify important patterns Update `.github/copilot-instructions.md` for the user, then ask for feedback on any unclear or incomplete sections to iterate.



### 第一部分：引路人文件（极简版）

这三个文件的作用仅仅是“路牌”，把所有 AI 工具都引导到同一个地方。这样你以后只需要维护一份 `AGENT.md`。

**📄 `.cursorrules` (给 Cursor 看)**

Markdown

```
# 全局指令
在回答任何问题或生成代码之前，必须严格阅读并遵循项目根目录下的 `AGENT.md` 文件。
Please read and strictly follow the `AGENT.md` file in the root directory before proceeding.
```

**📄 `.github/copilot-instructions.md` (给 Copilot 看)**

Markdown

```
# 全局指令
Please read and strictly follow the `AGENT.md` file in the root directory before proceeding.
```

_(OpenCode 通常默认会自动寻找并读取 `AGENT.md`，所以无需额外路牌)_

---

### 第二部分：核心大脑 📄 `AGENT.md` (你的通用模版)

这是你的“真身”，我把你之前的规则进行了专业级的结构化重组，并加入了对 CI/CD 和 PR 的联动引用。

Markdown

```
# 🤖 AI Agent 工作指南 (Project AGENT Rules)

## 1. 语言与沟通规范 (Language & Communication)
- **对话语言**：在聊天框解释代码、回答问题、讨论方案时，必须全程使用 **中文**。
- **代码与 Git**：在生成、修改代码时，代码内的函数注释 (Comments)、文档字符串 (Docstrings)、变量命名，以及 Git Commit Message，必须全程使用 **英文**。

## 2. 核心技术栈 (Tech Stack)
- **语言**：Python 3.11+
- **包管理器**：`uv` (⚠️ 绝对禁止使用 `pip`, `poetry` 或 `pipenv`)

## 3. 目录结构与架构 (Architecture)
- 核心业务代码主要位于 `src/` 和 `app/` 目录。
- 启动入口为根目录的 `main.py`。
- *(如果在此处拓展其他项目，简要写明核心文件夹用途，例如：`tests/` 为测试用例，`scripts/` 为独立脚本)*

## 4. 代码规范与质量控制 (Code Standards)
- **代码风格**：必须遵守 `Ruff` 的规范。
- **类型检查**：必须通过 `Mypy` [或 Pyright] 的严格静态类型检查。
- **类型注解**：所有新生成或修改的函数，必须包含完整的类型提示 (Type Hints，例如 `def foo(bar: str) -> int:`)。

## 5. 核心工作流与规范 (Workflows & Conventions)
- **依赖管理**：当需要添加新库时，请只提供 `uv add <package>` 或 `uv add --dev <package>` 命令。
- **运行命令参考**：如果你需要帮我编写构建、测试或部署命令，请优先查阅项目根目录下的 `Makefile`，复用里面已有的指令。
- **CI/CD 认知**：本项目的流水线配置在 `.github/workflows/ci.yml` [或 .gitlab-ci.yml]。在修改构建逻辑时，请确保不破坏现有的 CI 流程。
- **代码提交**：如果需要帮我生成 PR (Pull Request) 的描述，请务必严格按照 `.github/pull_request_template.md` 的格式和要求进行填写。

## 6. 强制自查清单 (Pre-flight Checklist)
在协助我完成一段重要的代码修改后，请主动建议或在终端中自动运行以下命令来验证代码质量：
1. `uv run ruff check src/ app/ main.py`
2. `uv run ruff format src/ app/ main.py`
3. `uv run mypy src/ app/ main.py`
```

---

### 🚀 为什么这套模版无懈可击？

1. **DRY 原则 (Don't Repeat Yourself)**：通过抽离引路人文件，你以后改规矩（比如把 Mypy 换成 Pyright，或者把 Python 3.11 升级到 3.12），只需要改 `AGENT.md` 这一处，Cursor 和 Copilot 都会瞬间同步。
    
2. **动态加载上下文**：模版里的第 5 点非常精髓。当你让 AI “帮我写个 PR” 时，它看到第 5 点，就会**自动去读取**你的 `pull_request_template.md`，然后完美填空，根本不用你废话。