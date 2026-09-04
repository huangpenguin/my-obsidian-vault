# Transformer 架构与核心机制：视频 + 问答整合笔记

> 视频来源：Bilibili `BV1Pyta66EDh`  
> 原链接：https://www.bilibili.com/video/BV1Pyta66EDh/  
> 整理日期：2026-09-04
>
> **说明**：当前环境无法直接取得该 Bilibili 视频的字幕/音频，因此下面的「Transformer 主线笔记」按你围绕该视频提出的问题、标准 Transformer 结构以及可验证的公开技术资料进行系统化整理；不会伪造视频中的逐句表述。你与 ChatGPT 的问答内容已尽量完整并入，并在最后加入了若干**纠错/补充**，避免把便于理解的比喻误当成严格事实。

---

## 0. 一页速览：Transformer 到底在做什么？

Transformer 可以粗略理解为：

1. **Tokenization**：文字切成 token。
2. **Embedding**：每个 token 变成一个高维向量。
3. **加入位置信息**：告诉模型 token 的顺序。
4. 反复经过多个 Transformer Block：
   - **Attention**：让不同 token 之间交换信息。
   - **FFN / MLP**：每个 token 对已经汇总好的信息做非线性加工。
   - **Residual + Norm**：保证深层网络训练稳定。
5. 最后通过线性层 / Softmax 得到下一 token 的概率。

最核心的两句话：

> **Attention = token 之间交流。**  
> **FFN = 每个 token 自己消化交流后的信息。**

---

# 1. FFN：Transformer 里的前馈神经网络是不是普通 MLP？

是。

Transformer 中的 FFN（Feed-Forward Network）本质上就是一个**逐 token 使用的 MLP**。

经典形式：

\[
\mathrm{FFN}(x)
=
\sigma(xW_1+b_1)W_2+b_2
\]

常见的现代形式会使用 GELU / SwiGLU 等，例如：

\[
\mathrm{FFN}(x)
=
W_2\left(\mathrm{SiLU}(xW_g)\odot(xW_u)\right)
\]

其中：

- 输入维度通常为 \(d_{\text{model}}\)
- 中间维度 \(d_{\text{ff}}\) 通常大于 \(d_{\text{model}}\)
- 输出重新回到 \(d_{\text{model}}\)

例如：

\[
768 \rightarrow 3072 \rightarrow 768
\]

### “Feed-forward” 是不是表示只推理一次？

更准确地说：

- **FFN 内部没有像 RNN 那样的时间递归**
- 一次调用 FFN 就是一遍前向传播
- 但 Transformer 有很多层，所以同一个 token 会在不同 block 里反复经过不同 FFN

因此不能理解成“整个模型只算一次”，而应该理解为：

> 每个 block 里的 FFN 对当前隐藏状态做一次普通的前向 MLP 变换。

---

# 2. Attention、Self-Attention、Cross-Attention、Multi-Head Attention

## 2.1 Attention 是总框架

Attention 的一般形式：

\[
Q = X_QW_Q,\qquad
K = X_KW_K,\qquad
V = X_VW_V
\]

\[
\mathrm{Attention}(Q,K,V)
=
\mathrm{softmax}
\left(
\frac{QK^\top}{\sqrt{d_k}}
\right)V
\]

直觉：

- **Q / Query**：我现在想找什么？
- **K / Key**：我这里有什么信息，可以怎样被匹配？
- **V / Value**：如果你决定关注我，真正拿走的内容是什么？

可以把它记成：

> **Q 找 K，得到权重，再按权重读取 V。**

---

## 2.2 Self-Attention 为什么也叫 Attention？

Self-Attention 仍然使用完全相同的 Attention 公式。

区别只在 **Q/K/V 的来源**：

\[
Q=XW_Q,\qquad
K=XW_K,\qquad
V=XW_V
\]

也就是说 Q、K、V 都由**同一段隐藏状态 \(X\)** 投影而来。

因此：

> Self-Attention 是 Attention 的一种特例：  
> **序列自己和自己做 attention。**

