### 训练结果会写在哪（GitLab CI）

宿主机（SSH）：

/mnt/home/huang/cst_ai/

├── datasets/aligned_drive/ # 元数据（train/val/normalization）只读用

├── archive/ # 旧实验归档（非本次训练）

└── ci_outputs/

├── <PIPELINE_ID>/ # 正式训练

│ ├── experiments/

│ │ └── <exp_name>/ # 如 train_SwinIR_grayDN_paired_tiff_...

│ │ ├── models/ # net_g_*.pth

│ │ ├── training_states/

│ │ └── *.log

│ └── tb_logger/<exp_name>/

├── <PIPELINE_ID>_debug/ # debug job

└── latest -> <PIPELINE_ID>/ # 正式训练结束后的相对软链

容器内对应：`/home/huang/cst_ai/ci_outputs/...`  
图像源只读：`/mnt/data/...` ↔ `/data/cst_ai/datasets/aligned_drive/{LQ,GT}`

GitLab 上 push 到 `main` 后再手动跑 `train_paired_tiff_swinir_debug` → `train_paired_tiff_swinir`。需要我帮 push 的话直接说。


| 组   | Z       | 保留对数             |
| --- | ------- | ---------------- |
| 5-1 | 275–623 | 1396             |
| 5-2 | 185–740 | 2224             |
| 5-3 | 185–745 | 2244             |
| 5-4 | 185–745 | 2244             |
| 合计  |         | 8108（丢掉空白片 6228） |