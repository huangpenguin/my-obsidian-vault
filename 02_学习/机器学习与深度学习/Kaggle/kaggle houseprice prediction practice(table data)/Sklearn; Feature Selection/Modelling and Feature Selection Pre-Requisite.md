---
title: "Modelling and Feature Selection Pre-Requisite"
publish: false
tags: ["机器学习"]
---
# Modelling and Feature Selection Pre-Requisite

---

```python
# Loading neccesary packages for modelling and feature selection
from sklearn.model_selection import cross_val_score, KFold, cross_validate
from sklearn.feature_selection import RFE, f_regression
from sklearn.linear_model import (LinearRegression, Ridge, Lasso)
from sklearn.preprocessing import MinMaxScaler
from sklearn.ensemble import RandomForestRegressor

# Setting kfold for future use
kf = KFold(10, random_state=42, shuffle=True)

# Train our baseline RF Regression model for feature importance scoring/feature selection
n_trees = 100
rf = RandomForestRegressor(n_jobs=-1, n_estimators=n_trees, verbose=1)
rf.fit(X, y)

```

```python
# 使用递归特征消除（RFE）方法进行特征选择的一个实现
def rfe_select_featurs(X, y, estimator, num_features) -> List[str]:
    rfe_selector = RFE(estimator=estimator, 
                       n_features_to_select=num_features, 
                       step=10, verbose=5)
    rfe_selector.fit(X, y)
    rfe_support = rfe_selector.get_support()
    rfe_feature = X.loc[:,rfe_support].columns.tolist()
    print(str(len(rfe_feature)), 'selected features')
    
    return rfe_feature
```

```python
# total list of features
colnames = X.columns
# Define dictionary to store our rankings
ranks = {}
# Create our function which stores the feature rankings to the ranks dictionary
def ranking(ranks, names, order=1):
    minmax = MinMaxScaler()
    ranks = minmax.fit_transform(order*np.array([ranks]).T).T[0]
    ranks = map(lambda x: round(x,2), ranks)
    return dict(zip(names, ranks))
```

## **Feature Importance Scores From Model and via RFE**

```python
# Do FRE feature importance scoring - 
# stop the search when only the last feature is left
# RFE 的基本思想是通过反复训练模型并删除最不重要的特征，来减少特征的数量。
# 每次训练模型后，特征重要性最低的特征会被移除，直到达到目标的特征数为止。
rfe = RFE(rf, n_features_to_select=1, verbose =3 )
rfe.fit(X, y)
ranks["RFE_RF"] = ranking(list(map(float, rfe.ranking_)), colnames, order=-1)

# Extract feature importance coefficients as calculated by the trained model
ranks["RF"] = ranking(rf.feature_importances_, colnames);
```

```python
# all ranks
# Put the mean scores into a Pandas dataframe
rfe_rf_df = pd.DataFrame(list(ranks['RFE_RF'].items()), columns= ['Feature','rfe_importance'])
rf_df = pd.DataFrame(list(ranks['RF'].items()), columns= ['Feature','alg_importance'])

all_ranks = pd.merge(rfe_rf_df, rf_df, on=['Feature'])

display(all_ranks.head(10))
```

## Embedded Feature Selection: Selecting Features From a Model

```python
from sklearn.feature_selection import SelectFromModel

embeded_rf_selector = SelectFromModel(rf, max_features=200)
embeded_rf_selector.fit(X, y)

embeded_rf_support = embeded_rf_selector.get_support()
embeded_rf_feature = X.loc[:,embeded_rf_support].columns.tolist()
print(str(len(embeded_rf_feature)), 'selected features')
print(embeded_rf_feature)
```

## Permutable Feature Importance

```python
from sklearn.inspection import permutation_importance

# Here's how you use permutation importance
def get_permutation_importance(X, y, model) -> pd.DataFrame:
    result = permutation_importance(model, X, y, n_repeats=1,
                                random_state=0)
    
    # permutational importance results
    result_df = pd.DataFrame(colnames,  columns=['Feature'])
    result_df['permutation_importance'] = result.get('importances')
    
    return result_df
    

permutate_df = get_permutation_importance(X, y, rf)
permutate_df.sort_values('permutation_importance', 
                   ascending=False)[
                                    ['Feature','permutation_importance'
                                    ]
                                  ][:30].style.background_gradient(cmap='Blues')
```

## Drop-Column Importance

```python
from sklearn.base import clone 

def drop_col_feat_imp(model, X_train, y_train, random_state = 42):
    
    # clone the model to have the exact same specification as the one initially trained
    model_clone = clone(model)
    # set random_state for comparability
    model_clone.random_state = random_state
    # training and scoring the benchmark model
    model_clone.fit(X_train, y_train)
    benchmark_score = model_clone.score(X_train, y_train)
    # list for storing feature importances
    importances = []
    
    # iterating over all columns and storing feature importance (difference between benchmark and new model)
    for col in X_train.columns:
        model_clone = clone(model)
        model_clone.random_state = random_state
        model_clone.fit(X_train.drop(col, axis = 1), y_train)
        drop_col_score = model_clone.score(X_train.drop(col, axis = 1), y_train)
        importances.append( round( (benchmark_score - drop_col_score)/benchmark_score, 4) )
    
    importances_df = pd.DataFrame(X_train.columns, columns=['Feature'])
    importances_df['drop_col_importance'] = importances
    return importances_df

drop_col_impt_df = drop_col_feat_imp(rf, X, y)

drop_col_impt_df.sort_values('drop_col_importance', 
                   ascending=False)[
                                    ['Feature','drop_col_importance'
                                    ]
                                  ][:30].style.background_gradient(cmap='Blues')
```

---

## **Merging All Feature Importance Metrics Into a Single Results Dataframe**

```python
# merge drop_col_impt_df
all_ranks = pd.merge(all_ranks, drop_col_impt_df, on=['Feature'])

# merge permutate_df
all_ranks = pd.merge(all_ranks, permutate_df, on=['Feature'])

# calculate average feature importance
average_fi_pipeline = pdp.PdPipeline([
    pdp.ApplyToRows(
        lambda row: (row['drop_col_importance'] + row['permutation_importance'] + row['rfe_importance'] + row['alg_importance'])/4, 
        colname='mean_feature_importance') # 'mean_feature_importance
])

all_ranks = average_fi_pipeline.apply(all_ranks)

display(all_ranks.reset_index().drop(['index'], axis=1).style.background_gradient(cmap='summer_r'))
```
