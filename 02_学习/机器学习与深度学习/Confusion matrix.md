---
title: "Confusion matrix"
publish: false
tags: ["机器学习"]
---
# Confusion matrix

混淆矩阵（Confusion Matrix）是一个用于评估分类模型效果的工具，特别适用于二分类问题。它展示了模型预测结果和实际情况的对比。混淆矩阵的形式如下：

|  | 实际正类 (Positive) | 实际负类 (Negative) |
| --- | --- | --- |
| 预测正类 (Positive) | 真正例 (True Positive, TP) | 假正例 (False Positive, FP) |
| 预测负类 (Negative) | 假负例 (False Negative, FN) | 真负例 (True Negative, TN) |

### 解释

- **TP (True Positive)**：真实标签为正，模型预测为正。
- **FP (False Positive)**：真实标签为负，模型预测为正（误报）。
- **FN (False Negative)**：真实标签为正，模型预测为负（漏报）。
- **TN (True Negative)**：真实标签为负，模型预测为负。

### 基于混淆矩阵的常用指标

1. **准确率 (Accuracy)**：
    - 表示模型整体的正确预测率。
    
    $$
    \text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}
    $$
    
2. **精确率 (Precision)**：
    - 表示在模型预测为正类的样本中，实际为正类的比例。
    - 适用于注重减少假正例（误报）的场景。
    
    $$
    \text{Precision} = \frac{TP}{TP + FP}
    $$
    
3. **召回率 (Recall)**（或灵敏度、查全率）：
    - 表示在所有实际为正类的样本中，模型正确识别为正类的比例。
    - 适用于注重减少假负例（漏报）的场景。
    
    $$
    \text{Recall} = \frac{TP}{TP + FN}
    $$
    
4. **F1分数 (F1 Score)**：
    - 精确率和召回率的调和平均数，用于平衡精确率和召回率的情况。
    - 适用于不希望偏重精确率或召回率的场景。
    
    $$
    \text{F1 Score} = \frac{2 \times \text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}
    $$
    
5. **特异度 (Specificity)**：
    - 表示在所有实际为负类的样本中，模型正确识别为负类的比例。
    - 适用于某些场景下需要特别关注负类识别率的情况。
    
    $$
    \text{Specificity} = \frac{TN}{TN + FP}
    $$
    
6. **假阳性率 (False Positive Rate, FPR)**：
    - 表示在所有实际为负类的样本中，被错误预测为正类的比例。
    
    $$
    \text{FPR} = \frac{FP}{TN + FP}
    $$
