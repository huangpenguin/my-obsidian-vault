## Dev Container 需要改哪些挂载

默认模板现在不再挂载任何宿主机数据目录，所以新项目可以先直接打开 Dev Container。

如果你的项目需要数据目录挂到容器 `/data`，需要手动改目标项目里的：

.devcontainer/devcontainer.json

参考模板里的：

.devcontainer/devcontainer.local.json.example

### 本机普通目录

"initializeCommand": "mkdir -p \"${localEnv:HOME}/.local/share/my-project-data\"",

"mounts": [

"source=${localEnv:HOME}/.local/share/my-project-data,target=/data,type=bind"

]

### 你这台 gpu01 的 autofs

如果数据在宿主机 `/mnt/data`：

"initializeCommand": "ls /mnt/data >/dev/null",

"mounts": [

"source=/mnt/data,target=/data,type=bind"

]


今後は以下の手順で開発・検証を進められます。

1. Cursorで `/home/huang/code/cst_ai/BasicSR` を開く。
    
2. コマンドパレットから **「Dev Containers: Reopen in Container」** を実行。
    
3. 初回起動時は `postCreateCommand` による自動環境構築を待つ（数分程度）。
    
4. 構築完了後、コンテナ内のターミナルで直接スクリプトを実行可能。
    




## 需要改 Dockerfile 吗？

不需要。 数据挂载是 运行时 / devcontainer 配置，不是镜像构建内容。

|改什么|何时需要|
|---|---|
|[`Dockerfile`](vscode-file://vscode-app/Applications/Cursor.app/Contents/Resources/app/out/vs/code/electron-sandbox/workbench/Dockerfile)|改 Python/CUDA/系统包、`vscode` 用户等镜像内容时|
|[`devcontainer.json`](vscode-file://vscode-app/Applications/Cursor.app/Contents/Resources/app/out/vs/code/electron-sandbox/workbench/.devcontainer/devcontainer.json)|改 mounts、containerEnv、GPU 参数时|
|代码|统一读 `os.environ["DATA_DIR"]` 等|

你要做的是改 `devcontainer.json`（以及可选地统一 smoke test 读 env），然后 Rebuild Container（重建容器实例），不是改 Dockerfile 再 build image。



### 第 1 步：设置环境变量并 Rebuild（必做）

在 gpu01 宿主机 或 Cursor 启动 Dev Container 之前：

export DATA_MOUNT_SOURCE=/home/huang/code/cst_ai/input

然后在 Cursor：Dev Containers: Rebuild Container

> 不需要改 Dockerfile，也不需要 rebuild image，只要 Rebuild Container。

若 Rebuild 报错 `ERROR: Set host env DATA_MOUNT_SOURCE`，说明变量没设好，先在宿主机 shell 里 `export` 再 Rebuild。

### 第 2 步：验证挂载

Rebuild 后进容器终端：

echo $DATA_DIR

ls $DATA_DIR/data_0607

find $DATA_DIR/data_0607 -name '*.tif' | head -20

确认能看到 cylinder 对应的子目录名（记下完整路径）。

### 第 3 步：验证 Python 环境

cd /workspace

source .venv/bin/activate # 或 uv run

python -c "import torch; print(torch.cuda.device_count())" # 应为 4

若 `import cv2` 或 `import basicsr` 报错，需要修 Dockerfile 加 `libgl1` 等系统包，或先用 PIL 版 smoke test（当前脚本主要用 PIL，可能仍能跑）。

### 第 4 步：先跑 cylinder smoke test（单 GPU，现有脚本）

在没做四 GPU 脚本前，可先用现有脚本：

uv run python scripts/smoke_test_swinir_gray_dn_battery.py \

--input-dir "$DATA_DIR/data_0607/<你的cylinder子目录>" \

--output-dir results/swinir_gray_dn_cylinder_smoke

把 `<你的cylinder子目录>` 换成第 2 步找到的实际路径。

### 第 5 步：四 GPU 并行（需 Agent 继续实现）

原计划里的 `smoke_test_swinir_gray_dn.py`（`--devices 0,1,2,3`）还没写。若 cylinder 图很多、想加速，切换到 Agent 模式 让我补这个脚本。

### 第 6 步：看结果

打开 `results/swinir_gray_dn_cylinder_smoke/noise{15,25,50}/`，对比 `*_input.png` 和 `*_SwinIR_grayDN.png`，判断预训练模型对 CT 辐射噪声是否有效。

---

## 一句话总结

- 已做：devcontainer 配置、DATA_DIR 代码改造、文档、`.venv` 重建
- 卡在你这：没 Rebuild → `/data` 挂载未生效，cylinder 数据容器里还看不到
- 还没做：四 GPU 脚本、cylinder 实际跑 test、结果评估

你现在最该做的一件事：

export DATA_MOUNT_SOURCE=/home/huang/code/cst_ai/input

然后 Rebuild Container，验证 `ls $DATA_DIR/data_0607` 能看到数据，再跑 smoke test。

Rebuild 成功后，若要继续四 GPU 脚本和自动找 cylinder 路径，切到 Agent 模式告诉我即可。