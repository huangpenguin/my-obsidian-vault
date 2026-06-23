核心设计：

- Docker 基底：`pytorch/pytorch:2.6.0-cuda12.4-cudnn9-devel`
- torch 约束：`torch>=2.6.0,<2.7.0`
- GitLab CI 使用 `shell executor` + runner tags：
    - `lab-gpu`
    - `gpu01`
    - `gpu02`
    - `v100`
    - `cu124`
- 手动 job：
    - `gpu_smoke_auto`
    - `gpu_smoke_gpu01`
    - `gpu_smoke_gpu02`
    - `single_gpu_train`
    - `multi_gpu_ddp`
- 数据契约：
    - host: `/mnt/data/<their-folder>`
    - container: `/data`
    - CI variables: `DATA_MOUNT_SOURCE`, `DATA_MOUNT_TARGET`

## 已验证

本机已成功：

python3 -m py_compile scripts/*.py

ruby -e "require 'yaml'; YAML.load_file('.gitlab-ci.yml'); YAML.load_file('.gitlab/ci/gpu.yml')"

docker build -t lab-gpu-runbook .

docker run --rm --gpus all lab-gpu-runbook python scripts/print_env.py

docker run --rm --gpus all lab-gpu-runbook python scripts/smoke_gpu.py

docker run --rm --gpus all lab-gpu-runbook python scripts/train_synthetic.py --steps 5

docker run --rm --gpus all -v /mnt/data:/data:ro -e DATA_DIR=/data lab-gpu-runbook python scripts/print_env.py

docker run --rm --gpus all --shm-size 16g lab-gpu-runbook torchrun --standalone --nproc_per_node=4 scripts/train_ddp.py --steps 3

实际验证结果符合预期：

- `Torch: 2.6.0+cu124`
- `Torch CUDA build: 12.4`
- `cuDNN: 90100`
- `CUDA device count: 4`
- `Tesla V100-PCIE-32GB`
- `GPU capability: sm_70`
- 4 卡 DDP `world_size=4` 成功

有一个验证细节：第一次手动 DDP 没加 `--shm-size 16g`，NCCL 报 `/dev/shm` 不足；已把这个写进 `docs/05-troubleshooting.md`，并用正确参数重跑通过。CI 模板里本来就已包含 `--shm-size 16g`。