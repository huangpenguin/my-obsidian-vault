system update->docker file  edit(NVIDIA Driver 570+;cuda:12.4.1) ->pth choose?/data preprocess?(network input/ output )，dataloader

已将 `~/downloaded_images` 下的 4 个数据集目录移动到：

`/home/huang/code/cst_ai/input/data_0607/`

共 14,782 个文件，约 49GB（与 rclone 下载量一致）。

## 关于解压

检查后发现数据没有 zip/tar/7z 等压缩包，rclone 下载的已是可直接使用的文件：

|类型|数量|
|---|---|
|`.tif`|14,722|
|`.tsv`|38|
|`.txt`|18|
|`.pdf`|4|

因此无需解压步骤。

## 目录结构（按原文件名保留）

input/data_0607/

├── 260520_1-battery_data/ (34M)

│ ├── 260520_battery image tiff data/ # 单张汇总 tiff

│ └── 260520_battery image tsv data/ # 对应 tsv

├── 260520_2-Cylinderblock_data/ (27G)

│ ├── 260520_cylinderblock image tiff data/

│ ├── 260520_cylinderblock image tsv data/

│ └── 260520_cylinderblock image data/

│ └── {序号}-{电压}_{电流}/XY/data_XXXX.tif

├── 260527_3-battery_data/ (25M)

└── 260527_4-Cylinderblock_data/ (22G)

## 数据现状（`260520_1-battery_data`）

Battery 的 tiff 汇总图特征：

| 属性   | 值                        |
| ---- | ------------------------ |
| 格式   | 单通道 `uint16`，LZW 压缩      |
| 尺寸   | 约 `1280×1148/1149`       |
| 动态范围 | `0 ~ 53000~65535`（随曝光变化） |
| 数量   | 10 张，同一物体不同 `110kV + uA` |

---
## 改动摘要

### 1. `.devcontainer/devcontainer.json`

- 名称：`BasicSR GPU Dev`
- GPU：`--gpus all`，`--shm-size 16g`
- 挂载：
    - `/data` ← `/mnt/data`
    - `/input` ← `cst_ai/input`（1_battery TIFF 等）
- 环境变量：`PYTHONPATH`、`INPUT_DIR`、`BATTERY_TIFF_DIR`
- `postCreateCommand`：只跑 `setup.sh` 装环境，不执行任何训练/测试脚本
- Python 解释器：`/workspace/.venv/bin/python`

### 2. `.devcontainer/setup.sh`（新建）

首次进容器时自动：

uv venv --python 3.11

uv pip install torch torchvision (cu124)

uv pip install -r requirements.txt && uv pip install -e .

并验证 `basicsr` / `torch.cuda` 是否可用。

### 3. `Dockerfile`

- 增加 OpenCV 需要的系统库（`libgl1` 等）
- 默认仍是 `CMD ["bash"]`，不自动跑任何项目脚本

### 4. 其他

- 更新 `devcontainer.local.json.example`（无 `/mnt/data` 机器可本地覆盖）
- `smoke_test` 脚本支持 `BATTERY_TIFF_DIR` 环境变量（容器内已预设）

## 使用方式

1. 在 Cursor 打开 `/home/huang/code/cst_ai/BasicSR`
2. 命令面板 → Dev Containers: Reopen in Container
3. 等待 `postCreateCommand` 完成（首次约几分钟）
4. 之后在容器终端直接写代码、跑命令，例如：
    
    python scripts/smoke_test_swinir_gray_dn_battery.py # 手动执行
    
    PYTHONPATH=./ python basicsr/train.py -opt options/train/...
    

容器内数据路径：

- 电池 TIFF：`/input/data_0607/260520_1-battery_data/260520_battery image tiff data`
- 共享数据盘：`/data`
- 预训练权重：`/workspace/experiments/pretrained_models/SwinIR/`