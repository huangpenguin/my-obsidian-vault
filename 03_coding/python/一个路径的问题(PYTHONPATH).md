---
tags:
  - 技术/软件工程
---

这是一个非常经典的问题！千万不要觉得是你没搞懂，**几乎所有刚接触 Python 项目结构的人，都曾被 `PYTHONPATH` 折磨过。**

为了让你彻底明白，我们不用复杂的计算机术语，用一个**“快递员找地址”**的比喻来解释。

### 1. Python 导入模块的“默认潜规则”

想象 Python 是一个快递员。当你的脚本 `extract_drainage_height.py` 里面写了一句：

Python

```
from cad.utils import parser
```

这就是给快递员派单，让他去找叫 `cad` 的包裹。

默认情况下，快递员**只会去两个地方找**：

1. **系统的全局仓库**（就是你用 `pip install ezdxf` 安装第三方库的地方）。
    
2. **当前运行的脚本所在的那个文件夹**（也就是 `src/cad/` 目录）。
    

### 2. 为什么不用 PYTHONPATH 会报错？

如果不加 `PYTHONPATH=src`，直接运行： `python src/cad/extract_drainage_height.py`

当快递员看到 `from cad.utils import ...` 时，他会在**当前脚本所在的文件夹**（`src/cad/`）里面去找一个叫 `cad` 的文件夹。

但他找不到！因为他**已经在 `cad` 这个文件夹里面了**，他四处环顾，只能看到 `extract_drainage_height.py` 等文件，根本看不到一个叫 `cad` 的顶级目录。于是程序直接崩溃报错： `ModuleNotFoundError: No module named 'cad'`

### 3. `PYTHONPATH=src` 施了什么魔法？

`PYTHONPATH` 是一个环境变量，它的作用是**强行给快递员增加一个搜索地址**。

当你在命令前面加上 `PYTHONPATH=src` 时，你是在告诉 Python：

> _"嘿！除了默认的地方，**请你把 `src` 这个文件夹也当作一个顶级的起点（根目录）** 去搜索！"_

这时候，快递员站在 `src` 门口往里一看： “哇！里面正好有个文件夹叫 `cad`，里面还有个 `utils.py`！” 于是 `from cad.utils import parser` 这句代码瞬间就跑通了。

---

### 总结：为什么总是需要它？

这其实是由于你们项目的**目录结构**决定的。

你们的项目把所有代码都放在了 `src` 这个文件夹下（这在现代 Python 工程中是非常标准且推荐的做法，叫 `src-layout`）。但是，当你要**直接把里面的某个子文件当成脚本来运行**时，Python 会“迷路”，不知道项目的根目录在哪里。

所以，`PYTHONPATH=src` 就像是给 Python 的一个**导航仪**，明确告诉它：“**从 `src` 开始往下找代码。**”

**一个小提示**： 你现在用的命令是以 `uv run` 开头的。`uv` 是一个非常先进且非常快的 Python 工具。通常在一个配置完善的工程里（如果有 `pyproject.toml` 文件并且正确安装了本地包），是不需要手动写 `PYTHONPATH` 的。但为了确保绝对不出错，你们的文档或同事让你加上了这一句，这是一种**最稳妥、百分之百能跑通**的做法。

以后只要看到带有 `src/` 目录的 Python 项目，再遇到 `ModuleNotFoundError`，你第一反应就可以是：“噢，忘了给快递员设 `PYTHONPATH=src` 导航了！”