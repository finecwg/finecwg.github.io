---
layout: single
title: "Machine learning protocol with scikit-learn's pipeline"
categories: computer-science
tags: [machine learning, scikit-learn]
toc: true
toc_sticky: true
toc_label: "ML pipeline"
author_profile: false
---

<head>
  <style>
    table.dataframe {
      white-space: normal;
      width: 100%;
      height: 240px;
      display: block;
      overflow: auto;
      font-family: Arial, sans-serif;
      font-size: 0.9rem;
      line-height: 20px;
      text-align: center;
      border: 0px !important;
    }

    table.dataframe th {
      text-align: center;
      font-weight: bold;
      padding: 8px;
    }

    table.dataframe td {
      text-align: center;
      padding: 8px;
    }

    table.dataframe tr:hover {
      background: #b8d1f3;
    }

    .output_prompt {
      overflow: auto;
      font-size: 0.9rem;
      line-height: 1.45;
      border-radius: 0.3rem;
      -webkit-overflow-scrolling: touch;
      padding: 0.8rem;
      margin-top: 0;
      margin-bottom: 15px;
      font: 1rem Consolas, "Liberation Mono", Menlo, Courier, monospace;
      color: $code-text-color;
      border: solid 1px $border-color;
      border-radius: 0.3rem;
      word-break: normal;
      white-space: pre;
    }

.dataframe tbody tr th:only-of-type {
vertical-align: middle;
}

.dataframe tbody tr th {
vertical-align: top;
}

.dataframe thead th {
text-align: center !important;
padding: 8px;
}

.page\_\_content p {
margin: 0 0 1.3rem !important;
}

.page\_\_content li > p {
margin: 0 0 0.6rem !important;
}

.page\_\_content p > strong {
font-size: 1.0rem !important;
}

  </style>
</head>

Data science and machine learning require various libraries and the construction of sophisticated pipelines. This article provides a detailed explanation of the tools and methods needed from data processing to model evaluation and SHAP feature importance. It offers a guide for efficient data analysis and machine learning tasks using Python.

**Key Contents**

- 📚 How to install essential **data analysis** and **machine learning libraries**
- 🛠️ How to use **GridSearchCV** for **pipeline construction** and optimization
- 📈 Methods for evaluating model performance and analyzing feature importance using **SHAP** values

# Exploring the code

This code installs the main libraries needed for data analysis and machine learning tasks. It includes `numpy` for numerical computations, `pandas` for data processing, `matplotlib` and `seaborn` for visualization, and `lightgbm`, `xgboost`, and `shap` for building machine learning models. Additionally, it installs `tqdm` to display the progress of repetitive tasks. The `matplotlib` library is specified to be installed at version 3.5.3.

```python
# install required libraries
!pip install numpy pandas matplotlib==3.5.3 seaborn scikit-learn lightgbm xgboost shap tqdm
```

This document shows how to import various libraries and functions necessary for data preprocessing, model training, and evaluation. For data analysis and visualization, it uses `pandas`, `numpy`, `matplotlib.pyplot`, and `seaborn`. To build and evaluate machine learning models, it uses several modules from `sklearn`, including preprocessing (`MinMaxScaler`, `StandardScaler`, `RobustScaler`, `IterativeImputer`, `SimpleImputer`), model selection (`train_test_split`, `GridSearchCV`, `KFold`, `StratifiedKFold`), dimensionality reduction (`PCA`), various classifiers (`DecisionTreeClassifier`, `KNeighborsClassifier`, `SVC`, `RandomForestClassifier`, `GradientBoostingClassifier`, `MLPClassifier`), pipeline construction (`ColumnTransformer`, `Pipeline`), and model evaluation (`accuracy_score`, `precision_score`, `recall_score`, `f1_score`, `roc_auc_score`, `average_precision_score`). Additionally, it uses `LGBMClassifier` from `lightgbm`, `XGBClassifier` from `xgboost`, `shap` for model explanation, and `tqdm` to visually display the progress of repetitive tasks. These tools are essential for data scientists and machine learning engineers to process data, train models, and evaluate results.

```python
import pandas as pd
import numpy as np

import matplotlib.pyplot as plt
import seaborn as sns

import sklearn
from sklearn.preprocessing import MinMaxScaler,StandardScaler,RobustScaler
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer, SimpleImputer
from sklearn.utils import class_weight
from sklearn.model_selection import train_test_split, GridSearchCV, KFold, StratifiedKFold
from sklearn.preprocessing import RobustScaler, StandardScaler
from sklearn.decomposition import PCA

from sklearn.tree import DecisionTreeClassifier
from sklearn.neighbors import KNeighborsClassifier
from sklearn.svm import SVC
from sklearn.ensemble import RandomForestClassifier
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.neural_network import MLPClassifier

from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline

from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, roc_auc_score, average_precision_score

from lightgbm import LGBMClassifier
from xgboost import XGBClassifier
import shap
from tqdm.notebook import tqdm

import os
```