---

## 2.3 Cross-Attention

Cross-Attention 中：

- Q 来自一个序列
- K、V 来自另一个序列

例如经典 Encoder-Decoder Transformer：

\[
Q = X_{\text{decoder}}W_Q
\]

\[
K = X_{\text{encoder}}W_K,\qquad
V = X_{\text{encoder}}W_V
\]

Decoder 用当前状态去“查询” Encoder 的表示。

---

## 2.4 Multi-Head Attention 是什么？

Multi-Head Attention（MHA）是：

> 同时在多个低维投影空间中计算 Attention，再把结果合并。

第 \(i\) 个 head：

\[
Q_i=XW_i^Q,\qquad
K_i=XW_i^K,\qquad
V_i=XW_i^V
\]

\[
\mathrm{head}_i
=
\mathrm{Attention}(Q_i,K_i,V_i)
\]

多个 head 拼接：

\[
H
=
\mathrm{Concat}
(
\mathrm{head}_1,\ldots,\mathrm{head}_h
)
\]

再经过输出投影：

\[
\mathrm{MHA}(X)=HW^O
\]

因此：

- **Self / Cross** 描述的是 QKV **从哪里来**
- **Multi-Head** 描述的是 Attention **并行分成多少个 head 计算**

它们不是互斥概念。

可以有：

- Multi-Head Self-Attention
- Multi-Head Cross-Attention

---

# 3. Multi-Head 到底是怎么“切”的？

这是你这次问答里最重要、也最容易产生误解的一部分。

假设：

\[
d_{\text{model}}=512,\qquad h=8
\]

经典 MHA 中：

\[
d_{\text{head}}=\frac{512}{8}=64
\]

## 3.1 不是直接把原始 \(X\) 的 0~63 维交给 Head 1

严格来说，真正流程是：

\[
X
\rightarrow
XW_Q
=
Q_{\text{total}}
\]

如果：

\[
X\in\mathbb{R}^{N\times512}
\]

而：

\[
W_Q\in\mathbb{R}^{512\times512}
\]

则：

\[
Q_{\text{total}}
\in
\mathbb{R}^{N\times512}
\]

然后 reshape 成：

\[
N\times h\times d_{\text{head}}
=
N\times8\times64
\]

### 关键点

虽然 reshape 最后确实是按连续内存维度分块：

- Head 1：投影输出第 0~63 维
- Head 2：第 64~127 维
- ...

但是：

> 这些已经不是原始输入 \(X\) 的“原始第 0~63 维”。

在此之前：

\[
XW_Q
\]

已经让 **\(X\) 的所有 512 个输入维度都可以参与生成任意一个输出维度**。

所以 Head 1 的 64 个 Q 特征，本质是：

> 对输入 512 维信息学习出来的一组线性组合。

---

# 4. 每个 Head 到底“看什么”是谁决定的？

## 4.1 人工确定的部分

工程师在建模时先决定：

- \(d_{\text{model}}\)
- head 数 \(h\)
- 每个 head 的维度 \(d_{\text{head}}\)

例如：

\[
d_{\text{model}}=4096,\qquad h=32,\qquad d_{\text{head}}=128
\]

这里的 \(h\) 是**架构超参数**，不是训练过程中自动增加或减少的。

---

## 4.2 学习出来的部分

每个 head 的：

\[
W_i^Q,\;W_i^K,\;W_i^V
\]

都是可学习参数。

初始化时通常是随机初始化，因此刚开始：

- Head 1 没有“语法 head”的固定含义
- Head 2 也没有“指代 head”的固定含义

训练时：

\[
\theta
\leftarrow
\theta-\eta\nabla_\theta L
\]

所有投影矩阵会通过梯度下降被更新。

所以更准确的描述是：

> **head 的数量和张量布局由人设计；head 的有效投影空间由训练学出来。**

---

## 4.3 为什么可以不在意 reshape 的前后顺序？

因为输出列的语义也是训练形成的。

假设 \(W_Q\) 是：

\[
512\times512
\]

