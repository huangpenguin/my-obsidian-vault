### 🚀 基本用法（你可以现在就试试）：

```bash

tmux        # 启动 tmux 会话
Ctrl + b    # 这是默认前缀键（先按它再按其它命令）

# 常用命令（Ctrl + b 后按的键）：
%           # 左右分屏
"           # 上下分屏
o           # 在面板间切换
d           # 分离当前会话（后台运行）
x           # 关闭当前面板,比如有俩面板的话就需要关俩次
c           # 新建窗口
n/p         # 切换窗口（next / previous）,另一个tmux

s           #会话选择器，之后按数字就可以切换

# 恢复 tmux 会话：
tmux attach

tmux ls
tmux kill-session -t mysession
tmux kill-server

```

[tmux与nohup](https://app.notion.com/p/tmux-nohup-22d1455baece80389bf3ff117c86d08b?pvs=21)

tmux new -s test # 新建会话  
Ctrl + b → Shift + 5 # 分屏（水平  
Ctrl + b → Shift + ' # 分屏（垂直）  
Ctrl + b → 方向键 # 面板之间移动  
Ctrl + b → d # 分离  
tmux attach -t test # 重新连接

---

tmux移动光标查看内容

- **进入复制模式**：
    
    - 按下您的 `tmux` 前缀键（Prefix），**默认为 `Ctrl + b`**。
    - 紧接着，按下 `[` 键。
    
    现在您就进入了可以自由滚动的“复制模式”。您会看到右上角出现一个黄色的 `[0/xxx]` 计数器，表示您在缓冲区的位置。
    
- **在复制模式中导航**：
    
    - **上下箭头 `↑` `↓`**：逐行上下移动。
    - **`PageUp` / `PageDown`**：整页上下翻动（在macOS上可能是 `fn + ↑` / `fn + ↓`）。
    - 如果您熟悉 `vi`，也可以用 `k` (上), `j` (下), `g` (到顶部), `G` (到底部) 来导航。
- **退出复制模式**：
    
    - 按下 `q` 键或 `Esc` 键即可退出，返回到命令提示符。

---

## ✅ 本地 iTerm2 分屏方法（不需要 tmux）

iTerm2 自带非常强大的分屏功能，无需 tmux 也能完成：

### 分屏快捷键：

|动作|快捷键|
|---|---|
|**垂直分屏（上下）**|`⌘ + D`|
|**水平分屏（左右）**|`⌘ + ⇧ + D`|
|**在当前分屏新建标签页**|`⌘ + T`|
|**切换分屏窗口**|`⌘ + Option + 方向键`|
|**关闭当前分屏**|`⌘ + W`|