This command changes the user's current working directory to `/your_directory`. It is mainly used to access data analysis or project-related files and is executed before starting work on a specific project. This allows the user to access the necessary data or files more easily.

```python
%cd /your_directory
```

This document explains how to change the settings of `sklearn` and redefine the `transform` methods of the `LGBMClassifier` and `XGBClassifier` classes. It uses the `sklearn.set_config` function to set the output format to `pandas`. Then, it redefines the `transform` methods of `LGBMClassifier` and `XGBClassifier` to return the input values as they are using a lambda function, thereby modifying the default `transform` behavior of these classes.

```python
sklearn.set_config(transform_output="pandas") # Set sklearn configuration to output pandas.
LGBMClassifier.transform = lambda self,x:x # Define the transform method for LGBMClassifier.
XGBClassifier.transform = lambda self,x:x # Define the transform method for XGBClassifier.
```

The `warnings` module in Python provides functionality for developers to issue warning messages. This code overrides the `warnings.warn` function to prevent warning messages from being displayed. This can be useful when you want to hide or ignore specific warnings.

```python
def warn(*args, **kwargs):
    pass
# Redefine the warn function of the warnings module to prevent warning messages from being displayed.
import warnings
warnings.warn = warn
```

In our example, we explore the ML code for disease classification, using life-log and genetic datas.

This function loads a dataset for a specific disease number and splits it into training and test sets. First, it loads the lifelog and biomarker data and selects the target variable based on the disease number. Then, it loads the genotype data and merges all the data together. It removes missing values from the target variable and creates labels based on disease criteria. Finally, it removes columns that will not be used for model training and removes any special characters from the column names. The resulting training and test sets are used for training and evaluating machine learning models. The `train_test_split` function splits the data into training and test sets, and the `test_size` parameter can be used to adjust the split ratio.

```python
def load_dataset(disease_no, test_size=0.2):

    lifelog = pd.read_csv("lifelog.csv")
    lifelog.rename(columns = {'col_prior':'col'}, inplace = True)

    biomarkers = pd.read_csv("biomarkers.csv")

    target_variables = {
        0 : ['Body_Mass_Index'],
        1 : ['FBS'],
        2 : ['Neutral_Fat'],
        3 : ['HDL'],
        4 : ['LDL'],
        5 : ['AST'],
        6 : ['ALT'],
        7 : ['Hemoglobin'],
        8 : ['Systolic_BP'],
        9 : ['Diastolic_BP']
    }

    file_list = {
        0 : 'BMI',
        1 : 'FBS',
        2 : 'Neutral_fat',
        3 : 'HDL',
        4 : 'LDL',
        5 : 'AST',
        6 : 'ALT',
        7 : 'Hemoglobin',
        8 : 'Systolic_BP',
        9 : 'Diastolic_BP'
    }

    genotype = pd.read_csv("genotype." + str(file_list[disease_no]) + "." + str(file_list[disease_no]) + ".tsv", delimiter='\t')
    genotype.rename(columns = {'col_prior': 'col'}, inplace = True)

    df = pd.merge(lifelog, genotype, on = 'col', how = 'inner')
    df = pd.merge(df, biomarkers, on = 'col', how = 'inner')


    biomarkers_list = ['Body_Mass_Index', 'FBS', 'Neutral_Fat', 'HDL', 'LDL', 'AST', 'ALT', 'Hemoglobin', 'Systolic_BP', 'Diastolic_BP']


    df.dropna(subset = target_variables[disease_no], inplace=True)


    criterion = {
        0: df['Body_Mass_Index']>=25,
        1: df['FBS']>=126,
        2: df['Neutral_Fat']>=200,
        3: df['HDL']<40,
        4: df['LDL']>=160,
        5: df['AST']>40,
        6: df['ALT']>40,
        7: df['Hemoglobin']+df['Sex']<14,
        8: df['Systolic_BP']>=130,
        9: df['Diastolic_BP']>=80
    }

    diseases = np.where(criterion[disease_no], 1, 0)
    diseases = pd.Series(diseases, index=df.index)

    x = df.copy()
    x.drop(columns=biomarkers_list, axis=1, inplace=True)
    x.drop(labels = 'col', axis = 1, inplace = True)

    # lgbm cannot adopt these letters
    x.columns = x.columns.str.replace('[', '_')
    x.columns = x.columns.str.replace(']', '_')
    x.columns = x.columns.str.replace('<', '_')
    x.columns = x.columns.str.replace('>', '_')
    x.columns = x.columns.str.replace('"', '_')
    x.columns = x.columns.str.replace("'", '_')
    x.columns = x.columns.str.replace(',', '_')
    x.columns = x.columns.str.replace(':', '_')
    x.columns = x.columns.str.replace('{', '_')
    x.columns = x.columns.str.replace('}', '_')

    x_train, x_test, y_train, y_test = train_test_split(x, diseases, test_size=test_size, stratify=diseases)

    return x_train, x_test, y_train, y_test
```

