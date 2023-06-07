# 刷怪速率的计算方法简述

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
= \sum_{t=0}^{\infty}{t\cdot P\left(\vec{S}_0 = \vec{r} \middle|\xi=t\right)\cdot \sum_{\vec{\rho}\in\delta _0}P\left(\xi=t\middle|\vec{S}_1 = \vec{r} + \vec{\rho}\right)\cdot\frac{P\left(\vec{S}_1 = \vec{r} + \vec{\rho}\right)}{P\left(\vec{S}_0 = \vec{r}\right)}} \\
= \begin{cases}
\end{cases}
$$
