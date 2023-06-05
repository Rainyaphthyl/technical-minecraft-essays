# 刷怪概率与速率的计算

**摘要：**

**关键词：**「」

# **引言**

## **游戏版本声明**

在没有特殊说明的情况下，本文所有讨论内容均默认只考虑MCJE 1.12.2的情况，没有验证对于其他版本的适用性。即使是提到了版本差异的段落，对于其他版本的讨论内容也是未经验证的。

本文的内容对其他版本仍然有参考意义，但引用时需要仔细甄别其中的各种版本差异。

# **单方块的刷怪效率计算**

## **参数设置**

考虑以下与刷怪过程有关的随机变量：

- $\vec{S}$ ：生物最终生成的位置；
- $\vec{A}_k$ ：剩余 $k$ 次游走机会时，游走选中的位置；
- $\vec{I}$ ：一组刷怪尝试中，游走的起点坐标；
- $B$ ：一组刷怪尝试中，游走的次数；
- $\xi$ ：一组刷怪尝试中，生物生成列表条目（`SpawnListEntry`），包含生物种类、成群生成数量范围、选择权重。

其中，
- $\vec{S}, \vec{A}_k, \vec{I}$ 均为二元向量，表示生物生成位置的 $\left(X, Z\right)$ 方块坐标；
- $B$ 的取值为正整数，具体范围与 $\xi$ 的取值有关；
- $\xi$ 的取值为`SpawnListEntry`对象，具体范围与方块所在的生物群系有关；
- $\vec{S}$ 可以视为 $\vec{A}_k$ 的特殊情况： $\vec{S} = \vec{A}_0$ ，即最后一次游走选中的位置即为实际生成位置；
- $\vec{I}$ 并不能视为 $\vec{A}_k$ 的特殊情况，具体原因可见于下文。

## **群系无关的单组概率**

### **概述**

本文的核心目的之一是计算在某一gt的刷怪阶段中，特定位置生成某类生物的概率。为简化模型，本节暂时只考虑与生物种类无关的情况，即只考虑与刷怪位置的选取有关的过程。

实际的刷怪循环中，每gt平均每区块会进行3组刷怪尝试；为继续简化模型，本节计算的概率是1组刷怪尝试（包含一次或多次游走）的生成概率，记为：

$$ P\left\lbrace \vec{S} = \left(x_0, z_0\right) \right\rbrace,
\vec{r}_0 = \left(x_0, z_0\right) $$

如果需要计算3组刷怪尝试提供的总概率，需要进一步的推演。

根据刷怪游走过程的基本机制，可以给出各类概率之间的关系。

### **生成位置的初步推导**

生成位置由最后一次游走的情况决定，概率关系为：

$$ P\left\lbrace \vec{S} = \vec{r}_0 \right\rbrace 
= \sum_{\vec{r}_1 \in \delta\left(\vec{r}_0 \right)} {\left(
    P\left\lbrace \vec{S} = \vec{r}_0 \space | \space
    \vec{A}_1 = \vec{r}_1 \right\rbrace \cdot P\left\lbrace \vec{A}_1 = \vec{r}_1 \right\rbrace
\right)} $$

其中， $\delta\left(\vec{r}_i\right)$ 表示可能以 $\vec{r}_i$ 为终点的全部单次游走起点的集合，实际定义为以 $\vec{r}_i$ 为中心的 $11 \times 11$ 正方形空间（ $11 = 5 + 1 + 5$ ），粗略描述如下：

$$ \delta\left(\vec{r}_i\right) = \left\lbrace \vec{r}_j \space | \space
\left(\vec{r}_j - \vec{r}_i\right) \in
\left(\left[-5, +5\right]^2 \cap \mathbb{Z}^2\right) \right\rbrace $$

或描述为：

$$ \delta\left(\vec{r}_i\right) = \left\lbrace
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

### **刷怪游走的迭代**

