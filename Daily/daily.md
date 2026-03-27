# 未分类的记录

## 地狱门计算器

实体（不含生存玩家）通过地狱门进行传送的机制。

此处以物品为例。

> （并不是CPP）

```cpp
...
World world;
...
tick = 0;
world.updateEntities();
├── for (Entity entity : loadedEntityList):
│   ├── // entity instanceOf EntityItem entityItem
│   ├── world.updateEntity(entity);
│   │   ├── world.updateEntityWithOptionalForce(entity, true);
│   │   │   ├── entityItem.onUpdate();
│   │   │   │   ├── entity.onUpdate();
│   │   │   │   │   ├── entity.onEntityUpdate();
│   │   │   │   │   │   ├── if (inPortal/*false*/) else
│   │   │   │   │   │   ├── 
│   │   │   │   ├── entity.move(SELF, motionX, motionY, motionZ);
│   │   │   │   │   ├── entity.setEntityBoundingBox(nextBoundingBox);
│   │   │   │   │   ├── entity.doBlockCollisions();
│   │   │   │   │   │   ├── BlockPortal.onEntityCollision(world, blockPos, blockState, entity);
│   │   │   │   │   │   │   └── entity.inPortal = true;
└── 
```

## mspt的叠加

将随时间（各个gt）变化的mspt记为 $f\left(t\right)$ ，

$$
F\left(\omega\right) = \int_{-\infty}^{+\infty}{{
    f\left(t\right)\mathrm{e}^{-\mathrm{i}\omega t}
}{\mathrm{d}t}}
$$

设游戏运行的标准tick周期为 $M$ （通常为50ms），一段时间 $T$ 内的平均tps的定义为：

$$
U = \frac{T}{\int_{0}^{T}{{
    \max{\left\{M, f\left(t\right)\right\}}
}{\mathrm{d}t}}}
$$

这需要在 $f\left(t\right)$ 中截断小于 $M$ 的值。如果 $f\left(t\right)$ 是多个mspt变量叠加的结果（例：多个机器同时运行），即：

$$
f\left(t\right) = \sum_{i=0}^{n}{f_i\left(t\right)}
$$

则 $f\left(t\right)$ 的截断情况很可能与任意 $f_i\left(t\right)$ 都不同。

例如以下叠加情况*：

$$
f\left(t\right) = \frac{1}{2\pi}\int_{-\infty}^{+\infty}{{
    F\left(\omega\right)\mathrm{e}^{\mathrm{i}\omega t}
}{\mathrm{d}\omega}}
$$

$$
F\left(\omega\right) = \int_{-\infty}^{+\infty}{{
    f\left(t\right)\mathrm{e}^{-\mathrm{i}\omega t}
}{\mathrm{d}t}}
$$

## 平均存活时间

自然生成速率 $u$ ，存活时间 $l$ （lifetime），生物容量 $m$ ，产出速率 $w$ 。

从全空到自然填满，时间相同，各类生物占用容量：

$$m_i\propto u_i$$

如果初始为均等状态，则每次移除任意数量任意种类生物，剩余容量仍然由各类生物按权重均匀填满。

$$
U = \sum_{i=1}^{n}{u_i}
$$

在上限已满的情况下，移除的总速率和生成的总速率相等，该速率为 $W$ 。

$$m_i = w_i l_i$$

$$ w_i = k u_i$$

其中 $k$ 与 $i$ 无关。

$$
M = \sum_{i=1}^{n}{m_i} = k \sum_{i=1}^{n}{u_i l_i}
= \sum_{i=1}^{n}{w_i l_i}
$$

$$
W = \sum_{i=1}^{n}{w_i} = \sum_{i=1}^{n}{\frac{m_i}{l_i}}
$$

由于 $M, l_i, u_i$ 均已知，可以得出待定系数 $k$ 的表达式：

$$
k = \frac{M}{\sum_{i=1}^{n}{u_i l_i}}
$$

从而可以得出每一个 $m_i$ 的表达式：

$$
m_i = k u_i l_i = M\cdot\frac{u_i l_i}{\sum_{j=1}^{n}{u_j l_j}}
$$

平均存活时间（或称为等效存活时间）为 $L$ ，其定义式为：

$$
L = \frac{M}{W}
$$

根据上述推导结果，注意到：

$$
W = \sum_{i=1}^{n}{\frac{m_i}{l_i}}
= M\cdot\sum_{i=1}^{n}{\frac{u_i}{\sum_{j=1}^{n}{u_j l_j}}}
= \frac{M\cdot\sum_{i=1}^{n}{u_i}}{\sum_{i=1}^{n}{u_i l_i}}
$$

$$
L = \frac{\sum_{i=1}^{n}{u_i l_i}}{\sum_{i=1}^{n}{u_i}}
$$