This code configures a `ColumnTransformer` to perform Principal Component Analysis (PCA) for dimensionality reduction on several categorical variables. The `ColumnTransformer` defines three PCA transformations, each set with `n_components`, extracting principal components from each set of variables. These transformations are named 'pca_A6', 'pca_H6', and 'pca_F', and are applied to different groups of variables. Finally, the `remainder='passthrough'` option ensures that the columns not transformed are retained as they are.

```python
pca_for_features = ColumnTransformer([
    ('pca_1', PCA(n_components=2), ['Cat_A6_3' ,'Cat_A6_20' ,'Cat_A6_1' ,'Cat_A6_16' ,'Cat_A6_4' ,'Cat_A6_5' ,'Cat_A6_17' ,'Cat_A6_2' ,'Cat_A6_15' ,'Cat_A6_9' ,'Cat_A6_11'
]),
    ('pca_H6', PCA(n_components=1), ['Cat_H6_10' ,'Cat_H6_11' ,'Cat_H6_14' ,'Cat_H6_15' ,'Cat_H6_4' ,'Cat_H6_7' ,'Cat_H6_1' ,'Cat_H6_16' ,'Cat_H6_9'
])
    ],
    remainder='passthrough'
)
```

Below codes are pipelines for six models: knn, dt, svm, rf, lgbm, and xgb.

```python
def knn_pipeline():

    knn_params = dict(n_neighbors = list(range(1, 31)))


    KNeighborsClassifier.transform = lambda self,x:x


    main_pipeline = Pipeline([
        ('imputer', IterativeImputer(max_iter = 5, tol = 1e-2, n_nearest_features = 10)), 
        ('scaler', RobustScaler()), 
        ('pca', pca_for_features),
        ('estimator', GridSearchCV(estimator=KNeighborsClassifier(), param_grid=knn_params, cv=StratifiedKFold(n_splits=5, shuffle=True), n_jobs=-1))
        ]
    )

    return main_pipeline
```



```python
def svm_pipeline():
    svm_params = {
        'C': [0.1, 1, 10, 100, 1000],
        'kernel': ['rbf'],
        'gamma': ['scale', 'auto'],
        'probability': [True]
    }

    SVC.transform = lambda self,x:x

    main_pipeline = Pipeline([
        ('imputer', IterativeImputer(max_iter = 5, tol = 1e-2, n_nearest_features = 10)),
        ('scaler', RobustScaler()),
        ('pca', pca_for_features),
        ('estimator', GridSearchCV(estimator=SVC(), param_grid=svm_params, cv=StratifiedKFold(n_splits=5, shuffle=True), n_jobs=-1))
        ]
    )

    return main_pipeline
```



```python
def dt_pipeline():
    dt_params = {
        'criterion': ['gini', 'entropy'],
        'max_depth': list(range(3, 16, 2)),
        'min_samples_split': list(range(2, 11, 2)),
        'min_samples_leaf': [1, 3, 5, 7, 9],
        'max_features': ['auto', 'sqrt', 'log2']
    }

    DecisionTreeClassifier.transform = lambda self,x:x

    main_pipeline = Pipeline([
        ('imputer', IterativeImputer(max_iter = 5, tol = 1e-2, n_nearest_features = 10)),
        ('scaler', RobustScaler()),
        ('pca', pca_for_features),
        ('estimator', GridSearchCV(estimator=DecisionTreeClassifier(), param_grid=dt_params, cv=StratifiedKFold(n_splits=5, shuffle=True), n_jobs=-1))
        ]
    )

    return main_pipeline
```


