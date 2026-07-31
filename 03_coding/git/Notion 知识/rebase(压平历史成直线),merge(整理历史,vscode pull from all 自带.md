---
title: "rebase(压平历史成直线),merge(整理历史,vscode pull from all 自带),revert(撤回push),reset（撤回本地commit）"
publish: false
tags: ["Git"]
---
# rebase(压平历史成直线),merge(整理历史,vscode pull from all 自带),revert(撤回push),reset（撤回本地commit）

> `rebase` 的核心思想是：“把一组提交 **‘搬家’** 到新的基准(commit) 上，重新生成新的历史”。常见场景有两类：
> 

### A. **同步主分支** —— 让你当前分支排在最新 `main` 后面

```bash
# 在你创建 PR 之前，团队很可能建议你执行

git checkout feature/#2_...
git fetch origin
git rebase origin/main
```

流程示意：

```

main:    A---B---C  (远端最新)
feature:        D---E
# rebase 之后
feature:        A---B---C---D'--E'

```

- 你的 `D`、`E` 会被“复制”成 `D'`、`E'`，看起来像是刚刚提交的。
- 历史线性，避免 merge 叉路。

### B. **交互式 rebase（历史整理）** —— 改 message / squash / 删除提交

```bash
git rebase -i HEAD~5      # 对最近 5 条提交动手

```

出现的编辑面板里你可以：

- `reword` → 改 message
- `edit` → 改代码
- `squash` → 把多条合成一条
- `drop` → 删掉某条提交

这一步会**重写 commit hash**，所以如果已经 push 过，需要强推。

它的目的不是“找到最近一次 push 然后删除”，而是**在本地把历史整理好再（强）推上去**。

---

> git merge 是将两个不同分支的提交历史合并，并生成一个新的 合并提交（merge commit）。
> 

不会重写任何已有的提交，而是保留双方的提交历史结构。

！！！ 相当于是VS Code 插件中的【Pull from all remotes】

---

### 🧠 合并的思路（核心逻辑）：

以两个分支为例：

```

      A---B---C  ← main
           \
            D---E ← feature

```

你在 `main` 上执行：

```bash

git merge feature

```

Git 会：

1. 找出两个分支的最近共同祖先（这里是 B）
2. 把从 B → C 和 B → E 的两条路径合并
3. 生成一个新的合并提交（假设是 M）

合并后的结构变为：

```

      A---B---C
           \   \
            D---E---M  ← main

```

---

## ✅ `merge` vs `rebase` 的区别总结

| 比较点 | `merge` | `rebase` |
| --- | --- | --- |
| 历史结构 | 分叉+合并（保留原样） | 线性（把你的提交“搬”到别人的后面） |
| 是否产生新提交 | 是（合并提交） | 是（每个旧提交都会生成新 hash） |
| 是否改历史 | ❌ 不改历史 | ✅ 会改提交 hash |
| 是否会冲突 | 有可能 | 有可能（相同冲突可能出现多次） |
| 是否容易看出历史发展过程 | ✅（保留真实分支结构） | ❌（历史看起来像“后来才加进去”） |
| 推荐场景 | 多人协作主干合并、正式发布 | 自己整理提交、清理历史、保持线性分支 |

---

> git revert 是用来 “撤销某次提交的内容”，并生成一条新的提交，来达到“回滚”的效果。它不会删除或修改原来的 commit，而是“反着再做一遍”。
> 

| 操作 | 思想 | 作用 |
| --- | --- | --- |
| `revert` | **保留历史、不动原 commit**，但逻辑上“取消”了它 | 适合多人协作、已 push 的情况 |
| `amend` | **直接修改原 commit（rewrite）** | 适合没 push 的情况 |
| `rebase` | **重写历史，删改、重排都可以** | 适合整理多个 commit 的情况 |

### `revert` 会干嘛？

假设你 commit 了一次加法：

```diff
+ print("Hello world")

```

那你执行：

```bash
git revert head
```

Git 会生成一个新 commit，其 diff 是：

```diff
- print("Hello world")
```

等于“把你那次加进去的删了”，实现**功能层面撤销**，但**历史上仍然保留原来那次 commit 的存在**。

---

**撤销提交但保留更改到暂存区**
`git reset --soft HEAD~1`

---

### 比较历史

git rev-list --left-right --count feature/#43_extract_from_room_table...feature/#39OCRResultanaly
sis
