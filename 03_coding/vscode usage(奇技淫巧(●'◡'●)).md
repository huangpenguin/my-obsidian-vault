---
tags:
  - 技术/工具/VSCode
  - 技术/软件工程
  - 技术/开发环境
---

## 🎛️ 核心枢纽：命令面板与搜索 (Command Palette)

VS Code 的精髓在于一个无所不能的顶部输入框。掌握它，你几乎可以脱离鼠标完成所有操作。

> [!note] 辨析：找文件 vs 找命令 (Obsidian 用户必看)
> - **找文件 (Path/Project Files)**：`Cmd / Ctrl + P`
>   弹出的输入框用来搜索**文件名字**。比如输入 `main` 可以快速跨文件夹打开 `main.py`。
> - **找命令 (Command Palette)**：`Cmd / Ctrl + Shift + P` 或 `F1`
>   加上了 `Shift` 键，代表切换了操作维度。此时弹出的框带有 `>` 前缀，用来**执行动作**（如格式化、调出 Git、换主题）。
> - **结论**：VS Code 里的 `Cmd + Shift + P`，才等价于 Obsidian 里的 `Cmd + P`。

### 🪄 魔法前缀：输入框首字母的隐藏功能
打开命令面板后，输入框默认带有 `>` 符号。如果你删掉它或输入其他特定符号，面板会瞬间切换成其他强大的辅助工具：

| 输入符号 | 功能类型 | 实用场景举例 |
| :--- | :--- | :--- |
| **`>`** (默认) | **执行命令** | 输入 `reload` 重启窗口，或 `theme` 换皮肤 |
| *(清空前缀)* | **搜索文件** | 直接输入文件名，快速跨文件夹打开文件 |
| **`@`** | **文件内跳转** | 输入 `@函数名` 或 `@变量名`，在当前代码中精准定位 |
| **`@:`** | **符号分类** | 将当前文件里的函数、类、变量分类展现，方便浏览 |
| **`#`** | **全局搜索** | 在整个项目的所有文件里寻找某个函数或类的定义 |
| **`:`** | **跳转行号** | 输入 `:50` 直接让光标跳到代码第 50 行 |
| **`?`** | **帮助菜单** | 忘记符号功能时，列出所有可输入的符号及其作用 |

### ⚡ 高频实用命令推荐 (带 `>` 模式)
在 `>` 执行命令模式下，支持模糊搜索（只需输入几个字母即可命中）。以下是开发者最常用的命令：
*   **Developer: Reload Window**：重新加载窗口（插件卡死、配置刚改完需要生效时的救命神器）。
*   **Format Document**：格式化当前文件代码。
*   **Preferences: Color Theme**：快速更换主题颜色。
*   **Preferences: Open Keyboard Shortcuts**：查看和自定义所有快捷键。
*   **Change Language Mode**：强制修改当前文件的代码解析语言（例如把纯文本 `text` 强制识别为 `python` 以获得语法高亮）。

---

## 🖱️ 鼠标与快捷键进阶操作

熟练使用以下快捷操作，可以大幅提升代码编辑和重构的效率：

*   **鼠标选中技巧**：
    *   **双击**：选中当前单词。
    *   **三击**：直接选中整行。
    *   **点击行号**：将鼠标移动到行号左侧，点击一下即可选中整行。
*   **键盘编辑快捷键**：
    *   **多光标编辑**：按住 `Alt` 并点击鼠标左键，可以在不同位置创建多个光标，实现同时编辑。
    *   **多处同名选中**：`Ctrl + D`，可以向下依次选中相同的单词（Find Match），方便批量修改。
    *   **行移动与复制**：
        *   `Alt + ↑/↓`：将当前行直接剪切并上下移动。
        *   `Alt + Shift + ↑/↓`：向上/向下复制并粘贴当前行。
    *   **快速多行注释**：`Ctrl + L` 可逐行向下选中并高亮（相当于鼠标向下滑动），随后配合 `Ctrl + /` 即可完成多行注释。

## 📂 文件与代码管理技巧

> [!tip] 快速创建多层文件夹
> 在新建文件时，如果需要将文件放在尚未创建的文件夹中，直接在文件名里使用斜杠即可（例如输入 `my_code/code1/code2/main.py`），VS Code 会自动帮你把多层目录一起建好。

> [!abstract] 优雅地查找与重命名代码
> 右键点击想要操作的变量或函数：
> *   **Rename Symbol (重命名符号)**：一键修改所有相关引用（快捷键 `F2`）。
> *   **Find all references (查找引用)**：查看该函数/变量在项目中的哪些地方被调用。
> *   **Go to Implementations (查找实现)**：查看接口或抽象方法的所有具体实现（OOP 面向对象编程中非常常用）。

---

## 🧩 核心插件推荐 (Extensions)

根据项目需求，以下是强烈推荐安装的扩展，已按用途分类：

*   **🐍 Python 开发环境**
    *   `Python` / `Jupyter`：官方核心开发扩展。
    *   `Ruff`：极速的 Python 代码分析与 linter 工具。
    *   `autopep8`：专注于自动格式化 Python 代码，使其强制符合 PEP 8 规范。
    *   `autoDocstring`：快捷生成规范的 Python Docstring 注释。
    *   `python indent`：优化 Python 的自动缩进体验。
*   **📝 Markdown 与文档**
    *   `Markdown All in One`：提供快捷键、目录生成等 Markdown 全能支持。
    *   `Markdown Preview Enhanced`：提供更强大的 Markdown 实时预览功能。
*   **📊 数据与排版辅助**
    *   `rainbow csv` / `edit csv`：以不同颜色高亮 CSV 文件的不同列，并提供便捷编辑。
    *   `indent rainbow`：为缩进添加交替的颜色，防止括号和层级看瞎眼。
    *   `Trailing Spaces`：高亮显示代码行尾多余的空格。
*   **🌐 版本控制与远程开发**
    *   `SVN / git` / `git history`：版本控制必备，方便查看提交历史和差异。
    *   `Remote Development Extensions` / `Remote - SSH`：远程连接服务器开发的官方神器。
*   **🛠️ 实用工具与 UI**
    *   `Path Intellisense`：自动补全文件路径。
    *   `Draw.io Integration`：直接在 VS Code 里画流程图（创建 `.drawio` 或 `.dio` 文件即可调出画板面板）。
    *   `Luna paint`：内置轻量级图像编辑器。
    *   `competitive programming helper`：算法竞赛/刷题辅助工具。

---

## ⚙️ C++ 多文件项目编译配置 (tasks.json)

在 VS Code 中开发多文件 C++ 项目时，需要通过 `.vscode/tasks.json` 来配置编译任务。

**准备工作**：确保已安装 C++ 编译器（如 `g++`）以及 VS Code 的 C++ 扩展。然后在项目根目录下创建 `.vscode/tasks.json` 文件。

### 标准配置模板
以下是一个可以编译多个 `.cpp` 文件的基础配置，输出文件名为 `main`：

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build",
      "type": "shell",
      "command": "g++",
      "args": [
        "-g",
        "main.cpp",
        "CLIApp.cpp",
        "Parser.cpp",
        "Calculator.cpp",
        "Memory.cpp",
        "-o",
        "main"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$gcc"],
      "detail": "Compile the whole project."
    }
  ]
}