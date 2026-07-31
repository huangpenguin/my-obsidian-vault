---
title: "为什么远程仓库默认叫 origin？"
publish: false
tags: ["Git"]
---
# 为什么远程仓库默认叫 origin？

```markdown
#通过查看对应关系可以看到远程仓库的默认名字是origin

git branch -vv
```

当你运行这类命令时：

```bash

git clone https://github.com/user/myrepo.git

```

Git 会自动帮你：

1. 把 GitHub 上的这个仓库下载（克隆）下来
2. 并给它起个**远程别名**叫 `origin`

也就是说：

```
ini
复制编辑
origin = 你克隆的那个 GitHub 仓库的“代号”

```

你可以完全改成别的名字，例如：

```bash
bash
复制编辑
git remote rename origin github

```

然后你以后就得这样写：

```bash
bash
复制编辑
git push github master

```

所以说，**`origin` 不是关键字，它只是一个习惯上的默认名字**，完全可以换。

---

## 🧭 举个例子理解一下：

### 情况：你从 GitHub 克隆了一个仓库

```bash
bash
复制编辑
git clone https://github.com/yourname/awesome-project.git

```

Git 做了两件事：

- 把代码拉下来（到一个叫 `awesome-project` 的文件夹）
- 给这个远程仓库起名叫 `origin`

你可以查看：

```bash
bash
复制编辑
git remote -v

```

输出：

```
perl
复制编辑
origin  https://github.com/yourname/awesome-project.git (fetch)
origin  https://github.com/yourname/awesome-project.git (push)

```

---

## ✅ 那我能不能不用叫 `origin`？可以！

你自己手动加远程仓库时，完全可以用别的名字：

```bash
bash
复制编辑
git remote add github https://github.com/yourname/myrepo.git
git push github master

```

比如你也可以加多个：

```bash
bash
复制编辑
git remote add backup git@192.168.1.3:repos/project.git
git push backup master

```

---

## ✅ 总结一句话：

| 名称 | 含义 |
| --- | --- |
| `origin` | 只是默认的远程仓库名字，等价于“GitHub 那个仓库”，可以改成别的 |
| `git push origin master` | 把 `master` 分支推送到名为 `origin` 的远程仓库 |
| `origin` 可以改名 | 用 `git remote rename` 或 `git remote add` 自定义 |
