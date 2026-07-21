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
1. **恢复训练（Resume Training / 加载了 Checkpoint）：** 如果您在重启训练时，代码配置中读取了上一次中断时保存的模型权重（Checkpoint），那么通常也会一并读取优化器的状态和当前的全局步数（Global Step）。此时，迭代次数并不是从 0 开始，而是**接着上一次断掉的地方（约 3900 step）继续往下训练**。这在深度学习中是标准且正确的断点续训行为。
    
2. **日志文件追加（Log Overlap / 没有清理旧日志）：** 如果您是想完全从头（从 0 开始）训练，但没有清空之前的 TensorBoard 日志文件夹。新的一轮训练将事件写到了同一个文件夹（`train_SwinIR_grayDN_paired_tiff_noise25_P128W8`）下。
    
    - TensorBoard 会读取该文件夹下的所有日志文件（包括旧的和新的）。
        
    - 如果发现旧日志有 0-3900 步的数据，它会将这 3900 步画出来。
        
    - 注意左侧列表中有一个带 `_archived_...` 后缀的文件夹，说明您的训练框架可能有自动归档旧日志的功能，但可能当前主文件夹里的旧 `events.out.tfevents...` 文件并未被彻底移除。
        

帮我进行每次ci启动训练的合理命名操作