---
title: "使用 screen 或 nohup 放置后台进程"
publish: false
tags: ["Linux"]
---
# 使用 screen 或 nohup 放置后台进程

## tmux?

### 为什么要使用 `screen` 或 `nohup`？

- **防止会话中断：** 即使你断开 SSH 连接，你的程序也会继续运行。
- **长时间任务：** 非常适合运行机器学习训练、数据处理、软件编译等需要很长时间的任务。

---

当你在 SSH 会话中运行一个命令时，如果 SSH 连接断开（例如网络问题或你关闭了终端），那么正在运行的命令也会随之终止。`screen` 和 `nohup` 就是用来解决这个问题的，它们允许你的程序在后台持续运行。

### 方法一：使用 `screen` (推荐)

## 一、安装 `screen`

在 Ubuntu 上通常已经预装，如未安装可执行：

```bash
sudo apt update
sudo apt install screen
```

---

## 🏁 二、基本用法

### 1. 创建一个新会话

```bash
screen
```

或者指定会话名称（推荐）：

```bash
screen -S mysession
```

进入 screen 后，你会看到一个新的终端界面，就像普通 shell。

---

### 2. 暂时离开（detach）会话

在 screen 中按下以下组合键退出当前 screen 会话，但保持它在后台运行：

```
Ctrl + a，松开后再按 d
```

> 这叫 detach，screen 会话仍在后台运行。
> 

---

### 3. 查看后台运行的会话

```bash
screen -ls
```

输出示例：

```

There is a screen on:
    12345.mysession    (Detached)
1 Socket in /run/screen/S-username.
```

---

### 4. 重新连接（attach）到某个会话

```bash
screen -r mysession
```

或者使用 ID：

```bash
screen -r 12345
```

> 如果只有一个 screen 会话，可以直接用 screen -r 恢复。
> 

---

### 5. 杀死某个会话（退出并关闭）

在 screen 中执行：

```bash
exit
```

或在 shell 中强行关闭某个会话：

```bash

screen -S mysession -X quit
```

---

## ⚙️ 三、常用快捷键（在 screen 中按下 `Ctrl+a` 后再按下面的键）

| 快捷键 | 功能说明 |
| --- | --- |
| `Ctrl+a` `d` | 暂时离开 screen（detach） |
| `Ctrl+a` `c` | 新建一个窗口（相当于开了个 tab） |
| `Ctrl+a` `n` | 下一个窗口 |
| `Ctrl+a` `p` | 上一个窗口 |
| `Ctrl+a` `"` | 列出所有窗口并选择 |
| `Ctrl+a` `A` | 给当前窗口重命名 |
| `Ctrl+a` `k` | 杀死当前窗口 |
| `Ctrl+a` `?` | 查看所有快捷键 |

---

## 💡 四、实用示例：后台运行脚本

假设你想运行一个长时间脚本，并断开 SSH 不中断脚本：

```bash
screen -S longtask
python3 long_script.py
# 按 Ctrl+a 然后按 d 来 detach

#之后你可以放心断线，过一会再重新连接：
screen -r longtask
```

---

## 📁 五、screen 配置文件（可选）

配置文件位置：

```
~/.screenrc
```

可以自定义启动行为，比如：

```bash

startup_message off
defscrollback 10000
caption always "%{= kw}%H %{= gk}%-w%{= BW}%n %t%{-}%+w %=%{= wk}%Y-%m-%d %c"

```

### 方法二：使用 `nohup` 和 `&`

`nohup` 命令用于在用户退出 shell 后仍能运行指定的命令。`&` 符号则将命令放入后台运行。

### 步骤 1: 运行你的程序

在 SSH 会话中，在你的命令前加上 `nohup`，并在命令末尾加上 `&`：

`nohup python your_long_running_script.py &`

### 解释：

- `nohup`: 表示“no hangup”，阻止进程在终端关闭时接收到 SIGHUP 信号而终止。
- `python your_long_running_script.py`: 你要运行的实际命令。
- `&`: 将命令放入后台运行，这样你可以在同一终端中继续输入其他命令，而不需要等待当前命令完成。

### 步骤 2: 查看输出

- 当你使用 `nohup` 时，程序的所有标准输出（`stdout`）和标准错误（`stderr`）默认会重定向到当前目录下的一个名为 `nohup.out` 的文件中。你可以使用 `cat nohup.out` 或 `tail -f nohup.out` 来查看输出。
- 如果你想把输出重定向到其他文件，可以这样做：
Bash
    
    `nohup python your_long_running_script.py > my_script_output.log 2>&1 &
    # > my_script_output.log: 将标准输出重定向到 my_script_output.log
    # 2>&1: 将标准错误重定向到与标准输出相同的地方`
    

### 步骤 3: 检查进程是否在运行

你可以使用 `ps` 命令来查看你的程序是否仍在后台运行：

Bash

`ps aux | grep your_long_running_script.py`

查找带有你的脚本名的进程。

### 步骤 4: 杀死后台进程 (如果需要)

如果你需要停止这个后台进程，首先找到它的进程 ID (PID)：

Bash

`ps aux | grep your_long_running_script.py
# 找到对应进程的 PID (通常在第二列)`

然后使用 `kill` 命令杀死它：

Bash

`kill PID_number
# 例如：kill 12345`

如果 `kill` 不起作用，你可以尝试强制杀死：`kill -9 PID_number` (慎用，可能导致数据丢失)。

### `screen` vs `nohup`

- **`screen` (推荐用于交互式任务或需要多次查看输出的任务)：**
    - 提供一个可恢复的虚拟终端环境。
    - 你可以随时附着回来查看输出，甚至重新输入命令。
    - 适合需要与程序交互或调试的情况。
    - 对于复杂的任务流，可以创建多个 `screen` 窗口。
- **`nohup` (推荐用于简单的、不需要交互的批处理任务)：**
    - 更轻量级，通常用于启动一个独立运行的程序。
    - 输出通常重定向到文件。
    - 一旦启动，你无法像 `screen` 那样“重新连接”到它的会话。
