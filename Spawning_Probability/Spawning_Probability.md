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
    P\left\lbrace \vec{S} = \vec{r}_0 \mid
    \vec{A}_1 = \vec{r}_1 \right\rbrace \cdot P\left\lbrace \vec{A}_1 = \vec{r}_1 \right\rbrace
\right)} $$

其中， $\delta\left(\vec{r}_i\right)$ 表示可能以 $\vec{r}_i$ 为终点的全部单次游走起点的集合，实际定义为以 $\vec{r}_i$ 为中心的 $11 \times 11$ 正方形空间（ $11 = 5 + 1 + 5$ ），粗略描述如下：

$$ \delta\left(\vec{r}_i\right) = \left\lbrace \vec{r}_j \mid
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
    P\left\lbrace \vec{A}_i = \vec{r}_i \mid \vec{A}_{i+1} = \vec{r} \right\rbrace
    \cdot P\left\lbrace \vec{A}_{i+1} = \vec{r} \right\rbrace
\right)} \\
+ P\left\lbrace \vec{I} = \vec{r}_i \right\rbrace
\cdot P\left\lbrace B = i \right\rbrace $$

其中， $1 \leq i \leq \beta$ ，并且 $\left[\alpha, \beta\right] \cap \mathbb{N}$ 是 $B$ 的可能取值范围，即：

$$ P\left\lbrace B = n \right\rbrace
= \begin{cases}
\frac{1}{\beta -\alpha +1} &, \left(\alpha \leq n \leq \beta\right) \cap \left(b \in\mathbb{N}^+\right) \\
0 &, \text{else}
\end{cases} $$

$\alpha, \beta$ 是与生物群系和生成列表有关的变量，因此本节暂不追问。

对于最大可能游走次数对应的初始位置，有以下表达式：

$$ P\left\lbrace \vec{A}_\beta = \vec{r}_\beta \right\rbrace 
= 0 + P\left\lbrace \vec{I} = \vec{r}_\beta \right\rbrace
\cdot P\left\lbrace B = \beta \right\rbrace $$

上述公式暗示了以下事实：

$$ \left(\forall{k}\right), k > \beta \to P\left\lbrace \vec{A}_k = \vec{r}_k \right\rbrace = 0 $$

如果采用更严谨的描述方式，应该定义两个随机变量：剩余游走次数 $K$ 和 游走起点位置 $\vec{A}$ ，上述事实的实际含义为：

$$ \left(\forall{k}\right), k > \beta \to P\left\lbrace \left(\vec{A} = \vec{r}\right) \cap \left(K = k\right) \right\rbrace 
= P\left\lbrace K = k \right\rbrace = 0 $$

即剩余游走次数不可能大于 $\beta$ ，此时对于游走位置的讨论无意义。

### **游走迭代公式的简化**

注意到上述表达式中的可重复部分，定义以下简化的表达式：

$$ g\left(\vec{\rho}\right)
= P\left\lbrace \vec{A}_k = \vec{0} \mid \vec{A}_{k+1} = \vec{\rho} \right\rbrace $$

可以这样解释 $g\left(\vec{\rho}\right)$ 的定义：
1. 考虑刷怪游走的机制，在相应取值范围内， $g\left(\vec{\rho}\right)$ 是一个与剩余游走次数 $k$ 无关的函数。
2. 又考虑到 $\vec{r}_{k+1}$ 与 $\delta\left(\vec{r}_k\right)$ 实际上具有固定的相对位置关系，可以将两者之差用单一的变量 $\vec{\rho} = \vec{r}_{k+1} - \vec{r}_k$ 来表示，即： $g\left(\vec{\rho}\right)$ 的返回值仅与两个坐标的相对位置有关，而与绝对位置无关：

$$ \forall{\vec{r}_1}, \forall{\vec{r}_2},
P\left\lbrace \vec{A}_k = \vec{r}_1 \mid \vec{A}_{k+1} = \vec{r}_1 + \vec{\rho} \right\rbrace
= P\left\lbrace \vec{A}_k = \vec{r}_2 \mid \vec{A}_{k+1} = \vec{r}_2 + \vec{\rho} \right\rbrace $$

