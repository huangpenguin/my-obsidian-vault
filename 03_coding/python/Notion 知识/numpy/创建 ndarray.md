---
title: "创建 ndarray"
publish: false
tags: ["Python"]
---
# 创建 ndarray

- 从列表创建：
    
    ```
    import numpy as np
    arr = np.array([1, 2, 3, 4])
    print(arr)  # 输出: [1 2 3 4]
    ```
    
- 创建全零数组：
    
    ```
    zeros_arr = np.zeros((2, 3))  # 2行3列的全零数组
    print(zeros_arr)
    ```
    
- 创建全一数组：
    
    ```
    ones_arr = np.ones((3, 2))  # 3行2列的全一数组
    print(ones_arr)
    ```
    
- 创建单位矩阵：
    
    ```
    eye_arr = np.eye(3)  # 3x3的单位矩阵
    print(eye_arr)
    ```
    
- 创建等差数列：(**array range** 的缩写，表示“生成一个基于范围的数组)
    
    ```
    range_arr = np.arange(0, 10, 2)  # 从0到10，步长为2
    print(range_arr)  # 输出: [0 2 4 6 8]
    ```
    
- 创建等间隔数组：(linear space)
    
    ```
    linspace_arr = np.linspace(0, 1, 5)  # 从0到1，生成5个等间隔的数
    print(linspace_arr)  # 输出: [0.   0.25 0.5  0.75 1.  ]
    ```
