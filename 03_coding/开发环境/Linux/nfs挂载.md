---
title: "nfs挂载"
publish: false
tags: ["Linux"]
---
# nfs挂载

### 前提条件：

- **NFS 服务器 (192.168.3.14) 正在运行且正常共享：** 这一步非常重要。如果 NFS 服务器本身没有正确配置共享，客户端无论怎么挂载都不会成功。我假设 NFS 服务器已经配置了共享目录，例如将 `/export/data` 共享为 `192.168.3.14:/export/data`。
- **客户端 (你的 Ubuntu Server，例如 192.168.3.11 或 192.168.3.18) 可以访问 NFS 服务器：** 确保它们之间网络畅通。

### 步骤 1：在客户端安装 NFS 客户端工具

你的 Ubuntu Server (例如黄さんマシン和 GPU 服务器) 需要安装 NFS 客户端工具才能进行挂载。

1. **通过 SSH 连接到你的 Ubuntu Server (例如黄さんマシン `192.168.3.11`)。**
2. **安装 NFS 客户端包：**
    
    `sudo apt update
    sudo apt install nfs-common -y`
    
    **检查 `nfs-common` 包中包含的常用命令是否可执行：**
    你可以检查 `nfs-common` 包中最常用的几个命令，例如 `showmount` 或 `nfsstat`。
    
    `which showmount
    which nfsstat`
    
    - **预期输出：** 如果 `nfs-common` 已经安装并正常工作，你应该能看到这些命令的路径，例如：
    `/usr/bin/showmount/usr/bin/nfsstat`
    - **这反过来证明 `nfs-common` 包已经安装了。**

### 步骤 2：测试手动挂载 NFS 共享

在尝试永久配置之前，我们先手动测试一下 NFS 挂载是否能成功。

1. **创建本地挂载点：**
如果 `/mnt/data` 和 `/mnt/home` 目录不存在，请先创建它们。即使它们存在，也要确保它们是空目录（如果里面有文件，挂载后会被覆盖）。
    
    `sudo mkdir -p /mnt/data
    sudo mkdir -p /mnt/home`
    
2. **尝试手动挂载 NFS 共享：**
你需要知道 NFS 服务器具体共享了哪些目录。假设 NFS 服务器共享了 `/export/data` 和 `/export/home`。Bash
    
    `# 挂载共享数据
    sudo mount 192.168.3.14:/export/data /mnt/data
    
    # 挂载个人数据 (如果需要，根据你的实际共享路径来)
    sudo mount 192.168.3.14:/export/home /mnt/home`
    
    - **重要：** 请将 `192.168.3.14:/export/data` 和 `192.168.3.14:/export/home` 替换为你的 NFS 服务器**实际共享的路径**。如果你不确定，需要咨询你的上司或检查 NFS 服务器的 `/etc/exports` 文件。
    - `192.168.3.14` 是 NFS 服务器的 IP 地址。
    - `/export/data` 是 NFS 服务器上被共享的目录。
    - `/mnt/data` 是你的客户端 Ubuntu Server 上的挂载点（本地目录）。
3. **检查挂载结果：**
    
    `df -h`
    
    现在你应该能看到类似这样的输出：
    
    `Filesystem              Size  Used Avail Use% Mounted on
    ...
    192.168.3.14:/export/data  XG  YG  ZG  Z% /mnt/data
    192.168.3.14:/export/home  XG  YG  ZG  Z% /mnt/home
    ...`
    
    192.168.3.14:/mnt/storage/data /datadrive                nfs     hard,intr,rw    0       0
    192.168.3.14:/mnt/storage/home /mnt/home                 nfs     hard,intr,rw    0       0
    
    同时，你也可以进入目录查看内容：
    
    `ls -l /mnt/data
    ls -l /mnt/home`
    
    如果你能看到文件，恭喜你，手动挂载成功了！这意味着 NFS 服务器是正常工作的，问题在于自动挂载。
    

### 步骤 3：配置 NFS 自动挂载 (`/etc/fstab`)

手动挂载只在当前会话有效，服务器重启后就会失效。为了让服务器每次启动时自动挂载 NFS 共享，你需要编辑 `/etc/fstab` 文件。

