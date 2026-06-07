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