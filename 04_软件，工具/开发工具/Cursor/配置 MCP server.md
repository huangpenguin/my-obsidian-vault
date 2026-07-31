---
title: "配置 MCP server"
publish: false
tags: ["待整理"]
---
# 配置 MCP server

**配置 MCP 真的就是在 Cursor 里加一条 Server 的启动命令（或地址）！**

不过，为了让你真正在操作时绝对不踩坑，三叔得给你稍微精确一下：在 Cursor 里，你加的通常不是一个像 `http://...` 这样的网址，而是一条**“本地终端启动命令”**。

这叫做 **stdio（标准输入输出）模式**，是目前 Cursor 和本地工具连通最主流的方式。

---

### 1. 实际操作：在 Cursor 里到底怎么加？

你现在就可以打开你的 Cursor 跟着看一眼，操作极其简单：

1. 打开 Cursor 的设置（`Cmd + ,`）。
2. 在左侧找到 **Features**（功能），然后往下滚动找到 **MCP**。
3. 点击 **+ Add New MCP Server**（添加新的 MCP 服务器）。
4. 这时候会弹出一个框，让你填三个东西：
    - **Name（名字）**：随便起，比如叫 `my-mysql-db`。
    - **Type（类型）**：默认选 **command**。
    - **Command（命令）**：这就是你说的“地址”了。你要在这里填入启动那个具体连库代码的终端命令。比如：`npx -y @modelcontextprotocol/server-postgres postgresql://user:pass@localhost/mydb` 或者 `python /你的绝对路径/my_mcp_server.py`。

### 2. 加完之后会发生什么“魔法”？

一旦你点击保存，Cursor 就会在后台悄悄运行你那条命令，把传达室（MCP Server）跑起来。

接着，奇迹就发生了：
你在 Cursor 的对话框（Ctrl/Cmd + L）里问它：“**帮我查一下数据库里最新的 10 条 OCR 训练记录。**”

1. Cursor 里的 AI 发现自己现在多了一个叫 `query_sql` 的工具。
2. 它会自己写一句 `SELECT * FROM train_logs ORDER BY time DESC LIMIT 10`。
3. 它通过 MCP 协议把这句 SQL 发给你的本地 Server。
4. Server 去查数据库，把结果的文本返回给 Cursor。
5. Cursor 直接在聊天框里回答你：“我查到了，最新的 10 条记录如下……”

**全过程你不需要复制粘贴任何数据，也不用切到数据库可视化软件（比如 Navicat 或 DataGrip）里去查！**
