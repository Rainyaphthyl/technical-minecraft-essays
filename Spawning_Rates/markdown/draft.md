# 刷怪速率的计算方法简述

## 错误的尝试

考虑以下随机变量：

- $\xi$ ，在某一位置，进行刷怪步骤对应事件的频次；此处“事件”可以是“最终生成”、“游走到达”等。
- $\vec{S}_k$ ，剩余 $k$ 次游走机会时，游走到达的位置，对应的游走位移是 $\vec{V}_k = \vec{S}_k - \vec{S}_{k+1}$ ； $k=0$ 时表示最终生成位置 $\vec{S}_0$ ，可默认简记为 $\vec{S}$ 。

本文的核心目的是在给定任意方块坐标 $\vec{r}$ 的条件下计算该坐标的平均刷怪次数：

$$ s\left(\vec{r}\right) = E\left(\xi\middle|\vec{S}_0 = \vec{r}\right) $$

方块坐标可以是二维的 $\left(x, z\right)$ 坐标，也可以是三维坐标。

理论上也可以计算 $\xi$ 的概率分布： $ P\left(\xi = t\middle|\vec{S}_0=\vec{r}\right) $ 。

这其实也是有意义的，有助于深入了解刷怪速率的波动情况，并推广到区域刷怪速率的情况；但比较难以计算。

可以做一些简单的定性分析： $\xi$ 的样本空间是 $\left\{0, 1, 2, \dots, n\right\}$ ，也可以直接认定为 $\mathbb{N}$ ——如果多个刷怪游走路径都在相同的 $\vec{S} = \vec{r}$ 处刷出生物， $\xi$ 就会取到很大的值。

在实际的刷怪过程中，生物需要进行碰撞检查，这就涉及某个版本差异：在1.9~1.12.2，生物可以在实体中生成，实体碰撞检查实际上失效了，因此显然有 $P\left(\xi \geq 2\right) > 0$ 。

> 但在1.13+，碰撞检查可以正常工作，生物不能在已经生成的生物占据的空间中生成，因此 $\xi$ 在最终生成的阶段只可能取到 $0$ 或 $1$ (待验证)

$$
E\left(\xi\middle|\vec{S}_0 = \vec{r}\right) \\
= \sum_{t=0}^{\infty}{t\cdot P\left(\xi=t\middle|\vec{S}_0 = \vec{r}\right)} \\
= \sum_{t=0}^{\infty}{t\cdot \frac{P\left(\vec{S}_0 = \vec{r} \middle|\xi=t\right)\cdot P\left(\xi=t\right)}{P\left(\vec{S}_0 = \vec{r}\right)}} \\
= \sum_{t=0}^{\infty}{t\cdot \frac{P\left(\vec{S}_0 = \vec{r} \middle|\xi=t\right)}{P\left(\vec{S}_0 = \vec{r}\right)}\cdot \sum_{\vec{\rho}\in\delta _0}P\left(\xi=t\middle|\vec{S}_1 = \vec{r} + \vec{\rho}\right)\cdot P\left(\vec{S}_1 = \vec{r} + \vec{\rho}\right)} \\
= \sum_{\vec{\rho}\in\delta _0}{\sum_{t=0}^{\infty}{t\cdot P\left(\vec{S}_0 = \vec{r} \middle|\xi=t\right)\cdot P\left(\xi=t\middle|\vec{S}_1 = \vec{r} + \vec{\rho}\right)\cdot\frac{P\left(\vec{S}_1 = \vec{r} + \vec{\rho}\right)}{P\left(\vec{S}_0 = \vec{r}\right)}}} \\
= \sum_{\vec{\rho}\in\delta _0}{P\left(\vec{S}_0 = \vec{r}\right)\cdot E\left(\xi\middle|\vec{S}_1 = \vec{r} + \vec{\rho}\right)\cdot\frac{P\left(\vec{S}_1 = \vec{r} + \vec{\rho}\right)}{P\left(\vec{S}_0 = \vec{r}\right)}} \\
= \begin{cases}
\end{cases}
$$

## 重构

考虑以下随机变量：

- $\xi$ ，在某一位置，进行刷怪步骤对应事件的频次；此处“事件”可以是“最终生成”、“游走到达”等。