```python
def rf_pipeline():
    rf_params = {
        'n_estimators': [100, 500, 1000],
        'criterion': ['gini', 'entropy'],
        'min_samples_split': [2, 6, 10],
        'min_samples_leaf': [1, 3, 5],
        'max_features': [None, 'sqrt', 'log2']
    }

    RandomForestClassifier.transform = lambda self,x:x

    main_pipeline = Pipeline([
        ('imputer', IterativeImputer(max_iter = 5, tol = 1e-2, n_nearest_features = 10)),
        ('scaler', RobustScaler()),
        ('pca', pca_for_features),
        ('estimator', GridSearchCV(estimator=RandomForestClassifier(), param_grid=rf_params, cv=StratifiedKFold(n_splits=5, shuffle=True), n_jobs=-1))
        ]
    )

    return main_pipeline
```


```python
def xgb_pipeline():
    xgb_params = {
        'n_estimators': [100, 500, 1000],
        'learning_rate': [0.01, 0.1],
        'max_depth': [3, 7, 11],
        'min_child_weight': [1, 5],
        'gamma': [0, 0.2],
        'subsample': [0.7, 0.75],
        'colsample_bytree': [0.64, 0.66],
        'reg_lambda': [1, 1.2]
    }

    XGBClassifier.transform = lambda self, x: x

    main_pipeline = Pipeline([
        ('imputer', IterativeImputer(max_iter = 5, tol = 1e-2, n_nearest_features = 10)),
        ('scaler', RobustScaler()),
        ('pca', pca_for_features),
        ('estimator', GridSearchCV(estimator=XGBClassifier(), param_grid=xgb_params, cv=StratifiedKFold(n_splits=5, shuffle=True), n_jobs=-1, error_score='raise'))
        ]
    )

    return main_pipeline
```


```python
def lgbm_pipeline():
    lgbm_params = {
        'learning_rate': [0.1]
    }

    LGBMClassifier.transform = lambda self, x: x

    main_pipeline = Pipeline([
        ('imputer', IterativeImputer(max_iter = 5, tol = 1e-2, n_nearest_features = 10)),
        ('scaler', RobustScaler()),
        ('pca', pca_for_features),
        ('estimator', GridSearchCV(estimator=LGBMClassifier(verbose=-1), param_grid=lgbm_params, cv=KFold(n_splits=5, shuffle=True), n_jobs=-1))
        ]
    )

    return main_pipeline
```

This function is used to evaluate the performance of a predictive model. The input parameters are the actual values (`y_test`), predicted probabilities (`y_predproba`), a `random_state` to distinguish experiments, and a `criteria` for classification. Within the function, predictions are made based on the `criteria`, and various performance metrics such as accuracy, precision, recall, F1 score, AUROC, and AUPRC are calculated and returned in a DataFrame. Each performance metric evaluates how well the model predicts from different perspectives.

```python
def scoring(y_test, y_predproba, random_state, criteria=0.5):
    y_pred = y_predproba[:,1] ">=" criteria

    score_df = pd.DataFrame()
    score_df['accuracy'] = [accuracy_score(y_test, y_pred)]
    score_df['precision'] = [precision_score(y_test, y_pred)]
    score_df['recall'] = [recall_score(y_test, y_pred)]
    score_df['f1_score'] = [f1_score(y_test, y_pred)]
    score_df['auroc'] = [roc_auc_score(y_test, y_predproba[:,1])]
    score_df['auprc'] = [average_precision_score(y_test, y_predproba[:,1])]

    score_df.index = ['experiment' + str(random_state)]

    return score_df
```

This function calculates SHAP values using the given model and test dataset, and evaluates feature importance through these values. After transforming the `x_test` data via `main_pipeline`, it uses the appropriate SHAP Explainer based on the model type (kernel-based or tree-based). For kernel-based models (`knn`, `svm`), it uses `KernelExplainer`, and for tree-based models (`dt`, `rf`, `xgb`, `lgbm`), it uses `TreeExplainer`. Specifically, for the random forest (`rf`) model, it calculates the mean of SHAP values differently. Finally, it returns the importance of the calculated SHAP values in the form of a DataFrame. This process helps identify the features that most influence the model's predictions.

```python
def featimp(main_pipeline, model, x_test, random_state):
    transformed_x_test = main_pipeline.transform(x_test)

    kernel_list = ['knn', 'svm']
    tree_list = ['dt', 'rf', 'xgb', 'lgbm']
    if model in kernel_list:
        explainer = shap.KernelExplainer(main_pipeline['estimator'].best_estimator_.predict_proba, transformed_x_test)
        shap_values = explainer.shap_values(transformed_x_test)
    if model in tree_list:
        explainer = shap.TreeExplainer(main_pipeline['estimator'].best_estimator_)
        shap_values = explainer.shap_values(transformed_x_test, check_additivity=False)

    if model=='rf':
        shap_values_mean = np.abs(shap_values).mean(axis=0)
        vals = shap_values_mean.mean(axis=1)
    else:
        vals = np.abs(shap_values).mean(0)

    shap_importance = pd.DataFrame([vals], index=['shap_value'+ str(random_state)], columns=main_pipeline.transform(x_test).columns)

    return shap_importance
```

