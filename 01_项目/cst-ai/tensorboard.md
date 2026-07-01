## TensorBoard 在本项目里怎么工作

训练配置里开了 `use_tb_logger: true`（见 `train_SwinIR_grayDN_cylinderblock.yml`）。BasicSR 在训练时把标量（主要是 `losses/l_pix` 等）写到 TensorBoard event 文件。

本地 / 容器内（未设 CI 环境变量时）默认路径：

<repo>/tb_logger/<实验名>/

Cylinderblock 实验名：`train_SwinIR_grayDN_cylinderblock_noise15_ft_P128W8`

容器里也可以直接：

uv run tensorboard --logdir tb_logger --port 6006 --bind_all

---

## CI 训练时写到哪里

CI job 会先跑 `scripts/ci_storage.py prepare`，设置：

BASICSR_TB_LOGGER_ROOT=/home/huang/cst_ai/ci_outputs/{PipelineID}/tb_logger

对应 GPU 宿主机 NFS：

/mnt/home/huang/cst_ai/ci_outputs/{PipelineID}/tb_logger/<实验名>/

同时有 `latest` 软链 → 最近一次训练：

/mnt/home/huang/cst_ai/ci_outputs/latest/tb_logger/...

文本日志在并行路径 `experiments/<实验名>/train_*.log`。

---

## CI 能不能直接打开网页看？

GitLab 网页里没有内置 TensorBoard 面板；访问要靠宿主机或本地下载。

### 方式 1：SSH + 脚本（推荐，训练进行中也可看）

在 GPU 宿主机上：

# 看所有 pipeline 的 TB（会扫整个 ci_outputs 树）

bash scripts/serve_tensorboard.sh /mnt/home/huang/cst_ai/ci_outputs

# 或只看某次

bash scripts/serve_tensorboard.sh /mnt/home/huang/cst_ai/ci_outputs/latest

脚本会起一个 `tensorflow/tensorflow` 容器，默认 6006 端口。

在你笔记本上端口转发：

ssh -L 6006:localhost:6006 huang@<gpu-host>

浏览器打开：`http://localhost:6006`

### 方式 2：GitLab Artifacts（备份，14 天）

CI `after_script` 会把 TB 拷到：

results/cylinderblock_swinir_graydn/tb_logger/

从 GitLab job 页面下载 Artifacts 后，在本机：

tensorboard --logdir /path/to/downloaded/tb_logger --port 6006

### 方式 3：只看文本 log（不用 TB）

tail -f /mnt/home/huang/cst_ai/ci_outputs/latest/experiments/*/train_*.log

---

## 当前 CI 配置下 TB 里有什么

`train_cylinderblock_swinir_graydn` 里用了 `val=~`，训练过程中不做 val，因此 TB 里主要是 训练 loss，一般没有 val PSNR/SSIM 曲线。要看验证指标需训完后单独跑 `basicsr/test.py`。

---

## 小结

|场景|路径|怎么访问|
|---|---|---|
|本地容器训练|`tb_logger/<exp_name>/`|容器内 `tensorboard --logdir tb_logger`|
|CI 持久化|`/mnt/home/huang/cst_ai/ci_outputs/{PipelineID}/tb_logger/`|宿主机 `serve_tensorboard.sh` + SSH 转发|
|CI Artifacts|job 下载包里的 `tb_logger/`|本机 `tensorboard --logdir ...`|
|GitLab 网页|—|无直接 TB UI|

训练跑起来后，在 GPU 机上执行 `bash scripts/serve_tensorboard.sh /mnt/home/huang/cst_ai/ci_outputs/latest`，再 SSH 转发 6006 即可实时看曲线。

### 2. 意外中断了怎么办？

如果 Runner 挂了，只要 `training_states` 目录下有 `.state` 文件，就可以断点续训：

Bash

```
uv run python basicsr/train.py -opt options/train/SwinIR/train_SwinIR_grayDN_cylinderblock.yml --auto_resume
```

### 3. 指标很高，就算成功了吗？


对于 X 光低剂量去噪，PSNR/SSIM 只能作参考。指标高不代表条纹/环伪影消失了，甚至可能出现“过度平滑”或“假结构”。

**终极标准：肉眼看图！** 建议挑一个 Best PSNR 的 checkpoint，和之前的 smoke test 结果做 10uA/30uA/50uA 的三联对比图（LQ / Output / GT）。

现在 Runner 跑了大概多少个 iter 了？目前的 `l_pix` 数值看起来稳定吗？需不需要我帮你分析一下某条看不懂的报错或者日志输出？