将相应位置的方块坐标记为 $\vec{r}$ ，剩余游走次数记为 $w$ ，则 $\xi$ 应是与 $\vec{r}, w$ 都有关的量。可以考虑四维或三维数组 ${\left\{\xi\left(\vec{r}, w\right)\right\}}$ （方块坐标包含三维，或者只考虑二维 $\left(x, z\right)$ 坐标），其中每个元素都是一个随机变量。

当 $w = 0$ 时，游走次数均已用完，此时的坐标 $\vec{r}$ 即表示生物生成位置的坐标，相应的 $\xi\left(\vec{r}, 0\right)$ 表示生物在 $\vec{r}$ 处生成的数量。

本文关注的值是 $\xi$ 的分布律或期望：

$$
\begin{equation}
f_\xi\left(t; \vec{r}, s\right) = P\left\{\xi\left[\vec{r}, w\right] = t\right\}
\end{equation}
$$

$$
\begin{equation}
s\left(\vec{r}, w\right) = E\left\{\xi\left[\vec{r}, w\right]\right\}
\end{equation}
$$

为方便描述，对游走次数剩余0的情况定义以下专门的符号：

$$
\begin{equation}
s\left(\vec{r}\right) = s\left(\vec{r}, 0\right)
\end{equation}
$$

考虑 $\left(\forall \vec{r} \forall w\right) \xi\left[\vec{r}, w\right]$ 之间的关系；为此需要先引入一个用于中间连接的随机变量 $\eta\left[\vec{r}, w, \vec{\rho}\right]$ ，表示刷怪过程以 $\vec{r}$ 为起点游走到 $\left(\vec{r} + \vec{\rho}\right)$ 的次数。 $\psi$ 仅用于描述某一次游走的位移量，与 $\xi$ 无关。

$$
\begin{equation}
\lambda = 6
\end{equation}
$$

$$
\begin{equation}
a\left(\vec{\rho}\right)
= \frac{\left(\lambda - \left|x\right|\right)
\left(\lambda - \left|z\right|\right)}{\lambda ^4}
= \frac{\left(6 - \left|x\right|\right)
\left(6 - \left|z\right|\right)}{1296}
;\space \left(\vec{\rho} \in \delta _0\right)
\end{equation}
$$

$$
\begin{equation}
a\left(\vec{\rho}\right)
= \begin{cases}
\frac{\left(\lambda - \left|x\right|\right)\left(\lambda - \left|z\right|\right)}{\lambda ^4} &, \vec{\rho}\in\delta _0
\\ 0 &, \text{else}
\end{cases}
\end{equation}
$$

$$
\begin{equation}
P\left\{\eta\left[\vec{\rho}\right] = k\right\}
= P\left\{\eta _{\left[\vec{r}, w, \vec{\rho}\right]} = k\right\}
= \begin{cases}
a\left(\vec{\rho}\right) &, k = 1
\\ 1 - a\left(\vec{\rho}\right) &, k = 0
\\ 0 &, \text{else}
\end{cases}
\end{equation}
$$

$$
\begin{equation}
E\left\{\eta\left[\vec{\rho}\right]\right\}
= a\left(\vec{\rho}\right)
\end{equation}
$$

根据定义，可以得到以下关系：

$$
\begin{equation}
\xi\left[\vec{r}, w\right] =
\sum_{\vec{\rho}\in\delta _{w+1}}{\xi\left[\vec{r}+\vec{\rho}, w+1\right]\cdot\eta\left[\vec{r}+\vec{\rho}, w+1, -\vec{\rho}\right]}
\end{equation}
$$

以上关系仅在某些条件下成立，但这里暂未给出所需的条件。

设游走次数 $\psi$ 的全部可能取值的范围是 $\left[\alpha, \beta\right] \cap \mathbb{N}$ ，其中 $1\leq\alpha\leq\beta;\space \alpha, \beta \in \mathbb{N}$ 。

> 注意：此处的游走次数是随机变量，并不是上述 $\xi$ 定义中的实际剩余游走次数 $w$ 。

$w = 0$ 表示正式刷出生物。设 $\psi$ 在一次事件中实际取值为 $c$ ，则 $w = c$ 表示初次游走的开始、刷怪尝试的初始情况。