The `export_to_csv` function takes the `file_lists` argument, iterates over it, and saves each element's DataFrame (`df`) as a CSV file. If the file name (`name`) does not already exist, it creates a new file and saves the data. If the file already exists, it appends the data to the existing file. In this case, the index of the new data is preserved, and the header is included only during the first save.

```python
def export_to_csv(file_lists):
    for df, name in file_lists:
        if not os.path.exists(name):
            df.to_csv(name, index=True, mode='w', encoding='utf-8-sig')
        else:
            df.to_csv(name, index=True, mode='a', encoding='utf-8-sig', header=False)
```

This function loads the dataset for a specified disease number, trains a selected machine learning model, and exports the performance evaluation scores and feature importance to CSV files. The user must provide the model name (`model`), disease number (`disease_no`), and the file names to save the performance scores and feature importance (`score_file_name`, `featimp_file_name`). The function first sets the seed for the random number generator and loads the dataset. Then, it constructs the pipeline corresponding to the selected model from the supported model list (`knn`, `svm`, `dt`, `rf`, `xgb`, `lgbm`). Depending on whether the model is kernel-based (`knn`, `svm`) or tree-based (`dt`, `rf`, `xgb`, `lgbm`), it applies the appropriate training method. Finally, it calculates the performance evaluation scores and feature importance based on the prediction probabilities for the test dataset and exports them to CSV files.

```python
def train_sklearn(disease_no, model, score_file_name, featimp_file_name, random_state=0):
    np.random.seed(random_state)

    x_train, x_test, y_train, y_test = load_dataset(disease_no = disease_no)

    pipeline_dict = {
        'knn': knn_pipeline(),
        'svm': svm_pipeline(),
        'dt': dt_pipeline(),
        'rf': rf_pipeline(),
        'xgb': xgb_pipeline(),
        'lgbm': lgbm_pipeline()
    }

    kernel_list = ['knn', 'svm']
    tree_list = ['dt', 'rf', 'xgb', 'lgbm']

    if model not in pipeline_dict:
        raise ValueError(f"Invalid model name '{model}' provided. Model name must be one of {list(pipeline_dict.keys())}.")

    main_pipeline = pipeline_dict[model]



    if model in kernel_list:
        main_pipeline.fit(x_train, y_train)
    if model in tree_list:
        weight = class_weight.compute_sample_weight('balanced', y_train)
        main_pipeline.fit(x_train, y_train, **{'estimator__sample_weight': weight})

    y_predproba = main_pipeline.predict_proba(x_test)

    score_df = scoring(y_test, y_predproba, random_state)
    featimp_df = featimp(main_pipeline, model, x_test, random_state) 

    export_to_csv([(score_df, score_file_name), (featimp_df, featimp_file_name)])
```

This function takes a specific disease number (`disease_no`), model (`model`), save path (`path`), and a list of random states (`random_state_list`) as arguments. It creates a directory at the specified path and performs model training for each random state. During the training process, it uses the disease number and model name to generate evaluation score files (`score_file_name`) and feature importance files (`featimp_file_name`). This process is repeated for the given list of random states.

```python
def main(disease_no, model, path = './yourpath', random_state_list = range(0, 10)):
    os.makedirs(path, exist_ok=True)

    for random_state in random_state_list:
        print("Current training: Disease {}, Model: {}, random state: {}".format(disease_no, model, random_state))
        train_sklearn(disease_no, model, score_file_name=path + '/' + str(disease_no) + '_' + str(model) + '_eval.csv', featimp_file_name=path + '/' + str(disease_no) +  '_' + str(model) + '_featimp.csv', random_state=random_state)
```

This function uses several machine learning models defined in `sklearn_model_list` and repeatedly calls the `main` function with numbers from 0 to 9 as parameters for each model. When calling, it passes the model name and data path to the `main` function. This process can be used to experimentally apply various parameters during data analysis or model training.

```python
sklearn_model_list = ['knn', 'dt', 'svm', 'rf', 'xgb', 'lgbm']

for model in sklearn_model_list:
    for num in range(0, 10):
        main(num, model = model, path='./yourpath')
```

_Comments and explanations about the code was generated with gpt-4-turbo_
