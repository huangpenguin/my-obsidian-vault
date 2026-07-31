---
title: "cd,cp,mv,rsync"
publish: false
tags: ["Linux"]
---
# cd,cp,mv,resync

| 命令 | `/path/dir` | `/path/dir/` | `/path/dir/.` |
| --- | --- | --- | --- |
| `cd` | 进入目录 `dir` | 进入目录 `dir` | 等价于 `/path/dir/` |
| `cp -r` | 复制整个目录 `dir`（会包含 `dir`） | 行为同上（`cp` 对尾部 `/` 不敏感） | 只复制 `dir` 内的内容 |
| `rsync -a` | 复制整个目录 `dir`（包含目录名） | 只复制目录内的内容（**敏感！**） | 和 `/path/dir/` 等价 |

---

- **`mv`**: 移动或重命名文件/目录。
    - `mv old_name new_name`: 重命名。
    - `mv file /path/to/new_location`: 移动文件。

---

- **`cd`**: 改变目录。
    - `cd /path/to/directory`: 进入指定目录。
    - `cd ..`: 返回上一级目录。
    - `cd ~`: 返回当前用户的主目录。

---

- **`cp`**: 复制文件或目录。
    - `cp source_file destination_file`: 复制文件。
    - `cp -r source_folder destination_folder`: 递归复制目录。
        - sudo cp -r /datadrive/trws.backup/workspace/. /datadrive/workspace/
        - cp -r /datadrive/workspace/2021.aisin-tray-recognition/pretrained_models /mnt/home/huang/pretrained_models

---

- **`rsync`**: 对于大文件或目录的复制，**`rsync` 是一个更强大和常用的工具**，它默认就支持显示进度条，并且在中断后可以恢复，比 `cp` 更适合这种场景。
- sudo rsync -ah --info=progress2 /datadrive/trws.backup/workspace/ /datadrive/workspace/
- sudo rsync -ah --info=progress2 /datadrive/trws.backup/workspace/2021.aisin-tray-recognition /datadrive/workspace/

### 1. **本地目录同步（保持结构和权限）**

```bash

rsync -avh /src/dir/ /dst/dir/

```

- `v`：显示详细输出。
- 注意末尾的 `/`：表示复制目录内容（不含目录本身）。

---

### 2. **通过 SSH 远程同步文件**

```bash

rsync -avz -e ssh user@remote:/path/to/data/ /local/dir/

```

- `z`：压缩数据传输，加快远程传输。
- `e ssh`：指定使用 SSH 协议。
- 远程目录要以冒号 `:` 开头。

---

### 3. **同步时删除目标中源已删除的文件**

```bash

rsync -avh --delete /src/dir/ /dst/dir/

```

- `-delete`：删除目标目录中，源目录已不存在的文件（保持完全同步）。

---

### 4. **只同步大于某大小的文件**

```bash

rsync -avh --min-size=10M /src/ /dst/

```

- 只同步大于 10MB 的文件。

---

### 5. **同步指定类型文件**

```bash

rsync -avh --include="*/" --include="*.jpg" --exclude="*" /src/ /dst/

rsync -avh --include="*/" --include="*.png" --include="*.mp4" --exclude="*"
```

- 只同步 `.jpg` 文件，保留目录结构。
- 注意：`rsync` 的 include/exclude 是**按顺序匹配规则的**，一旦某个文件/目录被匹配，就不会继续匹配后面的规则了。因此 `--include="*/"` 必须在最前面，否则不会进入子目录。

---

### 6.实现文件夹更名

使用 `mv` 命令把 `dataset` 文件夹重命名为 `datasets`，而不影响它里面的内容。

---

### ✅ 指令如下：

```bash

mv /home/user/workspace/2021.aisin-tray-recognition/dataset /home/user/workspace/2021.aisin-tray-recognition/datasets
```

这条命令：

- 只是修改文件夹名 `dataset → datasets`
- **不会更改文件夹中的任何内容或权限**
- 执行后原路径就不存在了，所有文件都在新路径下

# 路径斜杠 `/` 行为与路径解析对比：

---

## 1. 基础概念

- **路径末尾带斜杠**：通常表示“目录内容”或“目录本身”
- **路径末尾不带斜杠**：通常表示“目录或文件本身”，具体看命令语义

---

## 2. `mv` 的行为

- **目标是目录**，`mv src dst/`：把 `src`（文件或目录）移动到 `dst/` 目录下，结果是 `dst/src`
- **目标不存在**，`mv src dst`：把 `src` 重命名为 `dst`（不管 `dst` 是文件名还是目录名）

**斜杠对 `mv` 源路径无影响**，主要看目标：

示例：

```bash
mv folder1 folder2/
```

- `folder2/` 存在且是目录：移动 `folder1` 到 `folder2/folder1`

```bash
mv folder1 folder2
```

- `folder2` 不存在：重命名 `folder1` 为 `folder2`

---

## 3. `cp` 的行为

- **源路径末尾带斜杠（目录时）**：
    - `cp -r /src/ /dst/` ：复制 `/src` 目录下的内容到 `/dst/`（不包括 `/src` 目录本身）
- **源路径末尾不带斜杠（目录时）**：
    - `cp -r /src /dst/` ：复制整个 `/src` 目录到 `/dst/` 下，变成 `/dst/src`
- **目标目录**：
    - 如果 `/dst/` 存在且是目录，拷贝内容或目录放到其中
    - 如果目标不存在，行为会因命令不同有区别（但一般认为是新目录或文件）

---

## 4. `rsync` 的行为

- **源路径末尾带斜杠**：复制的是**目录下的内容**，不包括目录本身
- **源路径末尾不带斜杠**：复制的是整个目录（包括目录本身）

---

### 例子对比（假设 `/src` 目录下有文件 `a.txt`）

| 命令 | 作用 | 目标结果 |
| --- | --- | --- |
| `cp -r /src /dst` | 复制整个 `src` 目录 | `/dst/src/a.txt` |
| `cp -r /src/ /dst` | 复制 `src` 目录内容 | `/dst/a.txt` |
| `rsync -av /src /dst` | 同上，复制整个目录 | `/dst/src/a.txt` |
| `rsync -av /src/ /dst` | 复制目录内容 | `/dst/a.txt` |
| `mv /src /dst` | 移动并重命名（如果 `/dst` 不存在） | `/dst`（原 `/src`） |
| `mv /src /dst/` | 移动到目录 | `/dst/src` |

---

## 5. 视觉总结（示意图）

```

/src/
  ├── a.txt

执行：

cp -r /src /dst  => /dst/src/a.txt
cp -r /src/ /dst => /dst/a.txt

rsync -av /src /dst  => /dst/src/a.txt
rsync -av /src/ /dst => /dst/a.txt

mv /src /dst       => 如果 /dst 是目录，变成 /dst/src；如果不是，重命名为 /dst
mv /src /dst/      => 移动到 /dst 目录下，变成 /dst/src

```

---

## 6. 重点提示

- **`rsync` 源路径末尾斜杠非常重要，容易出错**
- **`cp` 也区分是否复制目录本身还是目录内容**
- **`mv` 更简单，主要看目标路径是目录还是文件名**
