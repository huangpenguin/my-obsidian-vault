---
title: "commit info规范"
publish: false
tags: ["Git"]
---
# commit info规范

| **前缀** | **英文全称** | **含义** | **例子** |
| --- | --- | --- | --- |
| **feat** | Feature | **新功能** (最常用) | `feat: add voice input support` |
| **fix** | Fix | **修 Bug** | `fix: crash when api key is missing` |
| **docs** | Documentation | **只改了文档** | `docs: update README.md` |
| **style** | Style | **格式调整** (空格、缩进，不影响代码运行) | `style: format code with black` |
| **refactor** | Refactor | **重构** (代码结构调整，没加新功能也没修Bug) | `refactor: optimize database query` |
| **test** | Test | **增加或修改测试用例** | `test: add unit test for login` |
| **chore** | Chore | **杂活** (改配置、依赖更新、Git忽略文件等) | `chore: update .gitignore` |
| **ci** | CI | **部署脚本、CI/CD 配置** | `ci: update deploy.sh permissions` |

**CI** 是 **Continuous Integration（持续集成）** 的缩写。

在 Git 的提交信息（Commit Message）规范（通常称为 **Conventional Commits**）中，`ci:` 是一个标准的**前缀**，用来告诉看代码的人：**“这次修改没有改动业务代码，而是改动了构建、部署或自动化的流程。”**

### 1. 为什么你的这个改动属于 `ci`？

你修改的是 `deploy.sh`。

- 这个脚本**不是** Python 业务代码（比如聊天逻辑）。
- 它是用来**构建镜像、推送到仓库、部署到 Azure** 的工具脚本。
- 这就属于“集成”和“部署”的范畴。

所以，用 `ci: ...` 作为开头是非常精准的。它告诉别人（或者未来的你自己）：**“这次提交改的是部署脚本，跟代码功能无关。”**
