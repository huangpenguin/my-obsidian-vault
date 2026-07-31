---
title: "Forecasting Model Experiments"
publish: false
tags: ["机器学习"]
---
# Forecasting Model Experiments

```python
def get_top_features_by_rank(metric_col_name: str, feature_number: int):
    features_df = all_ranks.copy()
    
    # features_df = features_df.sort_values(by=['feature_number'])
    
    # TODO: [:feature_number]
    
    # top n rows ordered by multiple columns
    features_df = features_df.nlargest(feature_number, [metric_col_name])
    
    result_list = list(features_df['Feature'])
    return result_list

def model_check(X, y, estimator, model_name, model_description, cv):
    model_table = pd.DataFrame()

    cv_results = cross_validate(estimator,
                                X,
                                y,
                                cv=cv,
                                scoring='neg_root_mean_squared_error',
                                return_train_score=True,
                                n_jobs=-1)

    train_rmse = -cv_results['train_score'].mean()
    test_rmse = -cv_results['test_score'].mean()
    test_std = cv_results['test_score'].std()
    fit_time = cv_results['fit_time'].mean()

    attributes = {
        'model_name': model_name,
        'train_score': train_rmse,
        'test_score': test_rmse,
        'test_std': test_std,
        'fit_time': fit_time,
        'description': model_description,
    }
    
    model_table = pd.DataFrame(data=[attributes])
    return model_table
```

---

```python
# baseline RF
baseline = model_check(X, y, rf, 'Baseline RF', "Baseline RF (100 trees, all features)", kf)
result_df = baseline
```

```python
n_estimators = [200, 300, 400, 500, 600, 700, 800, 900, 1000]
for n in n_estimators:
    rf2 = RandomForestRegressor(n_jobs=-1, n_estimators=n, verbose=1)
    description = "RF with n_trees = {}".format(n)
    model_check_df = model_check(X, y, rf2, 'RF - All Features', description, kf)
    
    # concatenate
    frames = [result_df, model_check_df]
    result_df = pd.concat(frames)
```

```python
# subset of features selected by RFE feature importance
top_rfe_features = 50
rfe_features = get_top_features_by_rank('rfe_importance', top_rfe_features)
X_important_features = X[rfe_features]
n_estimators = [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000]
for n in n_estimators:
    rf2 = RandomForestRegressor(n_jobs=-1, n_estimators=n, verbose=1)
    description = "RF with top {} RFE features, n_trees = {}".format(top_rfe_features, n)
    model_check_df = model_check(X_important_features, y, rf2, 'RFE Features', description, kf)
    
    # concatenate
    frames = [result_df, model_check_df]
    result_df = pd.concat(frames)
```

```python
# subset of features selected by RF embedded Feature Selection
X_embedded_features = X[embeded_rf_feature]
n_estimators = [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000]
for n in n_estimators:
    rf2 = RandomForestRegressor(n_jobs=-1, n_estimators=n, verbose=1)
    description = "RF witn n_trees = {}".format(n)
    model_check_df = model_check(X_embedded_features, y, rf2, 'RF - Embedded Features', description, kf)
    
    # concatenate
    frames = [result_df, model_check_df]
    result_df = pd.concat(frames)
```

```python
# train RF with the top importance feautres selected via the permutation method
top_features = 50
important_features = get_top_features_by_rank('permutation_importance', top_features)
X_important_features = X[important_features]

n_estimators = [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000]
for n in n_estimators:
    rf2 = RandomForestRegressor(n_jobs=-1, n_estimators=n, verbose=1)
    description = "RF with top {} permutatively important features, n_trees = {}".format(top_rfe_features, n)
    model_check_df = model_check(X_important_features, y, rf2, 'RF - Permutatively Important Features', description, kf)
    
    # concatenate
    frames = [result_df, model_check_df]
    result_df = pd.concat(frames)

```

```python
# train RF with the top importance feautres selected via the drop-column method
top_features = 50
important_features = get_top_features_by_rank('drop_col_importance', top_features)
X_important_features = X[important_features]

n_estimators = [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000]
for n in n_estimators:
    rf2 = RandomForestRegressor(n_jobs=-1, n_estimators=n, verbose=1)
    description = "RF with top {} drop-col-important features, n_trees = {}".format(top_rfe_features, n)
    model_check_df = model_check(X_important_features, y, rf2, 'RF - Drop-Column Important Features', description, kf)
    
    # concatenate
    frames = [result_df, model_check_df]
    result_df = pd.concat(frames)
```

```python
display(result_df.reset_index().drop(['index'], axis=1).style.background_gradient(cmap='summer_r'))
```