同时， $\delta\left(\vec{0}\right)$ 的值显然是常量：

$$ \delta\left(\vec{0}\right) = \delta _0
= \left(\left[-5, +5\right]^2 \cap \mathbb{Z}^2\right) $$

刷怪游走概率的迭代公式核心部分可以改写为以下形式：

$$ f\left(\vec{r}_k, k\right)
= \sum_{\vec{\rho} \in \delta _0} {\left(
    g\left(\vec{\rho}\right)
    \cdot P\left\lbrace \vec{A}_{k+1} = \vec{r}_k + \vec{\rho} \right\rbrace
\right)} $$

代入 $B$ 的分布情况之后，完整的迭代公式表示为以下形式：

$$ P\left\lbrace \vec{A}_k = \vec{r} \right\rbrace
= \begin{cases}
f\left(\vec{r}, k\right) &, 0 \leq k < \alpha \\
f\left(\vec{r}, k\right)
+ \frac{1}{\beta -\alpha +1} \cdot P\left\lbrace \vec{I} = \vec{r} \right\rbrace &, \alpha \leq k < \beta \\
\frac{1}{\beta -\alpha +1} \cdot P\left\lbrace \vec{I} = \vec{r} \right\rbrace &, k = \beta \\
\end{cases} $$

特别地，最终刷怪位置对应的概率可以记为：

$$ s\left(\vec{r}\right)
= P\left\lbrace \vec{S} = \vec{r} \right\rbrace
= f\left(\vec{r}, 0\right) $$

迭代步骤中对应的其他一些表达式可以简记为：

$$ a\left(\vec{r}, k\right)
= P\left\lbrace \vec{A}_k = \vec{r} \right\rbrace $$

$$ f\left(\vec{r}, k\right)
= \sum_{\vec{\rho} \in \delta _0} {\left(
    g\left(\vec{\rho}\right)
    \cdot a\left(\vec{r} + \vec{\rho}, k+1\right)
\right)} $$

$$ \phi\left(\vec{r}\right)
= P\left\lbrace \vec{I} = \vec{r} \right\rbrace $$

### **具体计算**

#### 前提介绍：

整理上述简化的表达式，可以得到以下迭代函数（忽略迭代路径不会到达的情况）：

$$ a\left(\vec{r}, k\right)
= \begin{cases}
\sum_{\vec{\rho} \in \delta _0} {\left(
    g\left(\vec{\rho}\right)
    \cdot a\left(\vec{r} + \vec{\rho}, k+1\right)
\right)} &, 0 \leq k < \alpha \\
\sum_{\vec{\rho} \in \delta _0} {\left(
    g\left(\vec{\rho}\right)
    \cdot a\left(\vec{r} + \vec{\rho}, k+1\right)
\right)}
+ \frac{1}{\beta -\alpha +1} \cdot \phi\left(\vec{r}\right) &, \alpha \leq k < \beta \\
\frac{1}{\beta -\alpha +1} \cdot \phi\left(\vec{r}\right) &, k = \beta \\
0 &, k > \beta \\
\end{cases} $$

在1.12.2中，可以直接认定游走次数的取值范围： $\alpha = 1,\space \beta = 4$ ；该参数的具体讲解和跨版本注意事项需参阅后文。

为完成上述计算，需要给出 $\phi\left(\vec{r}\right)$ 和 $g\left(\vec{\rho}\right)$ 的实现方式。

#### $\phi\left(\vec{r}\right)$ 的计算：

从刷怪机制层面分析，变量 $\vec{I}$ 的分布相当于：先在区块中以均匀概率随机抽取一个方块 $\vec{r} = \left(X, Z\right)$ 坐标，再根据该坐标对应的有效高度范围均匀随机抽取一个高度 $Y$ ，相应的三维方块坐标 $\left(X, Y, Z\right)$ 即为刷怪尝试的起始点。由此可得出 $\phi\left(\vec{r}\right)$ 的计算方式：

$$ \phi\left(\vec{r}\right) = \frac{1}{L^2\cdot h\left(\vec{r}\right)}$$

