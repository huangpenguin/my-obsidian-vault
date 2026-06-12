## 1. 本地 GPU 的 GitLab 状态：查了吗？

没有真正查过 GitLab Runner 在线状态。

之前在本机 GPU 服务器上验证的是 Docker 运行时，不是 GitLab CI 链路：

|已验证|未验证|
|---|---|
|`docker build`|GitLab Runner 是否安装|
|`print_env.py` / `smoke_gpu.py`|Runner 是否已注册到 `jil_atr/lab-gpu-runbook`|
|`train_synthetic.py`|Runner 是否 online（绿点）|
|4×V100 DDP（含 `--shm-size 16g`）|tag 是否与 CI 一致|
|代码已 push 到 GitLab|`config.toml` 里 `concurrent=1` 是否生效|
||在 GitLab 上手动跑过 pipeline 没有|

所以：GPU 硬件/Docker 栈是通的，但 CI 端到端还没验收。

### 建议你（或维护者）现在做的检查

GitLab 网页（项目 → Settings → CI/CD → Runners）：

- 是否有 runner 显示 online
- tags 是否包含 `lab-gpu`（以及 `gpu01` / `gpu02` 等）
- executor 是否为 shell（不是 docker）

每台 GPU 服务器上：

gitlab-runner verify

gitlab-runner list

sudo cat /etc/gitlab-runner/config.toml # 或 ~/.gitlab-runner/config.toml

nvidia-smi

docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi

端到端验收（项目 → Build → Pipelines → Run pipeline）：

1. 先点 `gpu_smoke_auto` 的 ▶
2. 成功后再试 `single_gpu_train`

---

## 2. 如何更好地管理 tag，并注册第二台服务器

当前设计在 `docs/01-prerequisites.md` 和 `docs/04-multi-server-queue.md` 里，核心规则如下。

### Tag 分层（推荐固定这套）

|层级|Tag|用途|
|---|---|---|
|池级|`lab-gpu`|所有 lab GPU runner 都打；job 只写这个 tag 时 GitLab 选任意空闲 runner|
|机器级|`gpu01` / `gpu02`|绑定到具体服务器|
|能力级（可选）|`v100`, `cu124`|文档/筛选用，smoke job 已用|

规则：job 的 tags 是 AND 关系——runner 必须同时拥有 job 列出的全部 tag 才会接活。

例如 `gpu_smoke_gpu01` 需要 runner 同时有：`lab-gpu` + `gpu01` + `v100` + `cu124`。

### 每台服务器的 Runner 配置

concurrent = 1 # 一台机器同时只跑 1 个 job → 自然排队

`concurrent = 1` 是服务器级队列；比单靠 GitLab 默认并发更可控。

### 注册第二台服务器（gpu02）步骤

在 gpu02 上执行（需项目 Settings → CI/CD → Runners 里的 registration token）：

# 1. 安装 gitlab-runner（按 GitLab 官方文档，选 shell executor）

sudo gitlab-runner register \

--url https://gitlab.com/ \

--token <PROJECT_REGISTRATION_TOKEN> \

--executor shell \

--description "gpu02-v100-cu124" \

--tag-list "lab-gpu,gpu02,v100,cu124" \

--run-untagged=false

# 2. 编辑 config.toml，确保 concurrent = 1

sudo gitlab-runner verify

sudo gitlab-runner restart

注意：

- 用 shell executor，不要用 docker executor（`.gitlab/ci/gpu.yml` 会直接调宿主机 `docker` + `nvidia-smi`）
- runner 进程用户要有：`docker` 权限、`nvidia-smi`、读写 `GPU_RUN_ROOT`（默认 `$HOME/lab-gpu-runs`）
- 两台机器 tag 不要混：gpu01 只打 `gpu01`，gpu02 只打 `gpu02`，但都打 `lab-gpu`

### 管理上的改进建议

1. 维护一张 tag 表（可写在 `docs/01-prerequisites.md`）：机器名、IP、tags、`concurrent`、维护者
2. Runner description 写清楚：如 `gpu02-v100-4x32G`，日志里 `CI_RUNNER_DESCRIPTION` 也能看到
3. 训练 job 补机器级 variant：目前只有 smoke 有 `gpu_smoke_gpu01/02`，训练 job 只有 `single_gpu_train` / `multi_gpu_ddp`（tag 仅 `lab-gpu`），无法指定「必须在 gpu02 上训练」。若需要，应加 `single_gpu_train_gpu02` 等 job（复制 smoke 的 tag 模式）
4. Group Runner vs Project Runner：多台机器、多个项目共用 GPU 时，可考虑在 `jil_atr` group 注册 group runner，tag 规范统一

