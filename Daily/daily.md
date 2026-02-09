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
