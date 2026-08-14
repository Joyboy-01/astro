---
title: 变分法和变分推断
publishDate: 2026-8-14 16:18:00
description: '变分'
tags:
  - 变分
language: '中文'
---

今天看了变分法的推导，来整理一下变分法和深度学习里变分推断的关系。

## 1. 变分法

### 最速降线

变分法的经典问题是最速降线：在重力作用下，质点从点 $A(0,0)$ 滑到点 $B(x_1, y_1)$，走哪条曲线用时最短？

这个问题和普通求极值问题的区别在于：我们要求的不是一个数 $x$，也不是一组参数 $\theta$，而是一个完整的函数 $y(x)$。

物理推导给出，下滑时间 $T$ 是路径 $y(x)$ 的泛函：

$$
\frac{1}{2}mv^2 = mgy
$$

得到：

$$
v = \sqrt{2gy}
$$

$$
ds = \sqrt{dx^2 + dy^2} = \sqrt{1 + (y')^2} \, dx
$$

$$
dt = \frac{ds}{v} = \frac{\sqrt{1 + (y')^2}}{\sqrt{2gy}} \, dx
$$

$$
T[y] = \int_0^{x_1} \sqrt{\frac{1 + (y')^2}{2gy}} \, dx
$$

常数 $\sqrt{2g}$ 不影响极值条件，可省略，得到：

$$
T[y] = \int_0^{x_1} \sqrt{\frac{1 + (y')^2}{y}} \, dx
$$

### 1.2 变分的核心思想

我们怎么判断一个函数 $y(x)$ 是否让 $T[y]$ 达到极值？

普通微积分是对于函数 $f(x)$，我们在极值点 $x_0$ 处给 $x$ 加一个微小变化 $\Delta x$，如果 $f$ 的变化率（导数）为 0，那么 $x_0$ 就是极值点。

变分法把同样的逻辑搬到"函数空间"里：

- 微分：让自变量 $x$ 经历微小变化，看函数值 $y$ 的变化率
- 变分：让函数 $y(x)$ 本身经历微小变化，看泛函 $T[y]$ 的变化率

为了在数学上实现"让函数 $y(x)$ 经历微小变化"，我们构造：

$$
y(x) \rightarrow y(x) + \epsilon \eta(x)
$$

这里：
- $\eta(x)$ 是一个任意选定的"扰动方向"，它是一个光滑函数，并且在边界处为 0（因为我们固定了起点和终点）
- $\epsilon$ 是一个很小的数，控制扰动的幅度

这个构造的本质是：把"求整个函数"的问题，强行变成"求一个数值 $\epsilon$"的问题。$\epsilon \eta(x)$ 就是在最优函数旁边试探性地迈出的一小步，$\eta(x)$ 代表朝哪个方向迈，$\epsilon$ 代表迈多远。

为什么要让 $\epsilon \to 0$？这和普通微积分里求导要让 $\Delta x \to 0$ 是同一个道理：我们想知道的是"这个点处"的变化率，而不是走了很远之后的平均变化率。只有无穷小试探才能反映该点的局部性质，从而判断它是不是极值点。

于是我们把 $T[y + \epsilon \eta]$ 看作关于 $\epsilon$ 的普通函数，令其在 $\epsilon = 0$ 处的导数为 0：

$$
\left. \frac{d}{d\epsilon} T[y + \epsilon \eta] \right|_{\epsilon=0} = 0
$$

### 1.3 欧拉-拉格朗日方程

对于泛函：

$$
J[y] = \int_{x_1}^{x_2} F(x, y, y') \, dx
$$

给函数加扰动：

$$
y(x) \rightarrow y(x) + \epsilon \eta(x), \quad \eta(x_1) = \eta(x_2) = 0
$$

$$

J(\epsilon) = \int_{x_1}^{x_2} F(x, \, y + \epsilon \eta, \, y' + \epsilon \eta') \, dx
$$

$$

\left. \frac{dJ}{d\epsilon} \right|_{\epsilon=0}
= \int_{x_1}^{x_2} \left( \frac{\partial F}{\partial y} \eta + \frac{\partial F}{\partial y'} \eta' \right) dx = 0
$$

对第二项分部积分：

$$
\int_{x_1}^{x_2} \frac{\partial F}{\partial y'} \eta' \, dx
= \left[ \frac{\partial F}{\partial y'} \eta \right]_{x_1}^{x_2} - \int_{x_1}^{x_2} \frac{d}{dx} \left( \frac{\partial F}{\partial y'} \right) \eta \, dx
$$

边界项消失，代入得：

$$

\int_{x_1}^{x_2} \left[ \frac{\partial F}{\partial y} - \frac{d}{dx} \left( \frac{\partial F}{\partial y'} \right) \right] \eta(x) \, dx = 0, \quad \forall \eta(x)
$$

因此：

$$

\frac{\partial F}{\partial y} - \frac{d}{dx} \left( \frac{\partial F}{\partial y'} \right) = 0
$$

这就是欧拉-拉格朗日方程。

### 1.4 最速降线的解析解

回到最速降线问题，被积函数为：

$$
F(x, y, y') = \sqrt{\frac{1 + (y')^2}{y}}
$$

由于 $F$ 不显含 $x$，可以使用贝尔特拉米恒等式：

$$
F - y' \frac{\partial F}{\partial y'} = C
$$

代入化简后得到微分方程：

$$
y(1 + (y')^2) = k
$$

解这个微分方程，得到摆线的参数方程：

$$
x = a(t - \sin t), \quad y = a(1 - \cos t)
$$

这是一个精确的解析解，它是封闭形式的数学公式。


## 2. 变分推断

### 2.1 贝叶斯推断的困境

在贝叶斯推断中，我们想求的是后验分布：

$$
p(z|x) = \frac{p(x|z)p(z)}{p(x)}
$$

其中 $z$ 是隐变量，$x$ 是观测数据。

分母 $p(x) = \int p(x|z)p(z) \, dz$ 是一个高维积分，在实际问题中无法得到解析式。所以我们根本无法写出 $p(z|x)$ 的具体表达式。

如果套用经典变分法，我们想找一个 $q(z)$ 来近似 $p(z|x)$，目标是最小化 KL 散度：

$$
J[q] = KL(q(z) \parallel p(z|x)) = \int q(z) \log \frac{q(z)}{p(z|x)} \, dz
$$

然后对 $q(z)$ 加扰动 $\epsilon \eta(z)$，令变分为 0，推导欧拉-拉格朗日方程。

但这条路走不通，因为 $p(z|x)$ 的表达式本身就含有那个算不出来的分母 $p(x)$，我们根本写不出完整的泛函形式。

### 2.2 参数化

既然精确求解不可能，深度学习变分推断做了一个简化：把"在无限函数空间里找最优函数"，变成"在有限参数空间里找最优参数"。

具体来说：

1. 限制搜索空间：我们不去所有可能的函数里找 $q(z)$，而是限定 $q(z)$ 属于某个简单的分布族，比如各分量独立的高斯分布（称为均值场变分族）：

$$
q(z) = \prod_{i=1}^d q_i(z_i), \quad q_i(z_i) = \mathcal{N}(z_i | \mu_i, \sigma_i^2)
$$

2. 参数化：寻找最优函数 $q(z)$ 变成了寻找最优参数 $\theta = \{(\mu_i, \sigma_i)\}_{i=1}^d$，记作 $q_\theta(z)$。

3. 优化目标：因为直接最小化 $KL(q \parallel p)$ 仍然困难（里面还有 $p(x)$），我们转而最大化证据下界（ELBO）：

$$
\text{ELBO}(\theta) = \mathbb{E}_{q_\theta(z)}[\log p(x|z)] - KL(q_\theta(z) \parallel p(z))
$$

这个表达式里不再有没办法写出解析式的 $p(x)$，所有项都可计算（$p(z)$ 是先验，是我们设定的；$p(x|z)$ 是似然，由解码器给出）。

4. 梯度下降：利用重参数化技巧把采样步骤变成可导的，然后用 SGD 更新 $\theta$。

所以变分推断虽然用了"变分"这个名字，但它借用的只是经典变分法"对函数求导/寻找极值"的数学思想。在工程实现上，它放弃了欧拉-拉格朗日方程，转而用参数化和梯度下降来做近似求解。