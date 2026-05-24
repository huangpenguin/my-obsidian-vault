## 检查服务器挂载状况
由于gpu服务器使用的是autofs的形式，随用随连，只能通过下面方式查看到
```
(base) ➜  ~ mount | grep nfs
nfsd on /proc/fs/nfsd type nfsd (rw,relatime)
/etc/auto.nfs on /mnt type autofs (rw,relatime,fd=6,pgrp=3495,timeout=300,minproto=5,maxproto=5,indirect,pipe_ino=68827)
(base) ➜  ~ cat /etc/auto.nfs
data  -fstype=nfs,rw,hard,intr,timeo=300,nfsvers=3  192.168.3.14:/mnt/storage/data
home  -fstype=nfs,rw,soft,_netdev,nfsvers=4        192.168.3.14:/mnt/storage/home
```

对应的docker启动窗口应该是
```
docker run -d --gpus all \
  --name my-fast-train \
  -v $(pwd):/workspace \
  -v /mnt/data/data_folder:/data \
  my-image:latest python train.py
```

`-v /mnt/data/data_folder:/data`：当 Docker 尝试去读取宿主机的 `/mnt/data/data_folder` 时，会**瞬间触发**宿主机的 Autofs 机制，把远程的 `data1` 数据流源源不断地送进容器内的 `/data` 目录。这样，你在写深度学习代码（`train.py`）时，加载数据集的路径直接写死的相对路径或绝对路径 `/data` 即可.