$$
\begin{equation}
\delta\left(\vec{r}_i\right) = \left\{ \vec{r}_j \middle|
\left(\vec{r}_j - \vec{r}_i\right) \in
\left(\left[-5, +5\right]^2 \cap \mathbb{Z}^2\right) \right\}
\end{equation}
$$

$$
\begin{equation}
\delta = \delta _0 = \delta\left(\vec{0}\right)
= \left(\left[-5, +5\right]^2 \cap \mathbb{Z}^2\right)
\end{equation}
$$

考虑初次游走起点对应的 $\xi$ 的分布（单组刷怪尝试，不考虑生物种类）：

$$
\begin{equation}
\begin{align*}
P\left\{\xi\left[\vec{r}, c\right]=k\middle|\psi=c\right\}
&= \begin{cases}
u\left(\vec{r}\right) &, k = 1
\\ 1 - u\left(\vec{r}\right) &, k = 0
\\ 0 &, \text{else}
\end{cases}
\end{align*}
\end{equation}
$$

$$
\begin{equation}
u\left(\vec{r}\right) = \frac{1}{L^2\cdot h\left(\vec{r}\right)}
\end{equation}
$$

$$
\begin{equation}
h\left(\vec{r}\right)
= L \cdot \left\lceil\frac{1}{L}
\left(\max_{\vec{\omega}\in C\left(\vec{r}\right)}\left\lbrace
y_m\left(\vec{\omega}\right)
\right\rbrace + 2\right)\right\rceil
\end{equation}
$$

$$
\begin{equation}
L = 16;\space \vec{r} = \left(x, z\right)
\end{equation}
$$

$$
\begin{equation}
C\left(\vec{r}\right)
= \left\lbrace \left(m, n\right)\in \mathbb{Z}^2 \middle|
\left(
\left\lfloor \frac{m}{L} \right\rfloor = \left\lfloor \frac{x}{L} \right\rfloor
\right)
\wedge
\left(
\left\lfloor \frac{n}{L} \right\rfloor = \left\lfloor \frac{z}{L} \right\rfloor
\right)
\right\rbrace
\end{equation}
$$

显然可以得到：

$$
\begin{equation}
\begin{align*}
E\left\{\xi\left[\vec{r}, c\right]\middle|\psi=c\right\}
&= u\left(\vec{r}\right)
\\&= \left(L^3
\left\lceil\frac{1}{L}
\left(\max_{\vec{\omega}\in C\left(\vec{r}\right)}\left\lbrace
y_m\left(\vec{\omega}\right)
\right\rbrace + 2\right)\right\rceil\right)^{-1}
\end{align*}
\end{equation}
$$

$$
\begin{equation}
P\left\{\psi = c\right\}
= \begin{cases}
\frac{1}{\beta - \alpha + 1} &, \alpha \leq c \leq \beta
\\ 0 &, \text{else}
\end{cases}
\end{equation}
$$

游走总次数是另一个随机变量；限定 $w \leq c$ ，考虑对生成数量的期望进行分解：

$$
\begin{equation}
\begin{align*}
E\left\{\xi _{\left[\vec{r}, w\right]}\right\}
= \sum _{c=w}^{\beta}{E\left\{\xi _{\left[\vec{r}, w\right]}\middle|\psi=c\right\}\cdot P\left\{\psi = c\right\}}
\\= \frac{1}{\beta-\alpha+1}\cdot\sum _{c=\max{\left\{\alpha, w\right\}}}^{\beta}{E\left\{\xi _{\left[\vec{r}, w\right]}\middle|\psi=c\right\}}
\end{align*}
\end{equation}
$$

$$
\begin{equation}
\begin{align*}
E\left\{\xi _{\left[\vec{r}, w\right]}\middle|\psi=c\right\}
&= \begin{cases}
\sum_{\vec{\rho}\in\delta _0}{E\left\{\xi _{\left[\vec{r}+\vec{\rho}, w+1\right]}\cdot\eta _{\left[-\vec{\rho}\right]}\middle|\psi=c\right\}} &, 0 \leq w < c
\\ u\left(\vec{r}\right) &, w = c
\end{cases}
\\&= \begin{cases}
\sum_{\vec{\rho}\in\delta _0}{a\left(\vec{\rho}\right)\cdot E\left\{\xi _{\left[\vec{r}+\vec{\rho}, w+1\right]}\middle|\psi=c\right\}} &, 0 \leq w < c
\\ u\left(\vec{r}\right) &, w = c
\end{cases}
\end{align*}
\end{equation}
$$

