---
title: "python求两个文件夹下的同名文件的数据的相关系数"
publish: false
tags: ["Python"]
---
# python求两个文件夹下的同名文件的数据的相关系数

```python
import matplotlib.pyplot as plt
import scipy.io
from pathlib import Path
import numpy as np
import pandas as pd
```

```python
processedpy_path = Path("processedpy")
#porting_results_path = Path("porting_results")
porting_results_path = Path(r"C:\Users\huang\Documents\20240724\mazda_cochlear_model\outputs\result")

processedpy_folders = {folder.name for folder in processedpy_path.iterdir() if folder.is_dir()}
porting_results_folders = {folder.name for folder in porting_results_path.iterdir() if folder.is_dir()}
common_folders = processedpy_folders & porting_results_folders

for folder_name in common_folders:
    np_folder_path = processedpy_path / folder_name
    mat_folder_path = porting_results_path / folder_name

    np_files = [f.stem for f in np_folder_path.glob("*.np")]
    
    for file_stem in np_files:
        np_file_path = np_folder_path / f"{file_stem}.np"
        mat_file_path = mat_folder_path / f"{file_stem}.mat"
        
        if not mat_file_path.exists():
            print(f"Warning: {mat_file_path} does not exist.")
            continue
        
        if (file_stem=='E1'or'E2'or'F1'or'F2'):
            newshape=(-1, 1)
        elif (file_stem=='v1'or'v2'or'y1'or'y2'):
            newshape=(-1, 156)
    
        mat_data = scipy.io.loadmat(mat_file_path)
        mat_key = list(mat_data.keys())[-1]
        mat_data = np.reshape(mat_data[mat_key], newshape)
        mat_df = pd.DataFrame(mat_data)   
        
        np_data = np.fromfile(np_file_path)
        np_data = np.reshape(np_data, newshape)
        np_df = pd.DataFrame(np_data) 
        
        correlation = np.corrcoef(mat_df.values.flatten(), np_df.values.flatten())[0, 1]
        print(f"Correlation between {folder_name}{file_stem}.mat and {folder_name}{file_stem}.np: {correlation}")

        # plt.figure(figsize=(8, 6))
        # plt.scatter(mat_df.values.flatten(), np_df.values.flatten(), alpha=0.5)
        # plt.title(f'Scatter Plot of {file_stem}.mat and {file_stem}.np')
        # plt.xlabel('MAT Data')
        # plt.ylabel('NP Data')
        # plt.grid(True)
        # plt.show()
        break
```
