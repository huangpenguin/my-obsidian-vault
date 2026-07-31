---
title: "use Android Studio jdk for system using of java"
publish: false
tags: ["Android"]
---
# use Android Studio jdk for system using of java

### 设置`JAVA_HOME`到`C:\Program Files\Android\Android Studio\jbr`

1. **设置`JAVA_HOME`环境变量**：
    - 右键点击**此电脑**（或者**我的电脑**），选择**属性**。
    - 点击**高级系统设置**，然后点击**环境变量**。
    - 在**系统变量**区域，点击**新建**，输入以下内容：
        - **变量名**：`JAVA_HO`
        - **变量值**：`C:\Program Files\Android\Android Studio\jbr`
2. **添加到`Path`变量**：
    - 在**系统变量**中找到`Path`，点击**编辑**。
    - 点击**新建**，然后输入以下内容：
        
        ```perl
        %JAVA_HOME%\bin
        
        ```
        
    - 点击**确定**保存所有更改。
3. **验证配置是否成功**：
    - 打开新的命令提示符窗口（务必使用新开的命令提示符），输入以下命令：
        
        ```bash
        java -version
        
        ```
        
    - 如果显示类似以下信息，说明配置成功：
        
        ```scss
        scss
        复制代码
        java version "17.0.x"
        Java(TM) SE Runtime Environment (build 17.0.x)
        Java HotSpot(TM) 64-Bit Server VM (build 17.0.x, mixed mode)
        
        ```
        
4. **重新运行Gradle命令**：
    - 返回到Android Studio的终端，或者在命令提示符中，进入你的项目目录，运行以下命令：
        
        ```bash
        bash
        复制代码
        ./gradlew clean
        
        ```
        
    - 这次应该可以正常找到Java并执行命令。