## 第二次重构 / 第三次建构

> 设置随机变量时，按照随机事件已经发生的情况进行考虑……

$\psi _a\left(\vec{r}\right)$ ：位置 $\vec{r}$ 处开始的游走总次数，其中 $1 \leq g \leq 3$ ，表示一次刷怪循环中的3组刷怪尝试。

$\phi \left(\vec{r}\right)$ ：位置 $\vec{r}$ 处成为刷怪尝试起点的次数，只能取0或1.

不要试图考虑 $\psi$ 或 $\phi$ 的分布；这里只考虑数学期望。

$\psi$ 与 $\phi$ 互相独立，实际应用的随机变量是 $\psi _g\left(\vec{r}\right)\cdot\phi\left(\vec{r}\right)$ 。

$$
\begin{equation}
P\left\{\psi _g\left(\vec{r}\right) = k\right\}
= \begin{cases}
\frac{1}{\beta - \alpha + 1} &, \alpha \leq k \leq \beta
\\ 0 &, \text{else}
\end{cases}
\end{equation}
$$

$\psi _1, \psi _2, \psi _3$ 互相独立。

$$
\begin{equation}
P\left\{\phi\left(\vec{r}\right) = k\right\}
= \begin{cases}
u\left(\vec{r}\right) &, k = 1
\\ 1 - u\left(\vec{r}\right) &, k = 0
\\ 0 &, \text{else}
\end{cases}
\end{equation}
$$

对于 $\phi\left(\vec{r}_1\right), \phi\left(\vec{r}_2\right)$ ：
1. 如果 $\vec{r}_1 = \vec{r}_2$ ，则 $\phi\left(\vec{r}_1\right), \phi\left(\vec{r}_2\right)$ 是同一个随机变量。
2. 如果 $\vec{r}_1 \neq \vec{r}_2$ ，且 $\vec{r}_1, \vec{r}_2$ 在不同区块中，则 $\phi\left(\vec{r}_1\right), \phi\left(\vec{r}_2\right)$ 互相独立。
3. 如果 $\vec{r}_1 \neq \vec{r}_2$ ，且 $\vec{r}_1, \vec{r}_2$ 在同一区块中，则 $\phi\left(\vec{r}_1\right), \phi\left(\vec{r}_2\right)$ 不独立。

虽然这种关系很复杂，但通过常识可以猜测 $E\left\{\phi\left(\vec{r}\right)\right\}$ 的表达式应该是比较均匀且通俗易懂的，并不影响简单计算。

对于上述第2种情况：

$$
\begin{equation}
\begin{align*}
& P\left\{\left(\phi\left(\vec{r}_1\right)=k_1\right)\cap\left(\phi\left(\vec{r}_2\right)=k_2\right)\right\}
\\ =& \begin{cases}
u\left(\vec{r}_1\right) &, k_1=1\wedge k_2=0
\\ u\left(\vec{r}_2\right) &, k_1=0\wedge k_2=1
\\ 1-u\left(\vec{r}_1\right)-u\left(\vec{r}_2\right) &, k_1=0\wedge k_2=0
\\ 0 &, \text{else}
\end{cases}
\end{align*}
\end{equation}
$$

$$
\begin{equation}
\begin{align*}
& P\left\{\bigcap_{i=1}^{\left|C\left(\vec{r}_i\right)\right|}{\left(\phi\left(\vec{r}_i\right)=k_i\right)}\right\}
\\ =& \begin{cases}
u\left(\vec{r}_i\right) &, \left(k_i=1\right)\wedge\left(\bigwedge_{j\neq i}{\left(k_j=0\right)}\right)
\\ \dots &, \dots
\\ 1-\sum_{i}{u\left(\vec{r}_i\right)} &, \bigwedge_{i}{\left(k_i=0\right)}
\\ 0 &, \text{else}
\end{cases}
\end{align*}
\end{equation}
$$

对于任何情况：

$$
\begin{equation}
E\left\{\phi\left(\vec{r}\right)\right\}=u\left(\vec{r}\right)
\end{equation}
$$

