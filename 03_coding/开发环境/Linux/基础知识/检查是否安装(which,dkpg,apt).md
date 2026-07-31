---
title: "检查是否安装(which,dpkg,apt)"
publish: false
tags: ["Linux"]
---
# 检查是否安装(which,dpkg,apt)

### 使用 `which` 命令 (最常用且快捷)

`which` 命令会告诉你一个命令的可执行文件在 `$PATH` 环境变量的哪个目录下。如果找到了，就说明这个命令是可执行的，通常意味着对应的工具已经安装。

- **用法:** `which [command_name]`
- **示例:**
    - 检查 `git` 是否安装：
    Bash
        
        `which git`
        
        - 如果已安装，可能会输出：`/usr/bin/git`
        - 如果未安装，通常没有输出或输出 `git not found`。
    - 检查 `python3` 是否安装：
    Bash
        
        `which python3`
        
        - 如果已安装，可能会输出：`/usr/bin/python3`
- **优点:** 简单、快速、直接告诉你命令的位置。
- **缺点:** 只能检查在 `$PATH` 环境变量中的可执行文件。如果工具安装了但不在 `$PATH` 中（不常见），或者它是库而不是独立可执行文件，`which` 就查不到。

### 2. 使用 `whereis` 命令 (查找二进制、源代码和手册页)

`whereis` 命令比 `which` 更全面，它会查找命令的二进制文件、源代码文件和手册页文件。

- **用法:** `whereis [command_name]`
- **示例:**Bash
    
    `whereis python3
    # 可能输出: python3: /usr/bin/python3 /usr/lib/python3 /usr/local/bin/python3 /usr/local/lib/python3 /usr/share/man/man1/python3.1.gz`
    
    - 如果显示了路径，通常说明已安装。
- **优点:** 查找范围更广，能提供更多相关信息。

### 3. 使用包管理器 (适用于通过包管理器安装的软件)

这是检查软件是否通过系统包管理器安装的最可靠方法。不同的 Linux 发行版使用不同的包管理器。你的 Ubuntu Server 使用的是 `apt`。

### 对于 Ubuntu/Debian (使用 `apt`)

- **检查软件包是否安装:** `dpkg -l` 命令可以列出所有已安装的 Debian 包。你可以结合 `grep` 来过滤。
    - **用法:** `dpkg -l | grep [package_name]`
    - **示例:** 检查 `nginx` Web 服务器是否安装：
    Bash
        
        `dpkg -l | grep nginx`
        
        - 如果已安装，你会看到一行以 `ii` 开头（表示已安装且配置正确）的输出，其中包含 `nginx` 包名。
        - 如果未安装，可能没有输出或不包含 `ii` 开头的行。
- **查询特定软件包信息:** `apt list --installed` 也可以查看已安装的包。
    - **用法:** `apt list --installed | grep [package_name]`
    - **示例:**Bash
        
        `apt list --installed | grep nfs-common`
        
        - 如果已安装，会显示 `nfs-common/jammy,now 1:2.6.1-1ubuntu1 amd64 [installed]` 类似信息。
- **查询软件包详情 (即使未安装也可以查):** `apt show` 可以显示软件包的详细信息，包括是否已安装。
    - **用法:** `apt show [package_name]`
    - **示例:**Bash
        
        `apt show tightvncserver`
        
        - 在输出中查找 `State: installed` 或 `Installed: yes` 字样。
- **优点:** 最权威的检查方式，能准确判断通过 `apt` 安装的软件状态。
- **缺点:** 只能检查通过 `apt` 安装的软件。对于手动编译安装的软件或通过 Docker 等容器运行的软件则无效。

### 4. 检查服务状态 (对于作为服务的工具)

如果某个工具是作为后台服务运行的（如 Web 服务器 Nginx、数据库 MySQL），你可以检查它的服务状态。

- **用法:** `systemctl status [service_name]`
- **示例:**
    - 检查 Nginx 服务是否运行：
    Bash
        
        `systemctl status nginx`
        
        - 如果已安装并运行，会显示 `Active: active (running)`。
        - 如果未安装或未启动，会显示其他状态或错误信息。
- **优点:** 能检查服务是否正在运行，而不仅仅是是否安装。

### 5. 查找特定目录或文件

有些软件可能不通过包管理器安装，或者安装在非标准路径。你可以直接查找其可执行文件或安装目录。

- **用法:** `find / -name "[filename]" 2>/dev/null` (这会搜索整个文件系统，可能会很慢)
- **示例:** 查找特定的可执行文件 `my_custom_tool`：Bash
    
    `find /opt /usr/local -name "my_custom_tool" 2>/dev/null`
    
    - `2>/dev/null` 用于抑制权限错误信息。
- **优点:** 适用于任何安装方式。
- **缺点:** 慢，需要知道大概的文件名或目录结构。

### 总结推荐：

1. **最快判断一个命令是否存在：** 使用 `which [command_name]`。
2. **判断通过 `apt` 安装的软件包：** 使用 `dpkg -l | grep [package_name]` 或 `apt show [package_name]`。
3. **判断后台服务是否运行：** 使用 `systemctl status [service_name]`。
