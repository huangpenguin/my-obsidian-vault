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
        


# 数据集（固定）
/mnt/data/cst_ai/datasets/aligned_drive/{LQ,GT}
/mnt/home/huang/cst_ai/datasets/aligned_drive/{train.txt,val.txt,...}

# 实验权重（跨 pipeline 续训看这里）
/mnt/home/huang/cst_ai/experiments/aligned_drive/
  └── train_SwinIR_grayDN_paired_tiff_noise25_P128W8/   ← 实验名
        ├── models/net_g_5000.pth
        └── training_states/5000.state

# 某次 CI 流水线的附属输出（会变）
/mnt/home/huang/cst_ai/ci_outputs/2695540961/            ← Pipeline ID
  ├── tb_logger/...
  └── experiments/   （旧逻辑会用；现在正式训练权重不靠这个续）
  
  
  
  
  
  ### 方向一：优化观测，让图表更细致平滑（强烈推荐优先尝试）

如我们前面分析的，你目前的损失图锯齿大，且验证集指标只有几个点。仔细看配置发现，你的 `total_iter` 是 15000，但 `val_freq` 竟然是 `5e3` (5000)。这意味着整个训练过程**只验证了 3 次**！

- **修改参数：**
    
    1. `val_freq: 1000` 或 `2000`（增加验证频率，让你能看到平滑上升的评估曲线）。
        
    2. `save_checkpoint_freq: 1000` 或 `2000`（配合验证频率同步存权重）。
        
    3. `batch_size_per_gpu: 8`（如果你的显卡显存没满，增加这个值可以大幅平滑左下角的像素损失曲线，并略微提速）。
       
     4. `total_iter: 30000` (翻倍)
        
    5. `train.scheduler.milestones: [15000, 25000]` (配合总步数往后推移)
        
        
        

---

### 方向二：改变损失函数，追求更清晰的纹理

目前你使用的是 `MSELoss` (L2 损失)。MSE 在数学上容易让模型给出一种“保守的平均预测”，这通常会导致图像的 PSNR 很高，但**视觉上可能稍微偏模糊或平滑**。对于图像恢复，`L1Loss` 或 `CharbonnierLoss`（平滑的 L1）往往能保留更好的边缘和高频纹理。

- **修改参数：**
    
    YAML
    
    ```
    pixel_opt:
      type: L1Loss    # 将 MSELoss 改为 L1Loss，或者尝试 CharbonnierLoss
      loss_weight: 1.0
      reduction: mean
    ```
    
    _(建议同时配合方向一，把 val_freq 降下来以便观察)_
    
- **推荐的实验 Name：** `train_SwinIR_grayDN_paired_tiff_noise25_P128W8_L1Loss`
    

---

### 方向三：扩大感受野，让模型看更全（吃显存）

对于去噪任务，有时候更大的图像块（Patch）能提供更丰富的上下文结构信息，帮助模型更好地区分“噪声”和“真实纹理”。目前你的 `gt_size` 是 128。

- **修改参数：**
    
    1. `datasets.train.gt_size: 192` 或 `256`（必须是 `window_size: 8` 的整数倍）。
        
    2. `network_g.img_size: 192` 或 `256`（**注意：这俩必须保持一致**）。
        
    3. 如果显存溢出，需要把 `batch_size_per_gpu` 降回 2 或 4。
        
- **推荐的实验 Name：** `train_SwinIR_grayDN_paired_tiff_noise25_P192W8` 或 `..._P256W8` _(注：P192 代表 Patch 变成了 192。)_
    

---

### 方向四：延长训练步数

15000 步（`total_iter`）对于基于 Transformer 的 SwinIR 来说其实是一个比较短的微调过程。如果上一次训练到最后，你发现验证集指标（SSIM/PSNR）**依然在明显上升，完全没有变平缓的趋势**，你可以让它多飞一会儿。

- **修改参数：**
    

        
- **推荐的实验 Name：** `train_SwinIR_grayDN_paired_tiff_noise25_P128W8_30k` _(注：30k 直观表明了训练的总迭代数。)_
    

---

**💡 总结建议：** 如果你想立刻开跑下一个实验，最稳妥的做法是：**保持现有的网络结构不变，只修改观测频率和损失函数（综合方向一和方向二）**。

你可以直接使用这个名字和改动： **`name: train_SwinIR_grayDN_paired_tiff_noise25_P128W8_L1Loss_Val1k`** 改动：将 `pixel_opt.type` 换成 `L1Loss`，并将 `val_freq` 和 `save_checkpoint_freq` 改为 `1000`。然后去 WandB 看看这次的曲线是不是漂亮多了！