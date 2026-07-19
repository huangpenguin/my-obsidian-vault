
---
tags:
  - 技术/软件工程
  - 技术/网络/SSH
  - 技术/工具/Git
---

# SSH 多密钥与多账号配置速查

## 1. 密钥生成与管理

**生成密钥**
指定加密方式、邮箱注释及输出路径：

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_github_personal
ssh-keygen -t rsa -b 4096 -C "work_email@example.com" -f ~/.ssh/id_rsa_github_work
ssh-keygen -t rsa -b 4096 -C "remote_user@example.com" -f ~/.ssh/id_rsa_remote
```

**权限设置** (必须，否则 SSH 会拒绝连接)

```
chmod 600 ~/.ssh/id_rsa_*
```

**SSH-Agent 托管** (避免频繁输密码)

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa_github_personal
ssh-add ~/.ssh/id_rsa_github_work
ssh-add ~/.ssh/id_rsa_remote
```

## 2. 核心配置 (~/.ssh/config)

编辑 `~/.ssh/config`，通过 `Host` 别名实现路由分发。

> [!note] 关键参数说明 `IdentitiesOnly yes`：强制只使用 IdentityFile 指定的私钥，防止多密钥冲突。


```
# ==========================================
# 多平台配置示例
# ==========================================

# GitLab 默认账号
Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_rsa_gitlab
  IdentitiesOnly yes

# 远程服务器 (VSCode Remote-SSH 可直接读取此别名)
Host remote-pc
  HostName remote.example.com
  User your_remote_username
  Port 22
  IdentityFile ~/.ssh/id_rsa_remote
  IdentitiesOnly yes

# ==========================================
# 同平台多账号配置示例 (以 GitHub 为例)
# ==========================================

# GitHub - 个人账号
Host github.com-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_github_personal
  IdentitiesOnly yes

# GitHub - 公司账号
Host github.com-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_github_work
  IdentitiesOnly yes
```

## 3. 部署与测试

**获取公钥内容**

```
cat ~/.ssh/id_rsa_github_personal.pub
# 复制输出内容，添加到 GitHub/GitLab 设置页面，或远程服务器的 ~/.ssh/authorized_keys
```

**连通性测试**

```
ssh -T git@github.com-personal
ssh -T git@gitlab.com
ssh remote-pc
```

## 4. Git 仓库克隆与局部身份

**使用自定义 Host 克隆仓库** 地址中的域名必须替换为 `~/.ssh/config` 中配置的 `Host` 别名。


```
# 个人项目
git clone git@github.com-personal:username/personal-repo.git

# 公司项目
git clone git@github.com-work:company/work-repo.git
```

**配置局部身份信息** 进入具体项目目录后，取消全局身份，配置该项目专属的用户名和邮箱：

```
git config user.name "Your Name"
git config user.email "your_email@example.com"
```