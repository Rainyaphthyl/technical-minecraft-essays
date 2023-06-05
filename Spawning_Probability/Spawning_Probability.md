# 刷怪概率与速率的计算

**摘要：**

**关键词：**「」

# **引言**

## **游戏版本声明**

在没有特殊说明的情况下，本文所有讨论内容均默认只考虑MCJE 1.12.2的情况，没有验证对于其他版本的适用性。即使是提到了版本差异的段落，对于其他版本的讨论内容也是未经验证的。

# **单方块的刷怪效率计算**

## **参数设置**

考虑以下与刷怪过程有关的随机变量：

- $\vec{S}$ ：生物最终生成的位置；
- $\vec{A_k}$ ：剩余 $k$ 次游走机会时，游走选中的位置；
- $\vec{I}$ ：一组刷怪尝试中，游走的起点坐标；
- $G$ ：一组刷怪尝试中，游走的次数；
- $K$ ：一组刷怪尝试中，生物生成列表条目（`SpawnListEntry`），包含生物种类、成群生成数量范围、选择权重。

其中，
- $\vec{S}, \vec{A_k}, \vec{I}$ 均为二元向量，表示生物生成位置的 $\left(X, Z\right)$ 方块坐标；
- $W$ 的取值为正整数，具体范围与 $K$ 的取值有关；
- $K$ 的取值为`SpawnListEntry`对象，具体范围与方块所在的生物群系有关；
- $\vec{S}$ 可以视为 $\vec{A_k}$ 的特殊情况： $\vec{S} = \vec{A_0}$ ，即最后一次游走选中的位置即为实际生成位置；
- $\vec{I}$ 并不能视为 $\vec{A_k}$ 的特殊情况，具体原因可见于下文。

## **群系无关的单组概率**

本文的核心目的之一是计算在某一gt的刷怪阶段中，特定位置生成某类生物的概率。为简化模型，本节暂时只考虑与生物种类无关的情况，即只考虑与刷怪位置的选取有关的过程。

实际的刷怪循环中，每gt平均每区块会进行3组刷怪尝试；为继续简化模型，本节计算的概率是1组刷怪尝试（包含一次或多次游走）的生成概率，记为：

$$ P\left\lbrace \vec{S} = \left(x_0, z_0\right) \right\rbrace,
\vec{r_0} = \left(x_0, z_0\right) $$

对于坐标取值采用以下符号描述：

$$ \forall{i}\left\lbrace{i \in \mathbb{N}} \rightarrow
\vec{r_i} = \left(x_i, z_i\right) \right\rbrace $$

如果需要计算3组刷怪尝试提供的总概率，需要进一步的推演。

根据刷怪游走过程的基本机制，可以给出 各类概率之间的关系。

生成位置由最后一次游走的情况决定，概率关系为：

$$ P\left\lbrace \vec{S} = \vec{r_0} \right\rbrace 
= \sum_{\vec{r_1}\in D\left(\vec{r_0}\right)} {
    P\left\lbrace \vec{A_1} = \vec{r_1} \right\rbrace \cdot
    P\left\lbrace \vec{S} = \vec{r_0} \mid \vec{A_1} = \vec{r_1} \right\rbrace
} $$

其中， $D\left(\vec{r_i}\right)$ 表示可能以 $\vec{r_i}$ 为终点的全部单次游走起点的集合，实际定义为以 $\vec{r_i}$ 为中心的 $11 \times 11$ 正方形空间（ $11 = 5 + 1 + 5$ ），粗略描述如下：
$$ D\left(\vec{r_i}\right) = \left\lbrace \vec{r_j} \mid
\left(\vec{r_j} - \vec{r_i}\right) \in
\left(\left[-5, +5\right]^2 \cap \mathbb{Z}^2\right) \right\rbrace \\
= \left\lbrace
\left(x_i-5, z_i-5\right), \left(x_i-5, z_i-4\right), \dots, \left(x_i-5, z_i\right), \dots, \left(x_i-5, z_i+5\right)
\right\rbrace \\
\cup \left\lbrace
\left(x_i-4, z_i-5\right), \dots, \left(x_i-4, z_i+5\right)
\right\rbrace \\
\cup \dots \\
\cup \left\lbrace
\left(x_i, z_i-5\right), \dots, \left(x_i, z_i+5\right)
\right\rbrace \\
\cup \dots \\
\cup \left\lbrace
\left(x_i+5, z_i-5\right), \dots, \left(x_i+5, z_i+5\right)
\right\rbrace $$

# 相关资料

