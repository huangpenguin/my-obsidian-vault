## 一、训练过程中会自动发生什么

你的 yml 里已经配置了：

logger:

print_freq: 100 # 每 100 iter 打印一次 loss

save_checkpoint_freq: 5000 # 每 5000 iter 存 checkpoint

use_tb_logger: true

val:

val_freq: 2000 # 每 2000 iter 跑一次 validation

save_img: false # 默认不保存 val 可视化图

metrics:

psnr: ...

ssim: ...

所以训练时：

|频率|发生什么|
|---|---|
|每 100 iter|终端/日志打印 `l_pix`、lr、eta|
|每 2000 iter|在 val 集上算 PSNR/SSIM，写入日志和 TensorBoard|
|每 5000 iter|保存模型权重和 training state|
|训练结束|保存 `net_g_latest.pth`，并再跑一次 val|

---

## 二、目录结构：去哪里看

实验名来自 yml 的 `name`：

train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8

默认路径（本地训练）：

BasicSR/

├── experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/

│ ├── train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8_2026-xx-xx-xx-xx-xx.log # 文本日志

│ ├── train_SwinIR_grayDN_cylinderblock.yml # 训练时复制的 yml

│ ├── models/

│ │ ├── net_g_5000.pth

│ │ ├── net_g_10000.pth

│ │ └── net_g_latest.pth # 含 params 和 params_ema（因为开了 ema_decay）

│ ├── training_states/

│ │ ├── 5000.state # 断点续训用

│ │ └── latest.state

│ └── visualization/ # 只有 save_img: true 时才有

│

└── tb_logger/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/

└── events.out.tfevents... # TensorBoard 事件文件

如果用 GitLab CI 训练，输出可能在：

/home/<user>/cst_ai/ci_outputs/<pipeline_id>/experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/

---

## 三、具体怎么做：看日志

### 1. 实时看终端输出

训练命令跑着时，每 100 iter 会看到类似：

[train..][epoch: 0, iter: 100, lr:(1.000e-04,)] [eta: 2:30:00, time: 0.350 (0.120)] l_pix: 1.2345e-02

关注：

- `l_pix` 是否总体下降（不必单调）
- 是否 OOM、NaN、dataset 读图报错

### 2. 看文本日志文件

# 进入项目根目录

cd /workspace # 或你的 BasicSR 路径

# 找到最新 log

