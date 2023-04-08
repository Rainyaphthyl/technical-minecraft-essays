# [1.12.2] 树苗催熟效率的一种理论分析

**摘要：**

**关键词：**「」

# **引言**

很多树厂需要使用发射器喷出骨粉并催熟树苗，树苗催熟效率是影响树厂输出效率的一个重要因素。在实际的树厂架构与电路设计中，发射器的布局与运行受到各种因素的制约，不可能为了催熟效率的极端优化而不计其他影响，因此需要对各种布局方案的催熟效率进行量化评估，以便在设计中选取恰当的方案。发射器的数量、骨粉发射的频率、相位（错峰催熟）、树木限高等因素都有可能对催熟效率产生影响，需要进一步的分析。

在未经特殊说明的情况下，本文所有内容仅考虑1.12.2的游戏环境，不保证任何其他版本的适用性；对于树苗生长的分析仅考虑骨粉催熟的影响，不考虑随机刻的影响。

# **树苗催熟概率分析**

## **树苗的催熟过程**

树苗的生长阶段 `STAGE` 有 `0` 和 `1` 两个取值。初始状态下取值为 `0` ；当受到骨粉作用时，有 $\alpha = 0.45$ 的概率进行以下尝试：

- 如果当前生长阶段为 `0` ，则进入下一个阶段，即生长阶段设置为 `1` 。
- 如果当前生长阶段为 `1` ，则进行树木生长尝试：程序随机选定树木即将生成的高度等结构参数，再根据树苗周围相关方块判断树木能否成功生成。如果尝试成功，则生成相应的树木结构，生长完毕；否则树苗维持当前状态。

在「树木生长尝试」的过程中，给定树苗周围的方块结构，将树苗成功

本文将生长阶段从 `0` 到 `1` 的过程通俗地称为「预催熟」，将生长阶段为 `1` 的树苗生成树木的过程称为树木的「生成」，这两个过程依次组成完整的树苗「催熟」过程。

根据上文定义，符合条件的树苗被骨粉作用一次时完成预催熟的概率为 $\alpha$ ，完成转化的概率为 $\alpha\beta$ 。其中 $\alpha = 0.45$ 为常量，而 $\beta$ 取决于树苗种类、限高结构等具体因素。

预催熟过程不受到树木生成检查的限制，也可以将其视为一种特殊的生成过程，其生长尝试成功率为 $\beta = 1$ 。至此，预催熟与生成的成功概率可以用形式相同的 $\alpha\beta$ 来描述。

将「预催熟」和「生成」统称为「转化」，则一次「催熟」过程包含两次「转化」过程。两个转化过程可以用相同的概率模型进行分析，将两个过程结合起来即可得到树苗催熟过程的概率模型。

## **树苗转化过程的概率模型**

在树苗的任意一次转化过程中，沿用上文对 $\alpha, \beta$ 的定义，将树苗成功转化时消耗过的骨粉数量记为随机变量 $\xi$ 。

树苗经过恰好 $n$ 次催熟时成功转化，意味着前 $\left(n-1\right)$ 次过程全部转化失败，第 $n$ 次过程转化成功。此过程显然必须满足 $n \geq 1$ ，由此很容易得出以下概率分布表达式：

$$ b\left(n\right) = P\left\lbrace\xi = n\right\rbrace = \left(1 - \alpha\beta\right)^{n-1}\cdot \alpha\beta, \left(n \in \mathbb{N}^{+}\right) $$

不加证明地给出以下引理：

$$ S\left(a, m\right) = \sum_{k=0}^{\infty}{\left(a^k\prod_{r=1}^{m}{\left(k+r\right)}\right)} = \frac{m!}{\left(1-a\right)^{m+1}} $$

由此考虑 $\xi$ 的均值与方差、标准差：

$$ E\left(\xi\right) = \sum_{k=1}^{\infty}{k\cdot b\left(k\right)} = \alpha\beta\cdot\sum_{k=1}^{\infty}{k\left(1 - \alpha\beta\right)^{k-1}} = \frac{1}{\alpha\beta} $$

$$ D\left(\xi\right) = E\left(\xi^2\right) - \left(E\left(\xi\right)\right)^2 = \frac{1 - \alpha\beta}{\left(\alpha\beta\right)^2} $$

$$ \sigma\left(\xi\right) = \sqrt{D\left(\xi\right)} = \frac{\sqrt{1 - \alpha\beta}}{\alpha\beta} $$

## **单树苗催熟过程的概率模型**

对于横截面为 $1\times1$ 的树木类型，其预催熟与生成的过程必然发生在同一个树苗上。将单树苗的两次转化过程组合起来，考虑其催熟过程的概率模型。

考虑骨粉使用 $n$ 次的情况，将各个「使用骨粉」的事件 $B_i \left(i \in \left\lbrace1, 2, \dots, n\right\rbrace \right)$ 划分为两个序列，每个序列的最后一个事件表示转化成功的情况，分别表示两次转化的全过程，并记 $n = n_1 + n_2$ ，其中 $n_i$ 表示第 $i$ 次转化过程的骨粉消耗量。

此时， $B_n$ 必然是唯一的树木成功生成的事件；树苗预催熟成功的事件可能是 $B_m \left(1 \leq m = n_1 \leq n - 1\right)$ 中的任意一个。事件 $B_m$ 将催熟过程分为两部分，其生长尝试成功率 $\beta_i$ 有所变化。

将两次转化过程的骨粉消耗量分别记为随机变量 ${\xi}_1, {\xi}_2$ ， $\xi = {\xi}_1 + {\xi}_2$ ，

$$ P\left\lbrace\xi = n\right\rbrace = \sum_{k=1}^{n-1}{P\left\lbrace{\xi}_1 = k\right\rbrace\cdot P\left\lbrace{\xi}_2 = n - k\right\rbrace} \\
= \sum_{k=1}^{n-1}{\left(1 - \alpha \beta _1\right)^{k-1}\cdot \alpha \beta _1 \cdot \left(1 - \alpha \beta _2\right)^{n-k-1}\cdot \alpha \beta _2} \\
= \alpha ^2 \beta _1 \beta _2 \left(1 - \alpha \beta _2\right)^{n-2} \cdot \sum_{k=1}^{n-1}{\left(\frac{1 - \alpha \beta _1}{1 - \alpha \beta _2}\right)^{k-1}} $$

其中 $n \in \mathbb{N}^+ \wedge n \geq 2$ 。

考虑实际情况，可以取 $\beta _1 = 1$ ，记 $\beta _2 = \beta$ ，则上式变为：

$$ P\left\lbrace\xi = n\right\rbrace 
= \alpha ^2 \beta \left(1 - \alpha\beta\right)^{n-2} \cdot \sum_{k=1}^{n-1}{\left(\frac{1 - \alpha}{1 - \alpha \beta}\right)^{k-1}} $$

继续化简得到以下结论：

$$ P\left\lbrace\xi = n\right\rbrace =
\begin{cases}
\alpha^2\left(n-1\right)\left(1-\alpha\right)^{n-2}, \space \beta = 1 \\
\frac{\alpha\beta}{1-\beta}\cdot\left(\left(1-\alpha\beta\right)^{n-1} - \left(1-\alpha\right)^{n-1}\right), \space 0 \leq \beta < 1
\end{cases} $$

## **四树苗催熟过程的概率模型**

# **含时的树苗催熟速率**

# 相关资料

- @{Author}, @{Author}. [15w31c](https://minecraft.fandom.com/zh/wiki/15w31a#%E7%94%9F%E7%89%A9_2)