其中， $L = 16$ 是区块的水平截面边长，也是区段的边长； $h\left(\vec{r}\right)$ 表示水平方块坐标 $\vec{r} = \left(x, z\right)$ 对应的有效高度范围。

引入辅助函数 $y_m\left(\vec{r}\right)$ 表示坐标 $\vec{r} = \left(x, z\right)$ 对应的最高遮光方块 $y$ 坐标； 引入记号 $C\left(\vec{r}\right)$ 表示 $\vec{r}$ 所在区块的全部 $\left(x, z\right)$ 坐标的集合。

在1.12.2中， $h\left(\vec{r}\right)$ 定义如下：

$$ h\left(\vec{r}\right)
= L \cdot \left\lceil\frac{1}{L}
\left(\max_{\vec{\omega}\in C\left(\vec{r}\right)}\left\lbrace
    y_m\left(\vec{\omega}\right)
\right\rbrace + 2\right)\right\rceil $$

其含义可以解释为： $\vec{r}$ 所在区块的最高遮光方块高度 $+1$ 后的高度（最高可刷怪高度）所在的区段总高度。

例如，假设一个区块的最高遮光方块高度为 $30$ ，则其最高可刷怪高度为 $31$ ，由于 $31 \in \left[16, 31\right]$ ，所以此高度位于第 $2$ 个区段， $h = 2 \times 16 = 32$ 。

在实际运行中，刷怪高度选取的范围是 $\left[0, h-1\right] \cap \mathbb{Z}$ ，恰好有 $h$ 种可能的取值。

实际计算方法可以总结为：

$$ \phi\left(\vec{r}\right) = \left(4096\cdot
\left\lceil\frac{1}{16}
\left(\max_{\vec{\omega}\in C\left(\vec{r}\right)}\left\lbrace
    y_m\left(\vec{\omega}\right)
\right\rbrace + 2\right)\right\rceil\right)^{-1}$$

#### $g\left(\vec{\rho}\right)$ 的计算：

$g\left(\vec{\rho}\right)$ 本质上是与 $\left(x, z\right)$ 有关的二元函数，但其中的两个坐标轴对概率的影响是互相独立的，因此可以在一维坐标的环境下分别考虑变量 $A_k = \left(X_k, Z_k\right)$ 的分布。下面以 $X_k$ 为例：

考虑游走范围的位置相对性，此处关心的变量其实是 $X = X_k - X_{k+1}$ 对刷怪机制进行简单分析，不难得出一个众所周知的规律： $X$ 并不是均匀分布的，而是两个独立的（离散型）均匀分布的线性叠加。具体表现如下：

$$ X = X_P - X_N;\space X_P \sim U\left[0, 5\right];\space X_N \sim U\left[0, 5\right] $$

将游走半径记作  $\lambda = 6$ 。分布情况如下：

$$ P\left\lbrace X = t \right\rbrace
= \frac{\lambda - \left|t\right|}{\lambda ^2} ;\space \left(-\lambda < t < \lambda\right) \wedge \left(t\in\mathbb{Z}\right) $$

$Z$ 方向的分布有类似的规律：

$$ P\left\lbrace Z = t \right\rbrace
= \frac{\lambda - \left|t\right|}{\lambda ^2} ;\space \left(-\lambda < t < \lambda\right) \wedge \left(t\in\mathbb{Z}\right) $$

将两者结合起来，可以得到 $g\left(\vec{\rho}\right)$ 的表达式：

$$ g\left(\vec{\rho}\right)
= g\left(x, z\right)
= P\left\lbrace X = x \right\rbrace \cdot P\left\lbrace Z = z \right\rbrace $$

$$ g\left(\vec{\rho}\right)
= \frac{\left(\lambda - \left|x\right|\right)
\left(\lambda - \left|z\right|\right)}{\lambda ^4}
= \frac{\left(6 - \left|x\right|\right)
\left(6 - \left|z\right|\right)}{1296}
;\space \left(\vec{\rho} \in \delta _0\right) $$

#### 总论：

WIP

# 相关资料