在统计结果中，一定时间内的生成速率与统计到的生物数量具有正比关系。设存活时间为 $l_i$ 且自然生成速率为 $u_i$ 的生物数量为 $c_i$ ，则 $L$ 的计算方法为：

$$
L = \frac{\sum_{i=1}^{n}{c_i l_i}}{\sum_{i=1}^{n}{c_i}}
$$

特别地，对于 $\left(\forall i\right)c_i = 1$ 的情况，即最简单的每个生物单独统计而不分组的情况，计算方法为：

$$
L = \frac{\sum_{i=1}^{n}{l_i}}{\sum_{i=1}^{n}{1}}
= \frac{1}{n}\sum_{i=1}^{n}{l_i}
$$

由此可见，统计意义上可以用来等效计算刷怪效率的平均存活时间就是每个生物存活时间的算术平均值。

## 存活时间的影响

考虑更改特定怪物种类存活时间对刷怪塔总效率的影响。

平均每个生物产出的物品数量 $c$ (count)，物品产出速率 $p$ (production) 。

如果简单粗暴地用物品产出总效率衡量刷怪塔效率，可以将 $c_i$ 视为怪物种类 $i$ 的权重。

同上：自然生成速率 $u$ ，存活时间 $l$ ，生物容量 $m$ ，产出速率 $w$ 。

自然刷怪速率 $u$ 可以通过carpet-mod的 `/spawn mocking` 来测量；产出速率 $w_i$ 指的是实际的刷怪速率。

刷怪塔物品产出 $P$ ：

$$
P = \sum_{i=1}^{n}{p_i} = \sum_{i=1}^{n}{c_i w_i}
$$

$$
\left(\forall i\right) w_i = k u_i = \frac{M u_i}{\sum_{j=1}^{n}{u_j l_j}}
$$

$$
P = k \sum_{i=1}^{n}{c_i u_i} = \frac{M \sum_{i=1}^{n}{c_i u_i}}{\sum_{i=1}^{n}{l_i u_i}}
$$

对于任意怪物种类 $j$ ，考虑其存活时间 $l_j$ 的变化对总效率 $P$ 的影响：

$$
\ln{P} = \ln{M} + \ln{\left(\sum_{i=1}^{n}{c_i u_i}\right)} - \ln{\left(\sum_{i=1}^{n}{l_i u_i}\right)}
$$

$$
\frac{\partial{P}}{P \partial{l_j}} = \frac{\partial \ln{P}}{\partial l_j} = - \frac{u_j}{\sum_{i=1}^{n}{l_i u_i}}
$$

$$
\frac{\partial P}{\partial l_j} = - \frac{P u_j}{\sum_{i=1}^{n}{l_i u_i}} = - \frac{M \sum_{i=1}^{n}{c_i u_i}}{\sum_{i=1}^{n}{l_i u_i}} \cdot \frac{u_j}{\sum_{i=1}^{n}{l_i u_i}}
$$

$$
\frac{\partial P}{\partial l_j} = - \frac{M \sum_{i=1}^{n}{c_i u_i}}{\left(\sum_{i=1}^{n}{l_i u_i}\right)^2}\cdot u_j
$$

## 测试：固体方块对刷怪速率的影响

测试条件：
 - 2区段高度
 - 1*1刷怪面积
 - 使用carpet-mod `/spawn rates hostile 50` 调整生成速率
 - 沙漠群系
 - 非史莱姆区块
 - 使用循环命令方块 `/kill @e` 击杀

无固体方块：

- 物品产出：11.61 k/h (SE=0.24 k/h)
- 刷怪速率：2.3 m/t, 95.3-/4.7+, 0.12 s/att

|固体方块位置|物品产出|标准误差|相对误差|产出缺损|传递误差|
|:-:|:-:|:-:|:-:|:-:|:-:|
|无|2.37 k/h|0.23 k/h|9.86%|0|0|
|外围1圈|1.80 k/h|0.17 k/h|9.61%|-0.57 k/h|0.29 k/h|
|侧面1格-单向|2.30 k/h|0.21 k/h|9.07%|-0.07 k/h|0.31 k/h|
|侧面1格-四向|1.84 k/h|0.18 k/h|9.66%|-0.53 k/h|0.29 k/h|
|对角1格-四向|2.04 k/h|0.19 k/h|9.49%|-0.33 k/h|0.30 k/h|
|对角1格-单向|2.07 k/h|0.20 k/h|9.53%|-|-|

## 有卡怪损耗的存活时间折算

在拌线钩1gt固有延迟内生成导致卡怪，是灵魂沙预伤害架构的固有缺陷。如果不解决这一问题导致被卡住的怪物的掉落物无法收集，那么实际的效率损失将大于单纯的平均存活时间延长导致的效率损失。为了便于数值参考，需要考虑引入一个等效的额外存活时间，用来折算掉落物残留导致的额外效率损失。
