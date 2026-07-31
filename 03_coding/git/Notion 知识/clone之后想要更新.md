---
title: "clone之后想要更新"
publish: false
tags: ["Git"]
---
# clone之后想要更新

在你已经进入到项目目录（`pjt_aks`）之后，只要本地没做过什么重大的改动，就可以直接从远程拉取最新提交：

1. **查看当前分支**（确认你在哪个分支上）
    
    ```bash
    git branch
    ```
    
    输出会在当前分支前有个 `*`，比如 `* main` 或 `* master`。
    
2. **拉取远程更新**
    
    如果你的主分支是 `main`，执行：
    
    ```bash
    
    git pull origin main
    ```
    
    或者更通用地，直接：
    
    ```bash
    
    git pull
    ```
    
    这会把远端同名分支的最新提交合并到你当前分支。
    
3. **处理冲突（如有）**
    
    如果有本地改动且与远程冲突，Git 会提示你冲突文件。手动编辑这些文件解决冲突，然后：
    
    ```bash
    
    git add <冲突文件>
    git commit
    ```
    
4. **（可选）使用 Rebase 保持线性历史**
    
    ```bash
    
    git pull --rebase
    ```
    
    这样你的本地提交会“插”在远程更新之后，提交记录更整洁。
    

---

### 如果你在本地做过未提交的改动

1. **临时保存改动**
    
    ```bash
    git stash
    ```
    
2. **拉取远程最新**
    
    ```bash
    git pull
    ```
    
3. **恢复改动**
    
    ```bash
    git stash pop
    ```
    
    如有冲突，同样按上面的方法解决。
    

---

### 如果远程默认分支不是 `main`

先查看远程默认分支名：

```bash
git remote show origin
```

命令输出里会有类似：

```
HEAD branch: master
```

这时就用 `git pull origin master`。

## 典型流程示例

假设你在 `main`（或 `master`）分支上工作。

```bash

# 1. 切到你要工作的分支
git checkout main

# 2. 拉取远程最新改动并合并到本地
git pull

#    如果你希望保持线性历史，可以用 rebase：
# git pull --rebase

# 3. 本地修改、测试，通过后
#    编辑、添加文件...
git add <file1> <file2>
git commit -m "feat: 增加 XXX 功能"

# 4. 再次同步远程改动（防止在你本地编辑期间别人也 push 了）
git pull

#    若出现冲突，手动解决 -> git add 冲突文件 -> git commit

# 5. 最后把本地提交推送到远程
git push origin main
git push origin master
```

如果你要在一个 feature 分支上工作，也可：

```bash

git checkout -b feature/your-feature
# 工作完成后
git pull --rebase origin main   # 同步主分支最新
git push -u origin feature/your-feature

```
