---
title: "github PR"
publish: false
tags: ["Git"]
---
# github PR

## ✅ GitHub 的三种 Merge 策略对比

当一个 PR 被批准后，你可以点击「Merge pull request」按钮，但 GitHub 会提供三种合并方式：

| 类型 | 名称 | 特点 | 是否保留原 commit 记录 |
| --- | --- | --- | --- |
| ✅ 默认 | **Create a merge commit** | 保留完整提交历史，产生一个合并节点 | ✅ 保留全部 |
| ✅ 推荐 | **Squash and merge** | 把所有提交压缩成一条 | ❌ 只保留 1 条 |
| 🚫 特殊 | **Rebase and merge** | 把每条 commit 重新 replay 到目标分支上，历史更线性 | ✅ 保留全部但修改 hash |

---

## 📘 各种方式详细解释

---

### 1. 🟢 **Create a merge commit（合并提交）**

- 会产生一个新的 merge commit，例如：
    
    ```
    *   Merge pull request #123 from feature/add-login
    |\
    | * feat: add login button
    | * fix: remove unused var
    * |
    |/
    * main branch continues...
    ```
    
- 所有的 commit 都保留下来
- merge commit 提供了明确的“PR合并点”

📌 **适合：**

- 多人合作，想保留每个成员的提交痕迹
- 公司开发，保留审计追踪

---

### 2. 🟣 **Squash and merge（压缩合并）**

- 把整个 PR 中的所有 commit 合并为**一条新的提交**
- 提交信息可以从 PR 标题自动生成或自定义
- 整体看起来更简洁，主分支提交记录更线性

🔧 例如：

```
* feat: ユーザー登録APIを追加（PR #123）
* main branch continues...

```

📌 **适合：**

- 自己分支开发时 commit 很多但不重要（WIP、debug log 等）
- 想让 `main` 分支记录只包含“最终结果”

📝 **小技巧：**

PR 内用多个 commit 测试/开发 → 合并时 squash → 最后只留下「干净」的记录

---

### 3. 🟡 **Rebase and merge（变基合并）**

- 将 PR 中的每个 commit 直接 replay 到 `main` 的最新状态上
- 不会生成 merge commit
- 历史更线性，但 commit 的 hash 会被重写

🔧 例如：

```
* feat: add login UI
* fix: login edge case
* main branch continues...

```

📌 **适合：**

- 想保留每个 commit，但不想看到 merge commit
- 项目管理者对提交顺序非常讲究

⚠️ 注意：

- 因为 commit 会变，作者签名可能变，容易出冲突
- 不建议对外部贡献者使用（可能引发历史冲突）

---

## 🧠 总结：合并方式选哪个？

| 你的需求 | 推荐策略 |
| --- | --- |
| 保留全部 commit、多人协作 | **Merge commit** |
| 历史干净，commit 越少越好 | **Squash and merge** ✅ 推荐 |
| 保留 commit、历史线性 | **Rebase and merge**（慎用） |

---

## ✅ 实际项目建议

- **个人项目或功能开发分支：**
    
    ➤ 本地 commit 多 → 用 `Squash and merge` 整理干净后合并
    
- **团队合作，多个贡献者：**
    
    ➤ 默认用 `Merge commit`，保留所有人的贡献痕迹
    
- **开源项目（多人 PR）：**
    
    ➤ 统一用 `Squash and merge`，由 maintainer 决定最终提交信息
