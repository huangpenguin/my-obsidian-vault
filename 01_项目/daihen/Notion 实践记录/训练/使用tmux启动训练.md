---
title: "使用tmux启动训练"
publish: false
tags: ["OCR","项目实践"]
---
# 使用tmux启动训练

```bash
tmux new -s paddlex-train
cd /mnt/home/huang/workspace_daihen
source /home/huang/miniconda3/bin/activate paddle_env310
python /mnt/home/huang/workspace_daihen/third_party/PaddleX/main.py \
    -c /mnt/home/huang/workspace_daihen/third_party/PaddleX/paddlex/configs/modules/text_detection/PP-OCRv5_server_det.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./datasets/train_data/det \
```

```jsx
#rec
python /mnt/home/huang/workspace_daihen/third_party/PaddleX/main.py \
-c /mnt/home/huang/workspace_daihen/third_party/PaddleX/paddlex/configs/modules/text_recognition/PP-OCRv5_server_rec.yaml \
    -o Global.mode=train \

python /mnt/home/huang/workspace_daihen/third_party/PaddleX/main.py \
-c /mnt/home/huang/workspace_daihen/third_party/PaddleX/paddlex/configs/modules/text_recognition/PP-OCRv5_mobile_rec.yaml \
    -o Global.mode=train \
    
#det todo
python /mnt/home/huang/workspace_daihen/third_party/PaddleX/main.py \
-c /mnt/home/huang/workspace_daihen/third_party/PaddleX/paddlex/configs/modules/text_detection/PP-OCRv5_mobile_det.yaml \
    -o Global.mode=train \
 
```

训练过程中，如需回到普通终端，按下 `Ctrl`+`b`，松开后再按 `d` 即可“挂起”会话。稍后重新登录后，用下面命令恢复：

```jsx
tmux attach -t paddlex-train
```

若开启多个会话，可用 `tmux ls` 查看列表。

nohup也可以

```jsx
nohup python main.py -c paddlex/configs/modules/text_detection/PP-OCRv5_server_det.yaml \
    -o Global.mode=train \
    -o Global.dataset_dir=./datasets/train_data/det \
    > logs/paddlex_det_$(date +%Y%m%d_%H%M).log 2>&1 &
    
tail -f logs/paddlex_det_20251006_1530.log
```

• 查看进程：`ps -ef | grep main.py`；停止进程：`kill <PID>`。
• 若使用 GPU 训练，断线前可通过 `watch -n 5 nvidia-smi` 在另一个会话监控显存占用。

•tmux使用复制模式上下移动；而watch不好用所以用下面的代码代替

```jsx
while true; do clear; nvidia-smi; sleep 5; done
```
