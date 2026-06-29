# 📊 SwinIR 训练监控与日志读取指南

当前状态：使用 Runner 后台训练中

实验配置：`train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8.yml`

## 一、 核心技能：如何看懂日志 (Log)

既然是在 Runner 里跑，你通常面对的是不断滚动的终端输出或文本日志。

### 1. 找到日志文件

如果你能进到挂载了目录的宿主机或容器，日志通常在：

Bash

```
# 进入根目录
cd /workspace # 或你的 BasicSR 路径

# 找到最新的 log 文件
ls -lt experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/*.log
```

### 2. 实时追踪训练状态 (每 100 iter)

使用 `tail` 命令实时查看最后几行日志：

Bash

```
tail -f experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/train*.log
```

**日志长这样：**

`[train..][epoch: 0, iter: 100, lr:(1.000e-04,)] [eta: 2:30:00, time: 0.350 (0.120)] l_pix: 1.2345e-02`

**重点盯什么：**

- **`l_pix` (Pixel Loss)**：最核心的指标！不需要每次都变小，但**总体趋势必须是下降的**。如果它变成 `NaN`，或者一直卡在一个数字不动，说明训练崩了，直接停掉。
    
- **`eta`**：预计剩余时间，帮你规划什么时候来看结果。
    
- **报错信息**：留意有没有 `OOM` (显存溢出) 或找不到图片的报错。
    

### 3. 提取验证集得分 (每 2000 iter)

训练每 2000 步会自动在验证集上算分。不用在满屏的日志里翻，直接用 `grep` 抓取：

Bash

```
grep -A2 "Validation CylinderblockPairedVal" experiments/train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8/train*.log
```

**正常会输出类似：**

> psnr: 28.1234 Best: 28.5678 @ 4000 iter
> 
> ssim: 0.8123 Best: 0.8201 @ 4000 iter

**重点盯什么：**

- 看 `psnr` 和 `ssim` 是否在稳步提升。
    
- 看 `Best` 停留在哪个 iter。
    

## 二、 进阶监控：使用 TensorBoard (看趋势)

如果你觉得看文本太累，可以通过 TensorBoard 看可视化曲线。

**启动方法**（在能访问 `tb_logger` 目录的终端运行）：

Bash

```
uv run tensorboard --logdir tb_logger --port 6006 --bind_all
```

浏览器打开 `http://localhost:6006`（若在服务器上，需做端口转发）。

**看这三条线就够了：**

- `losses/l_pix`：训练 loss（应呈下降趋势）
    
- `metrics/.../psnr`：验证集 PSNR（应呈上升趋势）
    
- `metrics/.../ssim`：验证集 SSIM（应呈上升趋势）
    

## 三、 关键节点：训练时后台都在干什么？

为了让你心里有底，这是你的 YML 配置设定的自动时间表：

|**频率**|**后台动作**|**你的关注点**|
|---|---|---|
|**每 100 iter**|终端/日志打印 `l_pix`, `lr`, `eta`|Loss 是否在下降，有无报错|
|**每 2000 iter**|在 val 集算 PSNR/SSIM，写入日志和 TB|模型指标是否提升|
|**每 5000 iter**|保存模型权重 (`.pth`) 和状态 (`.state`)|确认磁盘空间足够，权重已生成|
|**训练结束**|保存 `net_g_latest.pth` 并再跑一次 val|准备进行最终的视觉对比测试|

## 四、 极简排障与验证指南

### 1. 跑起来的前 10 分钟看什么？

- 日志有无滚动？
    
- `l_pix` 是否为正常数值（不是 NaN）？
    
- 有没有立刻报错中断？
    

### 2. 意外中断了怎么办？

如果 Runner 挂了，只要 `training_states` 目录下有 `.state` 文件，就可以断点续训：

Bash

```
uv run python basicsr/train.py -opt options/train/SwinIR/train_SwinIR_grayDN_cylinderblock.yml --auto_resume
```

### 3. 指标很高，就算成功了吗？(⚠️ 避坑警告)

**绝对不是！**

对于 X 光低剂量去噪，PSNR/SSIM 只能作参考。指标高不代表条纹/环伪影消失了，甚至可能出现“过度平滑”或“假结构”。

**终极标准：肉眼看图！** 建议挑一个 Best PSNR 的 checkpoint，和之前的 smoke test 结果做 10uA/30uA/50uA 的三联对比图（LQ / Output / GT）。

现在 Runner 跑了大概多少个 iter 了？目前的 `l_pix` 数值看起来稳定吗？需不需要我帮你分析一下某条看不懂的报错或者日志输出？