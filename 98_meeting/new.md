自己的服务器更新了（22.04作为媒介）；重新更新需要卸载掉现在的英伟达驱动免得和新系统冲突ssh会链接不上


---
(base) ➜  ~ dpkg -l | grep ubuntu-desktop
ii  ubuntu-desktop                                1.539.2                                          amd64        Ubuntu desktop system
ii  ubuntu-desktop-minimal                        1.539.2                                          amd64        Ubuntu desktop minimal system


---
Could not download the upgrades

The upgrade has aborted. Please check your Internet connection or
installation media and try again. All files downloaded so far have
been kept.

Failed to fetch
https://linux.yz.yamagata-u.ac.jp/ubuntu/pool/universe/j/jsbundle-web-interfaces/node-webidl-conversions_7.0.0~1.1.0+~cs15.1.20180823-2_all.deb
403 Forbidden [IP: 133.24.248.19 443]



Restoring original system state

Aborting
Reading package lists... Done
Building dependency tree
Reading state information... Done
=== Command detached from window (Mon Jun  1 22:37:10 2026) ===
=== Command terminated with exit status 1 (Mon Jun  1 22:37:20 2026) ==


nfs-common modified

---
1.数据的用法：
