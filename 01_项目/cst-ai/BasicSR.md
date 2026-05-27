## 一、BasicSR 文档里的几种方式（先搞懂它在说什么）

可以想成 「装修房子」 两种大路线：

### 路线 A：Git clone 整个 BasicSR 仓库

> 买下一整栋毛坯房（完整源码），在原来的结构上改。

- 你能看到 `basicsr/`、`options/`、`setup.py` 全部代码
- 适合：直接微调官方 repo、改模型、改 cfg、改训练脚本
- 你说「pull 下来微调」——大概率是这种

### 路线 B：`pip install basicsr`，自己另建一个项目

> 只买建材包（basicsr 库），在你自己的空地上盖新房。

- 你的仓库很干净，只写自己的 `models/`、`train_xxx.py`
- 适合：项目结构和 BasicSR 官方 repo 差异大、只想用它的训练框架

官方样例主要讲 路线 B。下面 B 又分两档：

|BasicSR 说法|形象理解|技术含义|
|---|---|---|
|简单模式|工地不登记户口，来了就能干|不 `setup.py develop`，靠 `PYTHONPATH` 或直接跑脚本|
|安装模式|给项目办户口（editable install）|`pip install -e .` / `setup.py develop`，`import yourpkg.xxx` 随便用|
## 路线 A/B 和两种模式，能任意组合成四种吗？

不完全是「任意四种」，要分开看两层：

### 第一层：BasicSR 官方的两条大路线（二选一）

||路线 A：clone 全仓库|路线 B：`pip install basicsr` + 自建项目|
|---|---|---|
|是什么|在 BasicSR 官方 repo 里改|你自己新建一个 repo，只依赖 basicsr 包|
|关系|互斥为主——微调官方代码通常选 A；从零搭自己的工程通常选 B||

### 第二层：「简单 / 安装模式」主要属于路线 B

|模式|适用|
|---|---|
|简单模式（不 `develop` install）|多为 路线 B 的小项目：脚本直接跑，`PYTHONPATH` 凑合用|
|安装模式（`pip install -e .`）|路线 A 几乎必做；路线 B 里你的项目也推荐这样做|

所以不是干净的 2×2=4 格，更接近：

路线 A（clone BasicSR）

└─ 实际就是「安装模式」：在 BasicSR 根目录 pip install -e .

（一般不会选 A + 简单模式）

路线 B（自建项目 + pip install basicsr）

├─ 简单模式：小 demo、结构 flat

└─ 安装模式：正式一点的项目

形象说：

- A = 进 BasicSR 原厂车间改产线 → 总要「办户口」（editable install）
- B = 自己开个小作坊，只买 basicsr 零件 → 可以「临时搭棚」（简单）或「办户口」（安装）

你 超分 + 去 X 光噪声、第一阶段试数据，选 路线 A + editable install 最合适；不必纠结四种组合。
---

## 二、你的情况：pull BasicSR 微调 → 走哪条？

推荐：路线 A（clone 官方 repo）+ BasicSR 的「安装模式」+ 你的模板「场景四」

原因：

- 微调通常要改 cfg、网络、数据集路径，需要完整源码
- BasicSR 自带 `setup.py`，editable install 后 `import basicsr` 和改内部模块都顺
- 你的模板对「只有 `requirements.txt`、没有 `pyproject.toml`」的项目，对应 README 场景四：先装依赖，再 `init-ai`，不会强行改它的依赖结构

---

## 三、推荐操作顺序（gpu01 或本机均可）
## 执行步骤（润色版）

### 1. 拉取代码

```bash
cd /home/huang/code/cst_ai
git clone https://github.com/XPixelGroup/BasicSR.git
cd BasicSR
```

### 2. 用 uv 创建虚拟环境（模板规范）

```bash
# 优先 3.11；若本机无该解释器，再 fallback 3.10 或系统默认
uv venv --python 3.11
source .venv/bin/activate
```

后续命令统一在 `BasicSR` 根目录、已 activate 的 venv 中执行。

### 3. 按 BasicSR upstream 方式安装依赖

**顺序很重要**：先解决 PyTorch，再装其余依赖，最后 editable 安装包本身。

```bash
# 当前机器无 GPU：先装 CPU 版 torch，避免拉 CUDA 大包且保证可 import
uv pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu

# upstream 依赖（requirements.txt 含 addict/lmdb/opencv 等）
uv pip install -r requirements.txt

# 等价于 python setup.py develop；注册 basicsr 包到 venv
uv pip install -e .
```

**可选（仅当后续训练 EDVR/StyleGAN2 等需 C++ 扩展时）**：

```bash
BASICSR_EXT=True uv pip install -e .
```

默认微调 SR 模型一般**不需要**编译扩展；首次部署跳过即可。

|会加上的|不会破坏的|
|---|---|
|`.cursor/rules/`、Ruff/Pyright 配置|BasicSR 原有 `setup.py`、`requirements.txt`、cfg|
|GitLab CI、Dockerfile（可选）|已有 Python 训练脚本（`train.py` 用 copy_if_missing）|
|`.cursor/project-context/`|你的 `options/*.yml` 训练配置|