其前 64 列最终进入 Head 1。

训练时反向传播自然会更新这 64 列，使它们形成对任务有用的 Head 1 投影。

所以：

> “第几列”本身没有先验语义，  
> **只要训练和推理始终保持相同的数据布局即可。**

---

# 5. Q、K、V 的矩阵到底多大？

设：

- batch：\(B\)
- token 数：\(N\)
- 模型宽度：\(d_{\text{model}}\)
- head 数：\(h\)
- head dimension：\(d_h\)

## 单个 token

\[
x\in\mathbb{R}^{1\times d_{\text{model}}}
\]

经典单 head 投影：

\[
W_i^Q
\in
\mathbb{R}^{d_{\text{model}}\times d_h}
\]

因此：

\[
Q_i
=
xW_i^Q
\in
\mathbb{R}^{1\times d_h}
\]

例如：

\[
(1\times512)(512\times64)
=
1\times64
\]

---

## 整个序列

\[
X
\in
\mathbb{R}^{N\times d_{\text{model}}}
\]

则：

\[
Q_i
=
XW_i^Q
\in
\mathbb{R}^{N\times d_h}
\]

---

## 工程实现

很多实现不会真的声明 \(h\) 个小矩阵，而是使用一个大的矩阵：

\[
W_Q
\in
\mathbb{R}^{d_{\text{model}}\times(hd_h)}
\]

经典 MHA 若满足：

\[
hd_h=d_{\text{model}}
\]

则：

\[
W_Q
\in
\mathbb{R}^{d_{\text{model}}\times d_{\text{model}}}
\]

然后：

\[
Q=XW_Q
\]

再 reshape：

\[
[B,N,h,d_h]
\]

接着 transpose 成：

\[
[B,h,N,d_h]
\]

便于每个 head 独立计算：

\[
QK^\top
\]

---

# 6. Token 向量为什么通常写成行向量？

若写：

\[
1\times d_{\text{model}}
\]

表示：

- 1 行
- \(d_{\text{model}}\) 列

所以它是**行向量**。

多个 token 直接上下堆叠：

\[
X=
\begin{bmatrix}
x_1\\
x_2\\
\vdots\\
x_N
\end{bmatrix}
\in
\mathbb{R}^{N\times d_{\text{model}}}
\]

然后就可以高效地一次做：

\[
XW_Q
\]

这是一种工程表示习惯。

传统线性代数有时把向量写成列向量：

\[
x\in\mathbb{R}^{d\times1}
\]

并写：

\[
Wx
\]

两者本质只是 convention 不同。

---

# 7. Attention 的矩阵到底算了什么？

设：

\[
Q,K\in\mathbb{R}^{N\times d_k}
\]

则：

\[
QK^\top
\in
\mathbb{R}^{N\times N}
\]

第 \((i,j)\) 项：

\[
q_i\cdot k_j
\]

表示 token \(i\) 的 Query 对 token \(j\) 的 Key 的匹配程度。

再缩放：

\[
S
=
\frac{QK^\top}{\sqrt{d_k}}
\]

经过 softmax：

\[
A
=
\mathrm{softmax}(S)
\]

得到 Attention Matrix：

\[
A\in\mathbb{R}^{N\times N}
\]

最后：

\[
AV
\]

相当于：

> 对每个 token，根据它对其他 token 的关注程度，把其他 token 的 Value 信息加权汇总到自己身上。

---

# 8. 为什么要除以 \(\sqrt{d_k}\)？

如果 Q 和 K 各维近似独立且方差约为 1：

\[
q\cdot k
=
\sum_{j=1}^{d_k}q_jk_j
\]

其方差会随着 \(d_k\) 增大。

若点积数值很大：

- softmax 很容易饱和
- 概率极端接近 0 / 1
- 梯度变小
- 训练不稳定

因此用：

\[
\frac{1}{\sqrt{d_k}}
\]

把数量级拉回来。

---

# 9. Self-Attention 的“同一个序列”到底能有多长？

数学上，如果输入 token 序列为：

