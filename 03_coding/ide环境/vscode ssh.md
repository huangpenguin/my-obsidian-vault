远程跑linux其实最重要的是权限；

比如user没有特定权限的话，执行的paddlepaddle可能无法使用gpu，而nvidia-smi也无法直接使用。

此外，如果设置的动态是自动挂载的话，前一次从vscode退出，后一次可能就无法直接打开vscode进到上次的mnt里的文件夹了，因为尚未加载，需要另起一个cmd先访问mnt进行自动加载可以(已通过把ls /mnt/data > /dev/null 2>&1追加到.bashrc中解决)

## 设置步骤：

1. **在本地 Windows 安装 VS Code**
    
2. **安装 Remote-SSH 扩展**
    
3. **连接到服务器**
    
    Ctrl+Shift+P → Remote-SSH: Connect to Host
    
    输入：user@your-server-ip
    
4. **直接在 VS Code 中打开服务器目录**
    
    /datadrive/workspace/2021.aisin-tray-recognition
    

---

## 配套进行几个服务器的配置

```markdown
 username: huang
 初期pw: huang
# passwd コマンドでパスワードを変更してください
・黄さん専用マシン
   192.168.3.11
・GPUサーバ
   192.168.3.18
・nfs サーバ
   192.168.3.14

-192.168.3.24 
-user: jil
-passwd: ilovekyoto

192.168.3.24:/datadrive 
# 黄さんマシン、GPUサーバからアクセス可能
   /mnt/data     # 共有データ
   /mnt/home   # 個人データ
```

### SSH 密钥和配置 SSH 连接

```markdown
ssh-keygen -t rsa -b 4096 -C "MPF"

#linux的复制指令、如果不是请手动复制，每个服务器都需要手动复制一遍
ssh-copy-id huang@192.168.3.11
ssh-copy-id huang@192.168.3.18
ssh-copy-id huang@192.168.3.14

ssh-copy-id -i ~/.ssh/id_rsa_remote_pc.pub huang@192.168.3.11

```

按 `Ctrl+Shift+P`，输入 `Remote-SSH: Open SSH Configuration File`，编辑配置：

```markdown
# Workstation (公司开发机)
# 可用 ssh work 快速登录
Host work1
    HostName 192.168.3.11
    User huang
    Port 22
    IdentityFile ~/.ssh/id_rsa 
    LocalForward 6006 localhost:6006  # TensorBoard 端口转发

# GPU Server (训练服务器)
Host gpu01
    HostName 192.168.3.18
    User huang
    Port 22
    IdentityFile ~/.ssh/id_rsa

# Data Server (数据中心) 
Host data
    HostName 192.168.3.14
    User huang
    Port 22
    IdentityFile ~/.ssh/id_rsa

# 额外的服务器
Host work2
    HostName 192.168.3.24
    User jil
    Port 22
    IdentityFile ~/.ssh/id_rsa
    #ServerAliveInterval 60            # 保持连接活跃
    #ServerAliveCountMax 3
    #ForwardX11 yes                    # 如果需要图形界面（如标注工具），暂时有问题
```

### 文件共享

```markdown
#rsync 走的是 SSH 协议
#如果已经配置好了 SSH 密钥并上传了公钥到server,那这两条命令就不需要输入密码。
sudo mkdir -p /datadrive/workspace/2021.aisin-tray-recognition
cd /datadrive/workspace/2021.aisin-tray-recognition

sudo rsync -avz --progress --partial --inplace jil@192.168.3.24:/datadrive/workspace/2021.aisin-tray-recognition/ /datadrive/workspace/2021.aisin-tray-recognition/ 

#如果你不确定是否覆盖、或者只想看要复制什么，可以先加 --dry-run 看一眼：
rsync -avz --dry-run jil@192.168.3.24:/datadrive/... /datadrive/...

#查看报错信息的传输方式
rsync -avh --progress jil@192.168.3.24:/datadrive/workspace/2021.aisin-tray-recognition/ /datadrive/workspace/2021.aisin-tray-recognition/  2>rsync_errors.log
less rsync_errors.log

#如果有nfs服务器，可以直接mount
#远程服务器 192.168.3.24 上的目录将会在本地aisin_tp_tray 下可见
sudo mount -t nfs -o resvport,rw 192.168.3.24:/mnt/storage/data/aisin_tp_tray data/aisin_tp_tray
ssh-keygen -t ed25519 -C "huang.pengbin@jornan.com"
```

## nfs相关

```markdown
# autofs 服务，它是一个 动态自动挂载器
# 它根据 /etc/auto.* 系列文件（比如 /etc/auto.nfs）进行按需挂载
# 只要访问 /mnt/子目录，就会触发 autofs 去 NFS 挂载

systemctl status autofs

(base) huang@jil-08-01:~$ cat /etc/auto.nfs
data  -fstype=nfs,rw,hard,intr,timeo=300,nfsvers=3  192.168.3.14:/mnt/storage/data
home  -fstype=nfs,rw,soft,_netdev,nfsvers=4        192.168.3.14:/mnt/storage/home

# 自动挂载的文件 (系统开机时静态挂载)
sudo nano /etc/fstab
#确定没有mount到nfs之类的地方
mount | grep /datadrive
```

```markdown
# 你系统的自动挂载流程可能是这样的
#/etc/auto.master:
/mnt    /etc/auto.nfs

#/etc/auto.nfs:
data -fstype=nfs,rw 192.168.3.14:/mnt/storage/data
home -fstype=nfs,rw 192.168.3.14:/mnt/storage/home

#当你访问：(一定要访问才行，也就是vscode第一次打开时会找不到)
ls /mnt/data
#系统就临时挂载：
192.168.3.14:/mnt/storage/data → /mnt/data
```



