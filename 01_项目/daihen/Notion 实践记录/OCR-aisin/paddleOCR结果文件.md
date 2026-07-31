---
title: "paddleOCR结果文件"
publish: false
tags: ["OCR","项目实践"]
---
# paddleOCR结果文件

## 模型文件类型解释

### 1. latest（最新模型）

- **文件**：latest.pdparams
- **含义**：最近一次保存的模型权重
- **用途**：用于恢复训练（resume training）
- **特点**：会被不断覆盖更新

### 2. best_accuracy（最佳精度模型）

- **文件**：best_accuracy.pdparams
- **含义**：在验证集上表现最好的模型
- **用途**：用于最终推理和部署
- **特点**：只有当验证指标超过历史最佳时才会更新

### 3. iter_xxx（特定迭代模型）

- **文件**：`output/det_r50_db++_baseline/iter_2000.pdparams`
- **含义**：特定step数保存的模型
- **用途**：用于分析训练过程或回退到特定阶段

## 查看当前模型文件

```bash
# 查看模型输出目录
ls -la output/det_r50_db++_baseline/

# 典型的输出应该包含：
# latest.pdparams          - 最新模型权重
# latest.pdopt            - 最新优化器状态
# best_accuracy.pdparams  - 最佳模型权重
# best_accuracy.pdopt     - 最佳模型优化器状态
# config.yml              - 训练配置备份
# train.log               - 训练日志

```

## 使用建议

### 继续训练时使用

```bash
# 使用latest恢复训练
nohup python -u -m paddle.distributed.launch --ips="192.168.3.18" --gpus "0,1,2,3" \\
  PaddleOCR/tools/train.py \\
  -o Global.checkpoints=output/det_r50_db++_baseline/latest \\
  -c ../tray_recog/configs/det/det_r50_db++_baseline.yml

```

### 推理/评估时使用

```bash
# 使用best_accuracy进行推理
python PaddleOCR/tools/infer_det.py \\
  -c ../tray_recog/configs/det/det_r50_db++_baseline.yml \\
  -o Global.checkpoints=output/det_r50_db++_baseline/best_accuracy \\
  Global.infer_img=test_image.jpg

```

## 模型选择策略

### 开发阶段

- 使用 **latest** 继续训练
- 使用 **best_accuracy** 进行测试和验证

### 生产部署

- 始终使用 **best_accuracy**
- 这是在验证集上表现最佳的模型

## 检查模型信息

```bash
# 查看训练日志中的最佳指标
tail -100 output/det_r50_db++_baseline.yml.log | grep "best"

# 查看模型文件时间戳
ls -lt output/det_r50_db++_baseline/*.pdparams

```

**总结**：对于您当前的情况，如果要继续训练使用`latest`，如果要测试模型效果使用`best_accuracy`。通常`best_accuracy`是您最终想要的模型。