\[
X=(x_1,x_2,\ldots,x_N)
\]

所有 token 都属于当前 Attention 计算的 sequence。

长度 \(N\) 受模型 context window 和实现约束。

标准全注意力的主要瓶颈：

\[
QK^\top
\]

需要生成：

\[
N\times N
\]

的注意力关系，因此经典复杂度近似：

\[
O(N^2)
\]

---

## PDF 是不是“一个序列”？

**有可能，但不能简单说所有 PDF 都一定整本一次性变成一个序列。**

实际系统可能：

1. 整份 PDF 的文本能放入 context window  
   → 可以作为一条长序列送入模型。

2. PDF 超过上下文限制  
   → 需要截断、chunking、滑窗等。

3. 应用使用 RAG  
   → 先把 PDF 切块并检索，只把相关 chunk 放进当前模型 context。

4. 多模态 PDF  
   → 页面可能以图像 patch / OCR / layout token 等方式编码，并不只是纯文本流。

因此：

> “一次 forward 真正进入 Transformer 的 token 集合”才是当前序列；  
> “用户上传了整本 PDF”不等于“整本 PDF 一定在一个全局 self-attention 矩阵里”。

---

# 10. Multi-Head 和 PCA 像不像？

你的类比**有一定直觉价值**，但数学上不能等同。

## 相似点

都涉及：

> 高维表示 → 不同低维方向 / 子空间 → 提取不同信息

PCA：

\[
z=XW_{\text{PCA}}
\]

MHA：

\[
Q_i=XW_i^Q,\quad
K_i=XW_i^K,\quad
V_i=XW_i^V
\]

都存在投影到子空间的思想。

---

## 核心区别

| PCA | Multi-Head Attention |
|---|---|
| 目标是解释数据方差 | 目标是完成任务、降低训练 loss |
| 主成分正交 | head 没有正交约束 |
| PC 有方差大小排序 | head 一般没有天然主次 |
| 通常是固定统计分解 | 投影矩阵由梯度下降学习 |
| PCA 本身不做 token-token interaction | Attention 会动态计算 token-token 权重 |

所以更好的记忆：

> PCA：寻找数据的主要坐标轴。  
> MHA：训练多个可学习的 Attention 投影视角。

---

# 11. “不同 Head 学不同语义”该怎么准确理解？

初学时可以用：

- Head 1 看语法
- Head 2 看指代
- Head 3 看局部关系

来建立直觉。

但严格来说：

> **模型没有被强制规定每个 head 必须学习一个清晰、独立、可人类命名的语义功能。**

现实中：

- 不同 head 可能高度冗余
- 多个 head 可能做相似的事情
- 一个 head 可能同时承担多种模式
- 有些 head 可以被 pruning 而几乎不影响性能

所以“多位不同专业专家”是很好的入门比喻，但不是严格的结构约束。

---

# 12. Normalization vs Regularization：最容易混淆的两个词

## 12.1 Normalization

中文常见：

- 归一化
- 标准化
- 规范化（有时）

日语：

- **正規化（せいきか）**
- ノーマライゼーション

主要目的：

> 改善数值尺度 / 激活分布，让优化更稳定。

例如：

### LayerNorm

给一个 token 的隐藏向量：

\[
x=(x_1,\ldots,x_d)
\]

均值：

\[
\mu
=
\frac1d\sum_i x_i
\]

方差：

\[
\sigma^2
=
\frac1d
\sum_i(x_i-\mu)^2
\]

标准化：

\[
\hat x_i
=
\frac{x_i-\mu}
{\sqrt{\sigma^2+\epsilon}}
\]

再加可学习缩放与偏置：

\[
y_i
=
\gamma_i\hat x_i+\beta_i
\]

### 注意

Normalization 并不是一定要让数据真的服从“正态分布”。

“Normalization”更广义地表示：

> 对数值尺度/统计性质做规范化处理。

---

## 12.2 Regularization

中文：

- 正则化

日语：

- **正則化（せいそくか）**
- レギュラライゼーション

目的：

> 降低过拟合，提高泛化能力。