---

## 3. 如何对某一个训练任务加入队列

这个 runbook 没有自建调度器，队列完全靠 GitLab Runner + manual job + resource_group。

### 当前 CI 里的训练 job

gpu.ymlLines 115-131

single_gpu_train:

extends: .manual_gpu_job

stage: train

resource_group: lab-gpu-single-train

tags:

- lab-gpu

variables:

LAB_GPU_COMMAND: "python scripts/train_synthetic.py --steps ${TRAIN_STEPS}"

multi_gpu_ddp:

extends: .manual_gpu_job

stage: train

resource_group: lab-gpu-ddp

tags:

- lab-gpu

variables:

LAB_GPU_COMMAND: "torchrun --standalone --nproc_per_node=${GPU_COUNT} scripts/train_ddp.py --steps ${TRAIN_STEPS}"

### 实际操作：提交一个训练任务

1. GitLab → Build → Pipelines → Run pipeline（选 `main`）
2. 可选设置变量，例如：
    - `DATA_MOUNT_SOURCE=/mnt/data/intern-alice`
    - `TRAIN_STEPS=200`
    - `GPU_COUNT=4`（仅 DDP）
3. Run pipeline
4. 在 pipeline 页对目标 job 点 ▶（全是 manual，不会自动占 GPU）

### 队列是怎么形成的

否是是否用户点 Run / ▶Job 进入 GitLab 队列有 tag 匹配的 online runner?一直 pendingrunner concurrent 已满?等待该 runner 队列在 GPU 服务器上执行 docker jobresource_group 同组 job 串行

两层排队：

|机制|作用范围|效果|
|---|---|---|
|Runner `concurrent = 1`|单台 GPU 服务器|同一机器上 job 串行|
|`resource_group`|同一 GitLab 项目内|同组 job 不并行（如两个 `single_gpu_train` 不会同时跑）|

不是全局 GPU 调度：别的项目、别的 group 的 job 不受你的 `resource_group` 约束。

### 选哪台机器、怎么排队

|意图|点的 job|行为|
|---|---|---|
|任意空闲 GPU 服务器|`gpu_smoke_auto` / `single_gpu_train`|tag=`lab-gpu`，GitLab 分配给任意空闲 runner|
|指定 gpu01|`gpu_smoke_gpu01`|仅 gpu01 runner 会接|
|指定 gpu02|`gpu_smoke_gpu02`|仅 gpu02 runner 会接|
|单机多卡 DDP|`multi_gpu_ddp`|在一台机器上起 4 进程，不是跨机训练|

若 A、B 两人几乎同时点 `single_gpu_train`：

- 两台都有 online 的 `lab-gpu` runner → 可能各跑一台
- 只有一台 online → 第二人在 GitLab 里 pending，等第一台 `concurrent` 空出来
- 同一项目里两个 `single_gpu_train` 还会被 `resource_group: lab-gpu-single-train` 再串行一层

### 当前版本的局限（文档也写了）

- 不支持跨服务器 DDP
- 不支持 Slurm 式预约、抢占、公平份额
- 训练 job 不能像 smoke 一样指定 `gpu01`/`gpu02`（除非以后加对应 job）

---

## 总结

1. GitLab Runner 状态还没查过——只验证了 Docker/GPU；下一步应在 GitLab UI 看 runner 是否 online，并跑一遍 `gpu_smoke_auto`。
2. Tag 管理：`lab-gpu`（池）+ `gpu01`/`gpu02`（机器）+ 可选 `v100`/`cu124`；每台 `concurrent=1`；第二台用 `gitlab-runner register` + shell executor。
3. 训练进队列：Run pipeline → 设变量 → 手动点 `single_gpu_train` 或 `multi_gpu_ddp`；排队靠 runner 并发限制 + `resource_group`，不是自动调度。

如果你愿意切到 Agent 模式，我可以帮你在本机查 `gitlab-runner` 状态，或在 CI 里补上 `single_gpu_train_gpu01/gpu02` 这类 per-server 训练 job。