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
    

**【実行コマンド例】**

Bash

```
# スモークテストの手動実行（環境変数はコンテナ内で設定済み）
python scripts/smoke_test_swinir_gray_dn_battery.py

# BasicSRの学習コマンド例
PYTHONPATH=./ python basicsr/train.py -opt options/train/...