典型方法：

- L1 regularization
- L2 regularization / weight decay
- Dropout
- data augmentation
- early stopping

L2 示例：

\[
L_{\text{total}}
=
L_{\text{task}}
+
\lambda\lVert W\rVert_2^2
\]

---

## 12.3 最简单记忆

### Normalization
**N = Numbers**

> 管数值 / 激活尺度。

### Regularization
**R = Rules / Restriction**

> 给模型加限制，减少过拟合。

日语尤其可以记：

- 正**規**化 → Normalization
- 正**則**化 → Regularization

---

# 13. Activation / Activation Vector 到底是什么？

假设一层神经网络：

\[
z=Wx+b
\]

再经过激活函数：

\[
a=\sigma(z)
\]

这里的：

\[
a
\]

就是 activation。

如果该层有多个神经元：

\[
a=
[a_1,a_2,\ldots,a_d]
\]

就是一个 activation vector。

可以理解为：

> 输入经过当前层加工以后产生的**中间表示**。

---

## 权重 vs Activation

### Weight

\[
W
\]

- 模型长期保存的参数
- 训练完成后推理时通常固定
- 类似“机器结构 / 长期知识”

### Activation

\[
a
\]

- 根据当前输入实时计算
- 每次输入都不同
- 类似“当前这个样本经过模型后产生的工作状态”

---

# 14. Residual Connection 和 LayerNorm 为什么重要？

## Residual

\[
y=x+F(x)
\]

作用：

- 给梯度提供更直接的传播路径
- 减轻深层网络优化困难
- 允许子层学习“增量修改”而不是完全重建表示

---

## LayerNorm

稳定隐藏状态尺度。

现代 LLM 经常使用 Pre-Norm：

\[
x'
=
x+\mathrm{Attention}(\mathrm{Norm}(x))
\]

\[
y
=
x'+\mathrm{FFN}(\mathrm{Norm}(x'))
\]

而原始 Transformer 论文是经典 Post-Norm 风格：

\[
\mathrm{Norm}(x+\mathrm{Sublayer}(x))
\]

现代大模型大量采用 Pre-Norm 或其变体，因为深层训练通常更稳定。

---

# 15. Masked / Causal Self-Attention

GPT 生成第 \(i\) 个 token 时不能偷看未来 token。

因此在 Attention Score 上加 causal mask：

\[
M_{ij}
=
\begin{cases}
0,&j\le i\\
-\infty,&j>i
\end{cases}
\]

然后：

\[
A
=
\mathrm{softmax}
\left(
\frac{QK^\top}{\sqrt{d_k}}
+
M
\right)
\]

未来位置变成：

\[
\mathrm{softmax}(-\infty)\approx0
\]

于是 token \(i\) 只能读取：

\[
1,\ldots,i
\]

的信息。

---

# 16. Transformer Block 的完整数据流

一个典型 decoder-only LLM block 可以抽象成：

\[
x_0
=
\mathrm{Embedding}
+
\mathrm{PositionInfo}
\]

### Attention

\[
x_1
=
x_0
+
\mathrm{SelfAttention}
(
\mathrm{Norm}(x_0)
)
\]

### FFN

\[
x_2
=
x_1
+
\mathrm{FFN}
(
\mathrm{Norm}(x_1)
)
\]

重复 \(L\) 层：

\[
x_L
\]

最后：

\[
\mathrm{logits}
=
x_LW_{\text{vocab}}
\]

\[
p(token)
=
\mathrm{softmax}(\mathrm{logits})
\]

---

# 17. 你问到的“自适应”思想：深度学习中到处存在

你抓到的核心哲学是：

> **人类决定计算图和结构约束；梯度下降决定参数怎样利用这个结构。**

Multi-Head 只是其中一个例子。

---

## 17.1 CNN 卷积核

人类指定：

- kernel size
- channel 数
- stride

但不会规定：

> 第 17 个卷积核一定负责检测眼睛。

训练自己决定每个 filter 最终学什么。

---

## 17.2 MoE

