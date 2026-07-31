---
title: "pytest"
publish: false
tags: ["OCR","项目实践"]
---
# pytest

```markdown
uv run pytest tests/test_api.py::test_process_production_request_with_real_pdf -v -s

### 1. `uv run`
- `uv` 是一个类似 `pip` 和 `venv` 的新一代 Python 包管理工具，性能很高。
- `uv run` 类似于 `poetry run` 或 `pipenv run`，表示在当前虚拟环境中运行某个命令。
    - 它会自动在 `uv venv` 创建的虚拟环境中查找依赖并运行命令。

👉 所以这一部分意思是：**在 uv 的环境中运行接下来的命令**

---

### 2. `pytest`

- 启动 Python 的测试框架 `pytest`，它会自动识别 `test_*.py` 文件中的测试函数。

---

### 3. `tests/test_api.py::test_process_production_request_with_real_pdf`

这部分是 **指定具体的测试目标**，语法为：

```

<路径>::<测试函数名>

```

- `tests/test_api.py`：你项目中的某个测试文件。
- `test_process_production_request_with_real_pdf`：你要运行的一个具体测试函数。

👉 这意味着只运行这个文件中的这个函数，**不会运行其他测试**

---

### 4. `v`

**verbose 模式**，详细显示测试运行情况，比如：

- 哪些测试运行了
- 运行的顺序
- 每个测试是通过、失败、跳过等

---

### 5. `s`

**不要捕获标准输出**（stdout）

- 默认情况下，`pytest` 会捕获 `print()` 或日志的输出，只在测试失败时显示。
- `s` 会关闭这个功能，**让你看到所有输出**，比如你写的 `print(...)` 内容。
```

### 测试类

### 🔧 命令结构：

```bash
uv run pytest tests/test_production_request.py::TestProductionRequestWithRealPDFs::test_single_production_request_pdf -v -s
```

---

### 💡 每部分含义：

| 部分 | 解释 |
| --- | --- |
| `uv run` | 使用 [`uv`](https://github.com/astral-sh/uv) 工具来运行 Python 命令，相当于在虚拟环境中运行程序（类似 `poetry run` 或 `pipenv run`）。 |
| `pytest` | 测试框架 `pytest` 的命令。 |
| `tests/test_production_request.py` | 要测试的 Python 文件路径。 |
| `::TestProductionRequestWithRealPDFs` | 指定要运行的测试类名。 |
| `::test_single_production_request_pdf` | 指定测试类中的一个特定测试方法。 |
| `-v` | verbose 输出，显示更详细的运行信息（比如测试名）。 |
| `-s` | 允许 `print()` 的内容直接输出到终端（不屏蔽 stdout）。 |