1. **备份 `/etc/fstab`：**
在修改系统关键文件之前，总是建议先备份。Bash
    
    `sudo cp /etc/fstab /etc/fstab.bak`
    
2. **编辑 `/etc/fstab` 文件：**Bash
    
    `sudo nano /etc/fstab`
    
3. **添加 NFS 共享条目：**
在文件的末尾添加以下两行（如果需要）：
    
    `# NFS shares from 192.168.3.14
    192.168.3.14:/export/data /mnt/data nfs defaults,nofail 0 0
    192.168.3.14:/export/home /mnt/home nfs defaults,nofail 0 0`
    
    - **解释：**
        - `192.168.3.14:/export/data`：NFS 服务器的 IP 地址和共享路径。**请务必替换为实际路径！**
        - `/mnt/data`：客户端的本地挂载点。
        - `nfs`：文件系统类型，表示这是 NFS 文件系统。
        - `defaults`：一组默认的挂载选项，包括 `rw` (读写), `suid` (设置用户 ID), `dev` (解析字符或块设备), `exec` (允许执行), `async` (异步写入), `auto` (系统启动时自动挂载), `nouser` (只有 root 用户可以挂载), `_netdev` (要求网络可用才能挂载)。
        - `nofail`：**非常重要！** 这个选项意味着如果挂载失败（例如 NFS 服务器不可达），系统**不会**因此而阻止启动。否则，如果 NFS 挂载失败，你的服务器可能会无法正常启动。
        - `0`：`dump` 选项，通常用于文件系统备份。`0` 表示不进行备份。
        - `0`：`pass` 选项，用于文件系统检查。`0` 表示启动时不检查。
4. **保存并退出 `nano`：** 按 `Ctrl + O`，回车确认，然后按 `Ctrl + X`。

### 步骤 4：测试 `/etc/fstab` 配置

在不重启服务器的情况下，你可以测试 `/etc/fstab` 中的新配置是否正确。

1. **卸载之前手动挂载的 NFS 共享：**
（如果提示忙碌，可能是其他进程正在使用这些目录。你可以使用 `lsof | grep /mnt/data` 查找占用进程并终止它们，或者强制卸载 `sudo umount -l /mnt/data`）。
    
    `sudo umount /mnt/data
    sudo umount /mnt/home`
    
2. **尝试重新挂载所有 `fstab` 中定义的项：**
    
    `sudo mount -a`
    
    - `mount -a` 命令会尝试挂载 `/etc/fstab` 中所有未挂载的、并且配置为自动挂载的文件系统。
3. **再次检查挂载状态：**
如果一切正常，你应该再次看到 NFS 共享已经成功挂载到 `/mnt/data` 和 `/mnt/home` 了。
    
    `df -h`
    

### 步骤 5：重启服务器进行最终测试 (可选但推荐)

如果你完成了前四步并且 `mount -a` 成功，那么理论上重启后 NFS 共享也应该自动挂载。

`sudo reboot`

服务器重启后，再次通过 SSH 连接，并运行 `df -h` 和 `ls -l /mnt/data` / `ls -l /mnt/home` 来确认 NFS 挂载是否在启动时自动完成。

---

**如果仍然有问题：**

- **检查 NFS 服务器配置：** 联系你的上司或检查 NFS 服务器上的 `/etc/exports` 文件，确认 `/export/data` 和 `/export/home`（或者其他实际路径）是否正确地共享给了你的 Ubuntu Server 的 IP 地址（例如 `192.168.3.11`）。例如，`/etc/exports` 中可能有这样的行：
    
    `/export/data 192.168.3.11(rw,sync,no_subtree_check)
    /export/home 192.168.3.11(rw,sync,no_subtree_check)`
    
- **检查防火墙：** 确保 NFS 服务器的防火墙允许你的客户端访问 NFS 端口（2049/tcp, 111/tcp/udp）。客户端的防火墙也需要允许出站连接。
- **查看系统日志：** 如果挂载失败，`dmesg` 或 `journalctl -xe` 可能会有提示。
Bash
    
    `dmesg | grep NFS
    journalctl -u remote-fs.target # 查看远程文件系统相关日志`
    

一步步仔细操作，你应该就能解决 NFS 自动挂载的问题了！