人类决定：

- 有多少 expert
- 每个 token 激活 top-k expert
- router 结构

但：

> router 如何分配 token、不同 expert 最终擅长什么，是训练形成的。

---

## 17.3 LoRA

LoRA 规定：

\[
\Delta W
=
BA
\]

其中 rank：

\[
r\ll d
\]

例如：

\[
A\in\mathbb{R}^{r\times d_{\text{in}}}
\]

\[
B\in\mathbb{R}^{d_{\text{out}}\times r}
\]

人类指定 rank \(r\)，但低秩子空间具体学什么由训练决定。

---

## 17.4 Embedding

人类只规定 embedding dimension，例如：

\[
d=4096
\]

不会规定：

- 第 1 维 = “性别”
- 第 2 维 = “颜色”
- 第 3 维 = “国家”

语义分布是模型训练形成的。

---

# 18. 关于 RoPE 的更准确理解

之前问答里有一个容易过度简化的地方。

RoPE（Rotary Position Embedding）不是简单地做到：

> “距离越近，attention 分数一定越高”。

它主要通过对 Q/K 按位置进行旋转，使点积能够显式依赖**相对位置差**。

可以抽象为：

\[
q_m
=
R_mq
\]

\[
k_n
=
R_nk
\]

则：

\[
q_m^\top k_n
=
q^\top
R_m^\top R_n
k
\]

而：

\[
R_m^\top R_n
\]

只依赖：

\[
n-m
\]

所以它把相对位置信息编码进 Q/K 点积。

---

# 19. 对本次问答中几个关键点的纠错 / 精化

## 19.1 “d_model 必须能被 h 整除”

对**经典标准 MHA 实现**通常成立：

\[
d_h=\frac{d_{\text{model}}}{h}
\]

但从一般 Attention 数学上，不是宇宙级硬约束。

你完全可以设计：

\[
h d_h
\ne
d_{\text{model}}
\]

再使用额外投影矩阵变换回来。

现代架构还会出现：

- MQA
- GQA
- 不同数量的 Q heads / KV heads

所以应该记成：

> 经典 Transformer 通常这样设计，主要为了结构和工程效率。

---

## 19.2 “每个 head 一定学不同语义”

不保证。

训练目标只要求整体 loss 下降，不要求 head 之间：

- 正交
- 独立
- 不重复
- 具有人类可解释分工

---

## 19.3 “PDF 就是一个序列”

只在**真正整份输入模型 context**时成立。

应用层经常做：

- chunk
- RAG
- page selection
- context compression

所以要区分：

> 文件级输入 ≠ 模型一次 forward 的序列。

---

## 19.4 “Regularization 只作用于 Weight / Loss”

这只是便于记忆。

例如：

- Dropout 直接操作 activation
- Data augmentation 操作输入数据
- Early stopping 操作训练过程

Regularization 的共同目标是：

> 减少过拟合，而不是限定必须作用在哪种 tensor 上。

---

# 20. 一张图式记忆整个 Transformer

```text
Token IDs
   │
   ▼
Embedding
   │
   + Position Information
   │
   ▼
┌─────────────────────────────────────┐
│ Transformer Block                   │
│                                     │
│   Norm                              │
│    │                                │
│    ▼                                │
│ Multi-Head Self-Attention           │
│    │                                │
│    └────── + Residual ───────────┐   │
│                                  │   │
│   Norm                           │   │
│    │                             │   │
│    ▼                             │   │
│   FFN / MLP                      │   │
│    │                             │   │
│    └────── + Residual ───────────┘   │
└─────────────────────────────────────┘
             × L layers
                  │
                  ▼
              Final Norm
                  │
                  ▼
               Linear
                  │
                  ▼
               Logits
                  │
                  ▼
               Softmax
                  │
                  ▼
            Next-token distribution
```

---

# 21. 最重要的 Shape Cheat Sheet

设：

\[
B=\text{batch size}
\]

\[
N=\text{sequence length}
\]

\[
D=d_{\text{model}}
\]

