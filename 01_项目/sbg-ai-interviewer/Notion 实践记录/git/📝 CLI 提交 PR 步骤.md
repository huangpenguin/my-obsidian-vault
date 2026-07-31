---
title: "📝 CLI 提交 PR 步骤"
publish: false
tags: ["Git"]
---
# 📝 CLI 提交 PR 步骤

### **第 1 步：提交本地改动**

```bash
cd /Users/huang.pengbin/MyScripts/chatbot/pjt-sbg

# 1. 检查改动（你已完成）
git status

# 2. 添加改动到暂存
git add .

# 3. 提交代码
git commit -m "feat: migrate to uv package manager and add GitHub templates"
```

### **第 2 步：推送到远程**

```bash
# 推送当前分支到远程
git push origin feature/llm-reasoning
```

### **第 3 步：用 CLI 创建 PR（需要 GitHub CLI）**

首先检查是否安装了 `gh` 工具：

```bash
which gh
```

如果没装，在 macOS 安装：

```bash
brew install gh
```

然后验证登录（首次需要）：

```bash
gh auth login
# 选择 GitHub.com
# 选择 HTTPS
# 选择 Paste authentication token (需要从 GitHub 网页创建个人令牌)
```

### **第 4 步：创建 PR**

**方式 A：交互式创建（推荐，更清晰）**

```bash
gh pr create --base dev --title "feat: English localization and deployment improvements" --body ""
```

这会打开编辑器让你填写 PR 内容。

**方式 B：直接从命令行**

```bash
gh pr create \\
  --base dev \\
  --title "feat: English localization and deployment improvements" \\
  --body "
## 背景・目的
完成项目的英文本地化和部署脚本改进

## 变更内容
- Translated all UI strings and comments to English
- Migrated to uv package manager with pyproject.toml
- Added GitHub workflow templates (.github/)
- Improved deploy.sh idempotency

## 风险・疑虑点
None

## 检查清单
- [x] PR标题符合格式
- [x] 模板已填写完整
- [x] make check通过
"
```

### **第 5 步：验证 PR 创建成功**

```bash
# 查看创建的 PR
gh pr view

# 或在浏览器中查看
gh pr view --web
```

---

## 🎯 快速命令总结

```bash
# 完整流程（一条条运行）
cd /Users/huang.pengbin/MyScripts/chatbot/pjt-sbg
git add .
git commit -m "feat: migrate to uv package manager and add GitHub templates"
git push origin feature/llm-reasoning

# 创建 PR（推荐方式）
gh pr create --base dev
# 然后在打开的编辑器中填写模板内容
```

---

## ⚠️ 注意事项

1. **PR 标题格式**：`(feat|refactor|bugfix|docs): description`
    - 你的应该是：`feat: migrate to uv and add GitHub workflow templates`
2. **模板填写**：按照 `.github/pull_request_template.md` 的各个部分填写，**全部使用英文**
3. **检查清单**：提交前必须确保：
    
    ```bash
    make check  # 必须通过
    ```
    
4. **如果还没安装 `gh` 工具**：
    
    ```bash
    brew install gh  # 一条命令搞定
    ```
    

---

## ❓ 需要帮助吗？

想要我帮你生成完整的 PR 描述文本吗？或者直接帮你创建？---

## ❓ 需要帮助吗？

想要我帮你生成完整的 PR 描述文本吗？或者直接帮你创建？
