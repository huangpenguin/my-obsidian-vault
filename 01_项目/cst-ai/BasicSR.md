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


## 为什么你的 SwinIR 会变糊？

SwinIR 这类基于 Transformer 或 CNN 的图像恢复模型，在微调时如果使用了 **MSE (L2 Loss)** 或 **L1 Loss** 作为主要损失函数，模型倾向于输出所有可能正确结果的“平均值”。

- **结果**：极大地平滑掉了高频噪声，但同时也顺手把边缘、纹理等高频细节给“抹平”了，导致视觉上变糊。

## 更好的优化方案（除了加 SR 模型）

如果你不想让工作流（Workflow）变得过于复杂（增加显存和推理时间），你也可以尝试在 **SwinIR 自身的微调/后处理** 上做文章：

### 方案 A：调整 SwinIR 的训练 Loss（如果你还在训练阶段）

- 纯 L1/MSE Loss 会导致变糊。建议在损失函数中加入 **Perceptual Loss（感知损失/VGG Loss）** 或 **Frequency/FFT Loss（频域损失）**，强制模型保留高频边缘。
    

### 方案 B：特征融合 / 残差叠加（最简单有效）

- **原理**：SwinIR 去掉的部分，本质上是 `[原始图像] - [SwinIR 输出] = [噪声 + 丢失的微小细节]`。
    
- **做法**：尝试将 SwinIR 输出的图与原图按照一定比例进行融合（Blend），例如：
    
    $$\text{最终图} = 0.8 \times \text{SwinIR输出} + 0.2 \times \text{原图}$$
    
    这样可以在很大程度上找回边缘锐度，同时压制大部分噪声。
    

### 方案 C：后接“边缘增强 / 频域锐化”节点

- 在 Workflow 的 SwinIR 节点后面，直接接一个 **Unsharp Mask (USM 锐化)** 或 **Laplacian 边缘增强** 算法。对于 CT 图像，有时简单的经典频域锐化算法比深度学习 SR 模型更快且更稳定，不会产生幻觉（Hallucination）。





**而且将残差/skip-connection的思想直接整合进 SwinIR 网络内部，往往比在网络外面做后处理混合效果要好得多。**

---

## 1. 为什么建议写进网络内部，而不是只在外部 Blend？

- **外部 Blend（简单加权相加）**：
    
    $$\text{Output} = \alpha \times \text{SwinIR}(X) + (1-\alpha) \times X$$
    
    - **缺点**：这是“盲目”的。它在恢复清晰度的同时，把已被 SwinIR 抹掉的**噪声也按比例带回去了**。
        
- **网络内部残差（Residual / Skip Connection）**：
    
    在网络结构内部（比如网络输入到输出之间，或者浅层特征与深层特征之间）引入残差连接：
    
    - **优点**：网络可以在训练过程中**自适应（Adaptive）地学习“该保留哪些边缘细节，该丢弃哪些噪声”**。梯度可以直接传回给网络，让神经网络自己去平衡“去噪”与“保真”的权重。
        

---

## 2. 具体可以在网络中怎么改？（3 个实用改造方案）

针对 SwinIR 的架构特点，你可以考虑以下几种改法：

### 方案 A：全局长残差（Global Residual Connection）——最简单直接

- **做法**：直接让网络预测**残差（噪声/伪影图）**，而不是直接预测重构后的图。
    
    $$\text{Loss} = \Vert{}\text{SwinIR}(X_{noisy}) + X_{noisy} - Y_{clean}\Vert{}$$
    
    或者在输出层加入：
    
    $$\text{Output} = \text{SwinIR\_Features}(X) + \text{Conv}_{1\times 1}(X)$$
    
- **效果**：迫使 SwinIR 的深层网络只专注于学习“高频改动（去噪量）”，降低了整个网络的学习难度，天然保留了输入图像中的低频和结构信息。
    

### 方案 B：浅层特征融合（Multi-scale / Shallow Feature Fusion）

- **做法**：SwinIR 的前几层（浅层 CNN / Early Transformer Blocks）捕捉到了非常丰富的边缘、轮廓等高频特征，而越往深层，语义信息越丰富但细节越平滑。
    
- **改进**：将浅层特征提取器（浅层 Feature Map）通过 一个 $1\times 1$ 卷积或 Channel Attention 模块，直接跨层连接（Skip-Connection）并加到最后重构层之前。
    

### 方案 C：引入边缘感知分支（Edge-guided / Multi-branch SwinIR）

- **做法**：设计一个双分支架构。
    
    - **主分支**：标准的 SwinIR Transformer 主干，负责去噪重构。
        
    - **辅助分支**：一个轻量级的边缘检测/高频提取分支（如 Sobel 算子筛选出的 Edge Map 或者小 CNN）。
        
    - **融合层**：在网络末端通过 Cross-Attention 或 Feature Concatenation 将边缘特征注入到主分支中。
        

---

## 3. 搭配损失函数（Loss Function）改进是关键

单纯改网络结构（增加残差）如果依然只用 **L1 Loss / MSE Loss**，网络最终还是会被“逼着”输出平滑的平均值，导致变糊。

建议在修改网络结构的同时，配合以下 Loss：

1. **Charbonnier Loss / L1 Loss**：作为基础 Reconstruction Loss，保证整体灰度和结构正确。
    
2. **Edge Loss / Gradient Loss（梯度损失）**：计算预测图与 GT 图在 $x, y$ 方向上的梯度差（Sobel/Laplacian 滤波后算 L1）。
    
    - 专治“边缘变糊”，强制网络输出锐利的边界。
        
3. **FFT / Frequency Loss（频域损失）**：对预测图和真值图做 2D 傅里叶变换（FFT），计算高频频域的 L1 距离。
    
    - 能极大改善工业/医学 CT 图像中高频纹理缺失的问题。
        

---

### 💡 总结

**写进网络内部不仅非常有必要，而且是解决图像恢复模型“过度平滑”的标准做法。**

- 如果你想**低成本试错**：先在 SwinIR 加上**全局残差连接** + 梯度损失（Gradient Loss）重新训练，通常就能看到边缘清晰度大幅提升。
    
- 如果你追求**极限细节**：再考虑加入浅层特征跨层融合（Skip Connection）。







---