\[
H=\text{num heads}
\]

\[
d_h=\text{head dimension}
\]

经典情况：

\[
D=Hd_h
\]

| Tensor | Shape |
|---|---|
| \(X\) | \([B,N,D]\) |
| \(W_Q\) | \([D,Hd_h]\) |
| \(Q\) | \([B,N,Hd_h]\) |
| reshape Q | \([B,N,H,d_h]\) |
| transpose Q | \([B,H,N,d_h]\) |
| \(K^\top\) | \([B,H,d_h,N]\) |
| \(QK^\top\) | \([B,H,N,N]\) |
| Attention output per head | \([B,H,N,d_h]\) |
| concat | \([B,N,Hd_h]\) |
| output projection | \([B,N,D]\) |

这个表如果记住，Multi-Head Attention 的大部分实现都不会再混乱。

---

# 22. 最终概念对照表

| 概念 | 核心问题 | 一句话 |
|---|---|---|
| Attention | 信息应该从哪里读？ | Q 匹配 K，加权读取 V |
| Self-Attention | QKV 从哪里来？ | 都来自当前序列 |
| Cross-Attention | QKV 从哪里来？ | Q 与 KV 来自不同序列 |
| Multi-Head | 从几个投影视角做 Attention？ | 多组可学习 QKV 并行 |
| FFN | 每个 token 如何非线性加工？ | 逐 token 的 MLP |
| LayerNorm | 数值如何稳定？ | 规范化隐藏状态 |
| Regularization | 怎么减少过拟合？ | 给学习过程加入约束 |
| Activation | 当前输入算出了什么中间状态？ | 网络的动态中间表示 |
| Residual | 深层网络怎么更容易训练？ | \(x+F(x)\) |
| Causal Mask | 怎么禁止看未来？ | 未来 attention score 设为 \(-\infty\) |
| RoPE | token 位置怎么进入 Attention？ | 旋转 Q/K，使点积依赖相对位置 |
| LoRA | 如何低成本微调？ | 用低秩 \(BA\) 表示 \(\Delta W\) |

---

# 23. 推荐的学习顺序

如果要继续往 LLM / VLA 深入，建议按下面顺序：

1. **把本笔记中的 Shape 全部真正推一遍**
2. 手写一次：
   \[
   Q=XW_Q,\ K=XW_K,\ V=XW_V
   \]
3. 手算一个 3-token、2-head 的 tiny attention
4. PyTorch 手写：
   - linear
   - reshape
   - transpose
   - \(QK^\top\)
   - mask
   - softmax
   - \(AV\)
5. 再理解：
   - Pre-Norm / Post-Norm
   - RoPE
   - KV Cache
   - MQA / GQA
   - FlashAttention
6. 最后再进入 LLM / VLA：
   - token vs continuous action
   - action discretization
   - action chunk
   - multimodal attention
   - cross-attention / projector
   - flow matching action head

---

# 24. 你这次最值得保留的几个直觉

### 直觉 1

> **Attention 是 token 之间交流；FFN 是 token 自己消化。**

这是理解 Transformer block 最有价值的第一层抽象。

### 直觉 2

> **Multi-Head 和 PCA 都有“不同子空间观察数据”的味道，但 MHA 的子空间由任务 loss 学习，不要求正交。**

### 直觉 3

> **reshape 本身只是排布；真正形成每个 head 语义空间的是前面的可学习投影。**

### 直觉 4

> **神经网络里很多维度本身没有人工语义，语义是优化过程把它“填进去”的。**

### 直觉 5

> **架构规定可学习参数的活动范围，训练决定这些参数最终怎样使用这个范围。**

这条思想不只适用于 Transformer，也适用于 CNN、MoE、LoRA、Embedding、VLA 等几乎所有现代深度学习架构。

---

## Source

- Bilibili video: https://www.bilibili.com/video/BV1Pyta66EDh/
- Conversation notes: 本次对话中围绕 Transformer / Attention / FFN / Normalization / Multi-Head / QKV shape / Reshape / LoRA 等问题的连续问答