考虑 $\xi$ 的递推关系：

$$
\begin{equation}
\zeta _g\left(\vec{r}, c\right)
= \begin{cases}
1 &, \psi _g\left(\vec{r}\right) = c
\\ 0 &, \psi _g\left(\vec{r}\right) \neq c
\end{cases}
\end{equation}
$$

$$
\begin{equation}
\begin{align*}
\xi\left(\vec{r}, w\right)
=& \begin{cases}
\sum_{\vec{\rho}\in\delta _0}{\sum _{i=1}^{\xi\left(\vec{r}-\vec{\rho}, w+1\right)}{\eta _i\left(\vec{\rho}\right)}} &, 0 \leq w < \alpha
\\ \theta _I\left(\vec{r}, w\right) + \theta _J\left(\vec{r}, w\right) &, \alpha \leq w < \beta
\\ \phi\left(\vec{r}\right)\cdot\sum_{g=1}^{3}{\zeta _g\left(\vec{r}, w\right)} &, w = \beta
\end{cases}
\end{align*}
\end{equation}
$$

$$
\begin{equation}
\begin{cases}
\theta _I\left(\vec{r}, w\right) = \sum_{\vec{\rho}\in\delta _0}{\sum _{i=1}^{\xi\left(\vec{r}-\vec{\rho}, w+1\right)}{\eta _i\left(\vec{\rho}\right)}}
\\ \theta _J\left(\vec{r}, w\right) = \phi\left(\vec{r}\right)\cdot\sum_{g=1}^{3}{\zeta _g\left(\vec{r}, w\right)}
\end{cases}
\end{equation}
$$

考虑上述随机变量的期望：

$$
\begin{equation}
\begin{align*}
s\left(\vec{r}, w\right)
=& \begin{cases}
\sum_{\vec{\rho}\in\delta _0}{s\left(\vec{r}-\vec{\rho}, w+1\right)\cdot a\left(\vec{\rho}\right)} &, 0 \leq w < \alpha
\\ I\left(\vec{r}, w\right) + J\left(\vec{r}, w\right) &, \alpha \leq w < \beta
\\ u\left(\vec{r}\right)\cdot\frac{3}{\beta-\alpha+1} &, w = \beta
\end{cases}
\end{align*}
\end{equation}
$$

$$
\begin{equation}
\begin{align*}
s\left(\vec{r}, w\right)
=& \begin{cases}
\sum_{\vec{\rho}\in\delta _0}{s\left(\vec{r}-\vec{\rho}, w+1\right)\cdot a\left(\vec{\rho}\right)} &, 0 \leq w < \alpha
\\ \sum_{\vec{\rho}\in\delta _0}{s\left(\vec{r}-\vec{\rho}, w+1\right)\cdot a\left(\vec{\rho}\right)} + u\left(\vec{r}\right)\cdot\frac{3}{\beta-\alpha+1} &, \alpha \leq w < \beta
\\ u\left(\vec{r}\right)\cdot\frac{3}{\beta-\alpha+1} &, w = \beta
\end{cases}
\end{align*}
\end{equation}
$$

$$
\begin{equation}
\begin{align*}
I\left(\vec{r}, w\right)
&= \sum_{\vec{\rho}\in\delta _0}{E\left\{\xi\left(\vec{r}-\vec{\rho}, w+1\right)\right\}\cdot E\left\{\eta\left(\vec{\rho}\right)\right\}}
\\ &= \sum_{\vec{\rho}\in\delta _0}{s\left(\vec{r}-\vec{\rho}, w+1\right)\cdot a\left(\vec{\rho}\right)}
\end{align*}
\end{equation}
$$

$$
\begin{equation}
\begin{align*}
J\left(\vec{r}, w\right)
&= E\left\{\phi\left(\vec{r}\right)\right\}\cdot\sum_{g=1}^{3}{E\left\{\zeta _g\left(\vec{r}, w\right)\right\}}
\\ &= u\left(\vec{r}\right)\cdot\frac{3}{\beta-\alpha+1}
\\ &= \frac{3\cdot u\left(\vec{r}\right)}{\beta-\alpha+1}
\\ & \left(\alpha \leq w \leq \beta\right)
\end{align*}
\end{equation}
$$

以上公式适用于不考虑生物种类随机性的情况。
