---
title: "Decision Forests"
publish: false
tags: ["机器学习"]
---
# Decision Forests

```python
import tensorflow as tf
import tensorflow_decision_forests as tfdf
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Comment this if the data visualisations doesn't work on your side
%matplotlib inline
```

```python
print("TensorFlow v" + tf.__version__)
print("TensorFlow Decision Forests v" + tfdf.__version__)
```

```python
#可以在左上角选择别的input
train_file_path = "../input/house-prices-advanced-regression-techniques/train.csv"
dataset_df = pd.read_csv(train_file_path)
print("Full train dataset shape is {}".format(dataset_df.shape))
```

```python
#查看数据
dataset_df.head(3)
#dataset_df.iloc[0:4, [0, 1, 2, 3, -3, -2, -1]]
#dataset_df.info()

#list(set(dataset_df.dtypes.tolist()))#利用set特性去重复
#df_num = dataset_df.select_dtypes(include = ['float64', 'int64'])
#df_num.head()
#df_num.hist(figsize=(16, 20), bins=50, xlabelsize=8, ylabelsize=8);
```

```python
#丢数据
if "Id" in dataset_df.columns.tolist():
    dataset_df = dataset_df.drop('Id', axis=1)
dataset_df.head(3)

```

```python
print(dataset_df['SalePrice'].describe())
plt.figure(figsize=(9, 8))
sns.histplot(dataset_df['SalePrice'], color='g', bins=100, kde=True, alpha=0.4)

```

```python
#This dataset contains a mix of numeric, categorical and missing features. 
#TF-DF supports all these feature types natively, and no preprocessing is required.

import numpy as np

def split_dataset(dataset, test_ratio=0.30):
  test_indices_mask = np.random.rand(len(dataset)) < test_ratio
  return dataset[~test_indices_mask], dataset[test_indices_mask]#bool matrix

train_ds_pd, valid_ds_pd = split_dataset(dataset_df)
print("{} examples in training, {} examples in testing.".format(
    len(train_ds_pd), len(valid_ds_pd)))
```

```python
#create tf dataset
label = 'SalePrice'
train_ds = tfdf.keras.pd_dataframe_to_tf_dataset(train_ds_pd, label=label, task = tfdf.keras.Task.REGRESSION)
valid_ds = tfdf.keras.pd_dataframe_to_tf_dataset(valid_ds_pd, label=label, task = tfdf.keras.Task.REGRESSION)

```

```python
#model select
rf = tfdf.keras.RandomForestModel(hyperparameter_template="benchmark_rank1", task=tfdf.keras.Task.REGRESSION)
rf.compile(metrics=["mse"]) # Optional, you can use this to include a list of eval metrics
```

```python
rf.fit(x=train_ds)
#visualize
tfdf.model_plotter.plot_model_in_colab(rf, tree_idx=0, max_depth=3)
```

```python
import matplotlib.pyplot as plt
logs = rf.make_inspector().training_logs()
plt.plot([log.num_trees for log in logs], [log.evaluation.rmse for log in logs])
plt.xlabel("Number of trees")
plt.ylabel("RMSE (out-of-bag)")
plt.show()
```

```python
#see the result
inspector = rf.make_inspector()
inspector.evaluation()
```

```python
# evaluation
evaluation = rf.evaluate(x=valid_ds,return_dict=True)

for name, value in evaluation.items():
  print(f"{name}: {value:.4f}")
```

```python
#importances
print(f"Available variable importances:")
for importance in inspector.variable_importances().keys():
  print("\t", importance)
  
# variable_importances() 返回的字典中，有一个键名是 NUM_AS_ROOT，它代表特征作为根节点的次数。
# 决策树模型会在不同节点上分裂特征，根节点是最重要的节点之一。
# 如果一个特征在决策树模型中被选作根节点的次数越多，则表明该特征在模型中的影响力越大。
inspector.variable_importances()["NUM_AS_ROOT"]

# draw importance
plt.figure(figsize=(12, 4))

# Mean decrease in AUC of the class 1 vs the others.
variable_importance_metric = "NUM_AS_ROOT"
variable_importances = inspector.variable_importances()[variable_importance_metric]

# Extract the feature name and importance values.
#
# `variable_importances` is a list of <feature, importance> tuples.
feature_names = [vi[0].name for vi in variable_importances]
feature_importances = [vi[1] for vi in variable_importances]
# The feature are ordered in decreasing importance value.
feature_ranks = range(len(feature_names))

bar = plt.barh(feature_ranks, feature_importances, label=[str(x) for x in feature_ranks])
plt.yticks(feature_ranks, feature_names)
plt.gca().invert_yaxis()

# TODO: Replace with "plt.bar_label()" when available.
# Label each bar with values
for importance, patch in zip(feature_importances, bar.patches):
  plt.text(patch.get_x() + patch.get_width(), patch.get_y(), f"{importance:.4f}", va="top")

plt.xlabel(variable_importance_metric)
plt.title("NUM AS ROOT of the class 1 vs the others")
plt.tight_layout()
plt.show()
```

```python
#test
test_file_path = "../input/house-prices-advanced-regression-techniques/test.csv"
test_data = pd.read_csv(test_file_path)
ids = test_data.pop('Id')

test_ds = tfdf.keras.pd_dataframe_to_tf_dataset(
    test_data,
    task = tfdf.keras.Task.REGRESSION)

preds = rf.predict(test_ds)
output = pd.DataFrame({'Id': ids,
                       'SalePrice': preds.squeeze()})

output.head()

```

```python
#submit
sample_submission_df = pd.read_csv('../input/house-prices-advanced-regression-techniques/sample_submission.csv')
sample_submission_df['SalePrice'] = rf.predict(test_ds)
sample_submission_df.to_csv('/kaggle/working/submission.csv', index=False)
sample_submission_df.head()

```

| RFE Features | 0.047646 | 0.127766 | 0.020303 | 26.735729 | RF with top 50 RFE features, n_trees = 800 |
| --- | --- | --- | --- | --- | --- |
