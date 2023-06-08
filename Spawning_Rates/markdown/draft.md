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

考虑 $\left(\forall \vec{r} \forall w\right) \xi\left[\vec{r}, w\right]$ 之间的关系；为此需要先引入一个用于中间连接的随机变量 $\eta\left[\vec{r}, w, \vec{\rho}\right]$ ，表示刷怪过程以 $\vec{r}$ 为起点游走到 $\left(\vec{r} + \vec{\rho}\right)$ 的次数。此随机变量的设置建立在 $\xi\left[\vec{r}, w\right]$ 的基础上，即已经包含了对于“以 $\vec{r}$ 为终点”这一前置随机事件的描述。

根据定义，可以得到以下关系：

$$
\begin{equation}
\xi\left[\vec{r}, w\right] =
\sum_{\vec{\rho}\in\delta _{w}}{\eta\left[\vec{r}, w, \vec{\rho}\right]}
\end{equation}
$$

$$
\begin{equation}
\xi\left[\vec{r}, w\right] =
\sum_{\vec{\rho}\in\delta _{w+1}}{\eta\left[\vec{r}+\vec{\rho}, w+1, -\vec{\rho}\right]}
\end{equation}
$$

$$
\begin{equation}
\xi\left[\vec{r}, w\right] = \begin{cases}
\sum_{\vec{\rho}\in\delta _{w+1}}{\eta\left[\vec{r}+\vec{\rho}, w+1, -\vec{\rho}\right]} &, 0 \leq w \leq \alpha - 1 \\
\end{cases}
\end{equation}
$$

以上关系仅在某些条件下成立，但这里暂未给出所需的条件。

设游走次数 $\psi$ 的全部可能取值的范围是 $\left[\alpha, \beta\right] \cap \mathbb{N}$ ，其中 $1\leq\alpha\leq\beta;\space \alpha, \beta \in \mathbb{N}$ 。

> 注意：此处的游走次数是随机变量，并不是上述 $\xi$ 定义中的实际剩余游走次数 $w$ 。

$w = 0$ 表示正式刷出生物。设 $\psi$ 在一次事件中实际取值为 $c$ ，则 $w = c$ 表示初次游走的开始、刷怪尝试的初始情况。

$$
\begin{equation}
\xi\left[\vec{r}, w\right] =
\sum_{\vec{\rho}\in\delta _{w+1}}{\eta\left[\vec{r}+\vec{\rho}, w+1, -\vec{\rho}\right]} \space, \text{if} \left(0 \leq w \leq \psi - 1\right)
\end{equation}
$$