因为没有 `pyproject.toml`，`init-ai` 不会自动 `uv add ruff`（见 inject-ai 逻辑）——这是刻意的，避免把 BasicSR 改成另一套包结构。

---

## 六、MLOps / GitLab 训练怎么接 BasicSR？

模板自带的 `train.py` 是 GPU smoke test，不是 BasicSR 训练入口。微调项目里你要改 GitLab CI 变量，例如：

TRAIN_COMMAND="python basicsr/train.py -opt options/train/MyModel.yml"

# 或你实际用的入口

在 gpu-server 上仍是：shell executor Runner → 宿主机 docker build → docker run 跑上述命令。


---

## 八、一句话建议

> 微调 BasicSR：clone 官方 repo → `uv venv` + `requirements.txt` + `uv pip install -e .` → `init-ai`（场景四）→ 在 project-context 和 GitLab `TRAIN_COMMAND` 里写明 BasicSR 的真实训练命令。  
> 不要对 BasicSR 先 `uv init` 覆盖结构；Simple 模式只适合临时跑一下，微调请用安装模式。

如果你告诉我：是 clone 官方 BasicSR 还是 自己新建 repo + pip install basicsr，以及训练命令大概是 `python basicsr/train.py -opt xxx.yml` 哪一种，我可以按你的实际路径写一版「复制就能跑」的命令清单（仍保持 Ask 模式，只给文本不改正文仓库）。


最贴合：路线 A（clone 整个 BasicSR）+ 安装模式（`pip install -e .`）+ 你的模板「场景四」

| 你的目标           | 对应做法                                              |
| -------------- | ------------------------------------------------- |
| 还不改网络结构        | 不用自建小仓库、也不用动 `basicsr` 源码                         |
| 只换数据、cfg、预训练权重 | 改 `options/train/*.yml` 里的 dataset / model / path |
| 先跑通训练、看指标/肉眼效果 | 用官方 `basicsr/train.py -opt xxx.yml`               |
| 加上 AI 规则 / CI  | clone 后 `init-ai`，不要先 `uv init` 改项                |

---

1. clone BasicSR
2. uv venv + requirements + pip install -e .
3. init-ai（场景四）
4. 复制一份官方 train yml → 只改：
   - 数据集路径 / lmdb / meta info
   - batch size、iter 数（先小规模 smoke）
   - 预训练模型 path（若有）
5. 本地或 gpu-server 短训（几千～几万 iter 先看趋势）
6. 看：loss 是否降、验证集 PSNR/SSIM、肉眼看几张 output


## 效果不好：换仓库还是自己动手改？

### 第 1 层：数据和配置（80% 的问题在这里）

| 检查项         | 常见坑                             |
| ----------- | ------------------------------- |
| 数据格式/配对     | LR-HR 是否对齐、路径是否错                |
| 分辨率 / scale | x2/x4 和数据是否一致                   |
| 归一化 / 色彩空间  | RGB vs Y 通道、范围 [0,1] vs [0,255] |
| cfg 是否匹配任务  | 超分 / 去噪 / 去模糊 选错 model          |
| 预训练权重       | 没用或 load 失败（其实从零训）              |
| 训练量         | iter 太少，还没收敛就判死刑                |

动作：换/修 yml、换 checkpoint、加长一点训练、固定 eval 几张图对比。仍用 BasicSR，不动代码结构。

### 第 2 层：换「现成方案」，仍不换仓库

BasicSR 里通常可以只改配置就试：

- 另一个 model 类型（如 RRDB、SwinIR 等，看文档 supported list）
- 另一个 官方 pretrained
- 更小的 patch / 更小的 batch（显存或稳定性）

动作：相当于「同一套训练框架，换一道官方菜谱」，不需要你会改网络。

### 第 3 层：才考虑换仓库 / 换框架

适合这些情况：

- 任务和 BasicSR 主场景差太远（例如纯分类、检测，不是 low-level vision）
- 数据类型特殊，BasicSR 数据 pipeline 很难接
- 官方 + 多种 cfg 都试过后，指标和视觉都完全不可用

动作：再评估 Real-ESRGAN fork、其他 restoration repo 等——这是最后手段，不是第一步。

### 第 4 层：改网络 / 改 loss（你现在可以暂缓）

没经验时：

- 先别改 `arch` 定义
- 先别加自定义 module
- 用 issue / 文档 / 问 AI 改 yml 即可

等第 1～2 层都试过，仍明确是「模型容量/结构不够」再学改网络。

---

## 决策树（简短版）

用自己的数据训 BasicSR

│

▼

loss/指标/出图正常？

┌───┴───┐

是 否

│ │

进入 先查数据和 yml（第 1 层）

正式 │

微调 ├─ 修好了 → 再训

│

├─ 换官方 model/ckpt（第 2 层）

│

├─ 仍不行 → 数据标注/任务是否适合 restoration？

│ ├─ 不适合 → 考虑换任务/换仓库（第 3 层）

│ └─ 适合 → 加长训练、找相似 paper 配置

│

└─ 有经验后再改网络（第 4 层，你现在可跳过）

