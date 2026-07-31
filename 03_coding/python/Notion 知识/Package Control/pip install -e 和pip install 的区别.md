---
title: "pip install -e .和pip install .的区别"
publish: false
tags: ["Python"]
---
# pip install -e .和pip install .的区别

**本地源代码的优先级**：

- 如果你使用 `pip install -e .`（开发模式）来安装，Python 会在虚拟环境中建立一个指向本地项目目录的符号链接。这样一来，你的项目文件夹中的源代码会被直接加载，无需每次修改代码后重新安装。
- 但是，如果你使用的是 `pip install .` 而不是开发模式，那么每次修改项目代码后，都需要重新安装包以应用更改。

---

### 问:如果当前项目文件夹下只有pyproject.toml，那么还可以使用pip install .吗

如果你的项目文件夹下只有 `pyproject.toml` 文件，而没有 `setup.py` 或 `setup.cfg` 文件，你仍然可以使用 `pip install .` 来安装项目，但需要确保 `pyproject.toml` 文件配置正确并指定了构建系统。通常，`pyproject.toml` 文件需要包含以下部分：

1. **指定构建工具**：例如，使用 `setuptools` 或 `poetry` 作为构建工具。
2. **定义项目的基本信息和依赖项**。

以下是一个典型的 `pyproject.toml` 配置示例，使用 `setuptools` 作为构建工具：

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "your_project_name"
version = "0.1.0"
description = "A description of your project"
authors = [
    { name="Your Name", email="your.email@example.com" }
]
dependencies = [
    "aenum",
    "torch",
    "numpy",
    "pandas",
    "matplotlib",
    "pynput",
    "einops",
    "bayesian-torch",
    "opencv-python",
    "pyrealsense2"
]

```

### 使用 `pip install .` 安装项目

配置好 `pyproject.toml` 文件后，运行以下命令即可安装项目：

```bash
pip install .

```

这会告诉 `pip` 使用 `pyproject.toml` 中指定的构建系统（如 `setuptools` 或 `poetry`）来安装项目和依赖项。

### 额外提示

- 如果你使用 `poetry`，则不需要 `pip install .`，可以直接使用 `poetry install` 安装依赖和项目。
- 如果项目的 `pyproject.toml` 文件配置不完整或缺少必要的构建系统设置，`pip install .` 可能会失败，所以要确保 `[build-system]` 部分配置正确。

这样配置后，`pyproject.toml` 就能完全替代 `setup.py` 或 `setup.cfg`，用于安装和管理项目的依赖。

### 问：那为什么我下的代码中pyproject.toml和setup.cfg都存在

`pyproject.toml` 和 `setup.cfg` 同时存在的情况在一些项目中是比较常见的。原因通常与**兼容性**和**灵活性**有关。以下是一些可能的原因和用途：

### 1. **逐步迁移到 `pyproject.toml`**

一些项目正在从传统的 `setup.py` 和 `setup.cfg` 文件过渡到 `pyproject.toml`，因为 `PEP 518` 标准建议使用 `pyproject.toml` 来定义构建系统的依赖关系。然而，由于完全迁移到 `pyproject.toml` 需要调整工具和配置，很多项目选择在过渡期同时保留 `setup.cfg`。

- **目的**：`pyproject.toml` 可以定义项目的构建系统（如 `setuptools`、`poetry` 等）和依赖项，而 `setup.cfg` 则保持原有的配置，用于项目安装和分发。这样可以兼容支持 `setup.cfg` 的旧工具（如一些自动化打包和构建工具）和新标准。

### 2. **特定工具的配置需求**

一些工具或插件在项目根目录查找特定的配置文件，例如 `setup.cfg`。尽管 `pyproject.toml` 能够定义大多数设置，但部分工具的特定配置项（如测试、格式化或代码分析工具）仍可能依赖于 `setup.cfg` 或 `.flake8` 等文件。

- **目的**：在 `setup.cfg` 中存放工具的配置选项，而使用 `pyproject.toml` 进行构建系统和依赖管理，从而满足不同工具的需求。

### 3. **兼容性和扩展性考虑**

虽然 `pyproject.toml` 能够取代很多 `setup.cfg` 的功能，但部分早期的构建工具和自动化脚本可能尚未完全支持 `pyproject.toml` 格式。因此，一些项目选择同时保留 `setup.cfg` 以确保兼容性，尤其是在需要支持老旧的自动化构建和部署环境时。

### 4. **配置的分类管理**

有些项目将**构建和依赖管理**配置放在 `pyproject.toml` 中，而将**项目的元数据和工具设置**保存在 `setup.cfg` 中，以便配置更加清晰和模块化。

- **例如**：在 `pyproject.toml` 中定义构建系统和依赖关系，而在 `setup.cfg` 中设置项目的元信息（如包名称、版本号、作者等）或工具的配置。

### 示例：`pyproject.toml` 和 `setup.cfg` 的组合使用

### `pyproject.toml` 示例（主要用于定义构建系统）

```toml
toml
复制代码
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"

[tool.black]
line-length = 88

```

### `setup.cfg` 示例（用于项目元数据和额外的工具配置）

```
ini
复制代码
[metadata]
name = your_project_name
version = 0.1.0
description = A description of your project

[options]
packages = find:
install_requires =
    aenum
    torch
    numpy

```

### 总结

两者同时存在是为了兼容和满足不同工具的需求。逐步迁移的项目或使用多种配置工具的项目中，`setup.cfg` 和 `pyproject.toml` 共存能够确保灵活性和兼容性。