ls -lt experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/*.log

# 实时跟踪

tail -f experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/train_*.log

### 3. 从日志里 grep validation 结果

每 2000 iter 会有一条类似：

Validation CylinderblockPairedVal

# psnr: 28.1234 Best: 28.5678 @ 4000 iter

# ssim: 0.8123 Best: 0.8201 @ 4000 iter

grep -A2 "Validation CylinderblockPairedVal" \

experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/train_*.log

---

## 四、具体怎么做：看 TensorBoard

在另一个终端（容器内或宿主机，路径能访问到 `tb_logger` 即可）：

cd /workspace # BasicSR 根目录

uv run tensorboard --logdir tb_logger --port 6006 --bind_all

浏览器打开：

http://localhost:6006

或在 Cursor/VS Code 里做端口转发后访问。

TensorBoard 里重点看：

|曲线|含义|
|---|---|
|`losses/l_pix`|训练 pixel loss|
|`metrics/CylinderblockPairedVal/psnr`|val PSNR|
|`metrics/CylinderblockPairedVal/ssim`|val SSIM|

---

## 五、具体怎么做：看 checkpoint

ls -lh experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/models/

文件说明：

- `net_g_5000.pth`、`net_g_10000.pth` …：对应 iter 的 checkpoint
- `net_g_latest.pth`：最新权重
- 因为 yml 里 `ema_decay: 0.999`，每个 `.pth` 里通常有：
    - `params`：当前训练权重
    - `params_ema`：EMA 权重（推理时优先用这个）

选哪个 checkpoint：

- 默认先用 PSNR/SSIM Best 对应的 iter 对应的 `net_g_<iter>.pth`
- 若指标和视觉效果不一致，再对比几个 checkpoint 做肉眼看图

---

## 六、具体怎么做：PSNR/SSIM 验证

### 训练期间（已自动）

无需额外操作；每 2000 iter 自动在 `val/LQ` vs `val/GT` 上计算。

注意：

- 这是 paired supervised metric，LQ 是低剂量，GT 是 100uA
- PSNR/SSIM 高 ≠ 条纹/环伪影一定消失
- 对 X 光低剂量 denoising，只能作参考

### 训练结束后，单独再跑一遍 test（推荐）

目前还没有专门的 cylinder test yml，你可以仿照 `options/test/ESRGAN/test_RRDBNet_PSNR_x4.yml` 写一个 test 配置，核心是：

name: test_SwinIR_grayDN_cylinderblock

model_type: SwinIRModel

scale: 1

num_gpu: 1

datasets:

test_1:

name: CylinderblockPairedVal

type: PairedTiffImageDataset

dataroot_gt: .../val/GT

dataroot_lq: .../val/LQ

normalization: .../normalization.json

io_backend:

type: disk

network_g:

type: SwinIR

upscale: 1

in_chans: 1

img_size: 128

window_size: 8

upsampler: ''

path:

pretrain_network_g: experiments/.../models/net_g_20000.pth

param_key_g: params_ema

val:

save_img: true # 保存 output 图，便于肉眼看

suffix: finetuned

metrics:

psnr:

type: calculate_psnr

crop_border: 0

test_y_channel: false

ssim:

type: calculate_ssim

crop_border: 0

test_y_channel: false

然后运行：

uv run python basicsr/test.py -opt options/test/SwinIR/test_SwinIR_grayDN_cylinderblock.yml

结果在：

results/test_SwinIR_grayDN_cylinderblock/visualization/CylinderblockPairedVal/

---

## 七、具体怎么做：视觉对比（更重要）

定量指标不够时，建议做 LQ / Output / GT 三联对比。

### 方法 A：训练时打开 save_img

把训练 yml 里改成：

val:

val_freq: 2000

save_img: true # 改成 true

val 图会存到：

experiments/.../visualization/<image_name>/<image_name>_2000.png

### 方法 B：用现有 smoke test 脚本做推理对比（更贴近你现在的流程）

fine-tune 后，把 checkpoint 路径指到你的 `net_g_xxxx.pth`，对 cylinder val 或 test 图跑推理，和之前的 pretrained smoke 结果对比：

# 思路：用你现有的 swinir 推理脚本，但 model_path 换成 fine-tuned 权重

# 对比目录例如：

# results/swinir_gray_dn_cylinder_smoke/noise15/ # pretrained

# results/swinir_gray_dn_cylinder_finetuned/ # fine-tuned

重点挑这几类图：

- `10uA` / `30uA` / `50uA` / `90uA` 各一张
- 有明显 streak/ring artifact 的 slice
- val 集里 PSNR 提升最大 vs 最小的样本（看是否 over-smooth）

### 方法 C：手动对比 checklist

每张图看：

1. 粒状噪声是否下降
2. streak / ring 是否还在
3. 边缘结构是否被抹平（over-smoothing）
4. 是否出现 fake 结构（hallucination）
5. 和 GT 比，细节是“变清楚”还是“变假”

---

## 八、推荐的最小验证流程

训练开始后：

1. 前 10 分钟：看 log 是否正常、`l_pix` 是否有限、无报错
2. 到 2000 iter：看第一次 val 的 PSNR/SSIM
3. 到 5000 iter：确认 checkpoint 已生成
4. TensorBoard：看 loss 和 metrics 曲线是否稳定
5. 挑 1 个 checkpoint（如 Best PSNR 的 iter）：对 val 集跑 `save_img: true` 的 test
6. 和 pretrained smoke 结果并排看图，决定是否继续训或调 lr/iter

---

## 九、断点续训（监控过程中常用）

如果训练中断，用同一 yml 加 `--auto_resume`：

uv run python basicsr/train.py \

-opt options/train/SwinIR/train_SwinIR_grayDN_cylinderblock.yml \

--auto_resume

会从 `training_states/` 里最新的 `.state` 恢复。