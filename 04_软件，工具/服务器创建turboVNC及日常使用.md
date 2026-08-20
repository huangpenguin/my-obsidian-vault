请先在服务器上的 SSH 终端执行，设置你自己的 VNC 密码：

```
/opt/TurboVNC/bin/vncpasswd
```
因为只能设置8位所以我一般用19970530


输入两次新密码；不要把密码发给我。

然后创建自己的 `:3` 会话：

```
/opt/TurboVNC/bin/vncserver -kill :3 2>/dev/null || true
rm -f ~/.vnc/gpu01:3.pid
/opt/TurboVNC/bin/vncserver :3 \
  -geometry 1600x1000 \
  -name "huang-VNC"
```





---
## 使用 

Mac 上建立隧道：

```
ssh -N -L 15903:127.0.0.1:5903 huang@192.168.3.18
```

另开一个 Mac 终端连接：

```
open vnc://127.0.0.1:15903
```

TigerVNC 地址使用：

```
127.0.0.1::15903
```

这个新会话使用：

```
显示编号：:3
服务器端口：5903
Mac 本地端口：15903
```