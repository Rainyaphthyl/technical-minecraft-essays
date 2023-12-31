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
