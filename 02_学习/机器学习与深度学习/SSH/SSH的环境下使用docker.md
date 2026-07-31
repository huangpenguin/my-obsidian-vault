---
title: "SSH的环境下使用docker"
publish: false
tags: ["机器学习"]
---
# SSH的环境下使用docker

进入ssh后,选择合适的work directory!!!

创建一个docker，退出后删除

```python
sudo docker run -it --net host --rm --mount type=bind,src=./,dst=/work python3.12 /bin/bash
```

然后在docker列表里面选择对应的docker，

attach docker to VSCode

然后选择合适的python版本
