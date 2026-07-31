---
title: "yaml.ip(distributed)"
publish: false
tags: ["OCR","项目实践"]
---
# yaml.ip(distributed)

- **Rec训练正确地使用**: `PADDLE_MASTER=127.0.0.1:6070` (本地回环地址)
- **Det训练错误地使用**: `PADDLE_MASTER=192.168.11.11:41388` (无法连接的IP)，会冻结住

```yaml
#改一下paddlex的config即可，但是其实paddleocr的启动命令可以指定ips
Train:
  epochs_iters: 200
  batch_size: 16
  learning_rate: 0.001
  pretrain_weight_path: https://paddle-model-ecology.bj.bcebos.com/paddlex/official_pretrained_model/PP-OCRv5_server_det_pretrained.pdparams
  resume_path: null
  #resume_path: ./output/det/latest/latest.pdparams
  log_interval: 10
  eval_interval: 3
  save_interval: 10
  dist_ips:
    - 127.0.0.1
  loader:
    num_workers: 8
```

千万不要直接使用环境变量，这会导致tmux的一直是设置的环境变量，而在yaml里面怎么改都无法覆盖！！！！