考虑到 $\vec{S} = \vec{A}_0$ ，可以将上述公式推广到各次刷怪游走过程中，但需要注意一个额外选项：某次刷怪游走的起点可能由上一次游走决定，也可能在本次即为初始游走的情况下由刷怪循环直接决定。这就需要考虑 $B$ 和 $\vec{I}$ 的影响。

具体来说，此处需要考虑以下情况：

刷怪循环的1组刷怪尝试中，对于 $B$ 的任何可能的取值，在剩余 $i$ 次游走机会的情况下，此次游走以位置 $\vec{r}_i$ 为起点的概率： $P\left\lbrace \vec{A}_i = \vec{r}_i \right\rbrace$ 。

由上述讨论可得以下表达式：

$$ P\left\lbrace \vec{A}_i = \vec{r}_i \right\rbrace \\
= \sum_{\vec{r} \in \delta\left(\vec{r}_i\right)} {\left(
    P\left\lbrace \vec{A}_i = \vec{r}_i \space | \space \vec{A}_{i+1} = \vec{r} \right\rbrace
    \cdot P\left\lbrace \vec{A}_{i+1} = \vec{r} \right\rbrace
\right)} \\
+ P\left\lbrace \vec{I} = \vec{r}_i \right\rbrace
\cdot P\left\lbrace B = i \right\rbrace $$

其中， $1 \leq i < \beta$ ，并且 $\left[1, \beta\right] \cap \mathbb{N}$ 是 $B$ 的可能取值范围，即：

$$ P\left\lbrace B = b \right\rbrace
= \begin{cases}
\frac{1}{\beta}, \space b \in D_B = \left[1, \beta\right] \cap \mathbb{N} \\
0, \space b \notin D_B
\end{cases} $$

$\beta$ 是与生物群系和生成列表有关的变量，因此本节暂不追问。

对于最大可能游走次数对应的初始位置，有以下表达式：

$$ P\left\lbrace \vec{A}_\beta = \vec{r}_\beta \right\rbrace 
= 0 + P\left\lbrace \vec{I} = \vec{r}_\beta \right\rbrace
\cdot P\left\lbrace B = \beta \right\rbrace $$

### **游走迭代公式的计算**

注意到上述表达式中的可重复部分，定义以下简化的表达式：

$$ g\left(\vec{\rho}\right)
= P\left\lbrace \vec{A}_k = \vec{0} \space | \space \vec{A}_{k+1} = \vec{\rho} \right\rbrace $$

可以这样解释 $g\left(\vec{\rho}\right)$ 的定义：
1. 考虑刷怪游走的机制，在相应取值范围内， $g\left(\vec{\rho}\right)$ 是一个与剩余游走次数 $k$ 无关的函数。
2. 又考虑到 $\vec{r}_{k+1}$ 与 $\delta\left(\vec{r}_k\right)$ 实际上具有固定的相对位置关系，可以将两者之差用单一的变量 $\vec{\rho} = \vec{r}_{k+1} - \vec{r}_k$ 来表示，即： $g\left(\vec{\rho}\right)$ 的返回值仅与两个坐标的相对位置有关，而与绝对位置无关：

$$ \forall{\vec{r}_1}, \forall{\vec{r}_2},
P\left\lbrace \vec{A}_k = \vec{r}_1 \space | \space \vec{A}_{k+1} = \vec{r}_1 + \vec{\rho} \right\rbrace
= P\left\lbrace \vec{A}_k = \vec{r}_2 \space | \space \vec{A}_{k+1} = \vec{r}_2 + \vec{\rho} \right\rbrace $$

同时， $\delta\left(\vec{0}\right)$ 的值显然是常量：

$$ \delta\left(\vec{0}\right) = \delta _0
= \left(\left[-5, +5\right]^2 \cap \mathbb{Z}^2\right) $$

刷怪游走概率的迭代公式核心部分可以改写为以下形式：

$$ f\left(\vec{r}_k, k\right)
= \sum_{\vec{\rho} \in \delta _0} {\left(
    g\left(\vec{\rho}\right)
    \cdot P\left\lbrace \vec{A}_{k+1} = \vec{r}_k + \vec{\rho} \right\rbrace
\right)} $$

# 相关资料

