---
title: "git workflow"
publish: false
tags: ["Git"]
---
# git workflow

## 1 分支战略（Branch Strategy）

| 分支 | 作用 | 备注 |
| --- | --- | --- |
| **main / master** | 持续可部署、永远保持绿色 (CI 通过) | 不在此直接开发 |
| **feature/#issueId_描述** | 每个功能或修复各自的工作分支 | 例：`feature/#2_change_output_name` |
| **hotfix/***（可选） | 线上紧急修补 | 从 main 切出，修完后马上合回 |

**原则**

1. 所有开发都从最新 main 切分支。
2. 一项任务一个分支，分支名与 issue 一一对应，便于追踪。
3. 合并回 main 必须通过 Pull Request 并经 CI + Code Review。

---

## 2 日常开发完整流程

1. **同步主分支**
    
    ```bash
    git checkout main
    git pull --ff-only
    
    ```
    
2. **创建功能分支**
    
    ```bash
    
    git checkout -b feature/#24_table_analyse_seizou
    ```
    
3. **小步提交**
    - 用 `git add` 精确控制 **staging area**，一类变更一个 commit。
    - 提交信息用英文祈使句，格式参考 Conventional Commits：
        
        ```
        feat(ui): add login button
        refactor: extract auth helper
        fix: prevent empty username
        
        ```
        
4. **本地清理**
    - **修改最后一次** → `git commit --amend`（仅限尚未 push）
    - **整理多次提交** → `git rebase -i main` 或 `git rebase -i HEAD~N`
        - `reword` 改 message
        - `edit` 改内容
        - `squash`/`fixup` 合并
5. **推送分支**
    
    ```bash
    git push -u origin feature/#123_add_login_ui
    ```
    
6. **创建 Pull Request**
    - 目标分支：`main`
    - **标题**：`feat: #123 ログインUI追加`
    - **描述**：问题背景、解决方案、测试方法、风险点
    - 关联 issue：在描述里写 `Closes #123`，合并后自动关单
    - 添加 Reviewer / Assignee；确保 CI 状态检查通过
7. **在 PR 中修改**
    - **继续开发** → 普通 `git add` + `git commit` + `git push`，PR 自动更新
    - **想把新改动并入上一个未 review 的 commit** →
        
        ```bash
        git add .
        git commit --amend
        git push --force-with-lease
        
        ```
        
8. **代码评审通过后合并**
    - 推荐 **Squash and merge**：保留一条干净记录
    - 合并即自动部署 / 标记版本（可在 CI 配置）
9. **删除远程分支**（GitHub 按钮或）
    
    ```bash
    bash
    复制编辑
    git push origin --delete feature/#123_add_login_ui
    
    ```
    

---

## 3 常见场景决策表

| 需求 | 推荐命令 | 备注 |
| --- | --- | --- |
| 补充/修改 **未 push** 的最后一次提交 | `git add …` → `git commit --amend` | 不改历史 |
| 补充 **已 push** 的最后一次提交 | 同上 + `git push --force-with-lease` | 通知协作者 |
| 撤销已 push 的 bug 提交，但保留历史 | `git revert <hash>` → `git push` | 生成反向提交 |
| 撤销本地提交，并重写历史 | `git reset --soft/mixed/hard` | 仅限未 push |
| 整理多条历史，改 message / squash | `git rebase -i HEAD~N` | push 后需强推 |
| 主分支保持线性记录 | `git pull --rebase origin main` | 或在 PR 中 squash |

---

## 4 Pull Request 使用要点

1. **让改动易审阅**
    - 保持 PR 小而集中：< ~400 行 diff / < ~5 文件
    - 描述Why/What/How，而不是“修改了 bug”
2. **CI & 必要检查**
    - 设定必过的测试、lint、编译检查
    - 针对生产分支可启用保护：不通过检查不能合并
3. **代码评审**
    - Reviewer 专注逻辑与设计；可用行内评论交流实现细节
    - 作者响应后 **Resolve conversation**，保持讨论面板整洁
4. **合并策略**
    - 小团队建议 **Squash merge**：PR → 1 commit，历史简洁
    - 需要保留完整 commit 则用 **Merge commit**
5. **合并后动作**
    - 自动删除源分支
    - 自动部署 / 打 tag / 发布 Release Note
- 以上建议大多来自 GitHub 官方“让你的 Pull Request 易于审阅”指南[docs.github.com](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/getting-started/helping-others-review-your-changes?utm_source=chatgpt.com) 和“About pull requests”文档[docs.github.com](https://docs.github.com/articles/about-pull-requests?utm_source=chatgpt.com)。

---

## 5 命令速查（Cheat Sheet）

```bash
bash
复制编辑
# 配置
git config --global user.name "huang"
git config --global user.email "pbhuangedu@gmail.com"

# 分支
git checkout -b feature/#123_fix_bug     # 创建并切换
git branch -d <branch>                   # 删除本地
git push -u origin <branch>              # 推远程并建立 tracking
git push origin --delete <branch>        # 删远程

# 提交
git add <files>                          # 暂存
git commit -m "feat: add x"              # 提交
git commit --amend                       # 修改最后一次
git rebase -i HEAD~3                     # 交互式整理
git revert <hash>                        # 安全回滚

# 推拉
git pull --rebase                        # 拉取并重放
git push --force-with-lease              # 更安全的强推

```

---

### 👋 使用建议

- **一任务一分支，一功能一提交**：让审阅者读得懂，也便于回滚。
- **本地先整理，远程保持干净**：推前用 `rebase -i` 压平历史，改错用 `amend`。
- **线上出错先 `revert`，后补正式修复**：不要慌忙强推改历史。
- **PR 是协作核心**：写好描述，充分利用 Reviewer、CI、Status Check，确保主干永远可部署。
