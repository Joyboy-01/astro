---
title: 为什么反向markov不能只看一步？
publishDate: 2026-06-12T20:30:00+08:00
description: 'Diffusion Model'
tags:
    - markov
language: '中文'
---

## 背景 (Background)

在经典的扩散概率模型（DDPM）中，反向去噪过程实质上是在推导反向时间步的条件分布。然而，在通用的马尔可夫链框架下，反向单步转移 $q(x_{t-1} | x_t)$ 受到复杂的原始数据边缘分布 $q(x_0)$ 的纠缠，并不满足“只参考一跳（只看相邻一步）”的简明马尔可夫形式，必须回溯到原始样本 $x_0$ 构造马尔可夫桥（Markov Bridge）$q(x_{t-1} | x_t, x_0)$ 才能写出解析闭式解：

$$
q(x_{t-1} | x_t, x_0) = \mathcal{N}(x_{t-1}; \tilde{\mu}_t(x_t, x_0), \tilde{\beta}_t\mathbf{I})
$$

在通用的随机过程中，给定起点与当前点的逆向时间条件分布即马尔可夫桥（Markov Bridge）公式 $P(X_{t-1} | X_t, X_0)$ 往往只存在于理论层面，因为任意的转移矩阵在经历多步叠加后通常无法解析。而高斯扰动核的设计，使得多步边缘分布 $P(x_t|x_0)$ 能够被直接“一步到位”地解析表达。这使得原本不可解的反向轨迹，退化成了可以被神经网络直接训练的具象高斯闭式解。

## 前向马尔可夫链

考虑离散时间马尔可夫链 $X_0 \to X_1 \to \dots \to X_t$。由马尔可夫性质，序列的联合概率分布分解为：

$$
p(X_0, X_1, \dots, X_t) = p(X_0) \prod_{i=1}^t p(X_i | X_{i-1})
$$

在 DDPM 框架下，前向单步转移核定义为线性高斯参数化形式：

$$
q(x_t | x_{t-1}) = \mathcal{N}(x_t; \sqrt{1 - \beta_t}x_{t-1}, \beta_t\mathbf{I})
$$

利用高斯变量的独立叠加性质，令 $\alpha_t = 1 - \beta_t$ 且 $\bar{\alpha}_t = \prod_{i=1}^t \alpha_i$，通过递归代入：

$$
x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{\beta_t}\epsilon_{t-1}
$$

可直接写出边缘分布 $q(x_t | x_0)$ 的“一步到位”闭式解：

$$
q(x_t | x_0) = \mathcal{N}(x_t; \sqrt{\bar{\alpha}_t}x_0, (1 - \bar{\alpha}_t)\mathbf{I})
$$

## 标准反向轨迹的不可积性 (Intractability)

基于贝叶斯定理，单步反向转移概率表述为：

$$
q(x_{t-1} | x_t) = \frac{q(x_t | x_{t-1}) q(x_{t-1})}{q(x_t)}
$$

### 为什么不能只看一步？

从概率图模型的条件独立性来看，前向过程具有局域独立性：给定 $X_{t-1}$ 时，$X_t$ 独立于所有祖先节点。

然而，当对马尔可夫链进行逆时间分析时，反向单步核 $q(x_{t-1}|x_t)$ 的概率结构包含了 $q(x_{t-1})$ 和 $q(x_t)$ 两项边缘概率分布。这导致反向轨迹在本质上与前向路径存在拓扑差异：

1. 连通性：在前向链条 $X_{t-2} \to X_{t-1} \to X_t$ 中，给定中间节点 $X_{t-1}$ 可以阻断 $X_{t-2}$ 到 $X_t$ 的依赖。但是在反向推导中，由于不知道链条的初始起点分布，任何局部的逆向一跳都依赖于这一时刻整个状态空间的概率密度分布。
2. 边缘分布的耦合：反向转移核无法仅通过前向局部的独立噪声机制（超参数 $\beta_t$）决定，它耦合了前向所有历史演化信息的累积边缘概率（即 $q(x_{t-1})$）。因此，为了明确定义这一跳，其信息流必须沿前向路径一直连通回溯至初始的数据分布 $X_0$。

计算上式需要明确写出边缘密度函数 $q(x_t)$，即必须在整个数据分布上对初始状态 $x_0$ 进行积分：

$$
q(x_t) = \int q(x_t | x_0) q(x_0) dx_0
$$

* 数据分布复杂性：真实初始数据分布 $q(x_0)$ 代表一个任意的、高维的且非解析的分布（如自然图像空间）。
* 不可积性：由于 $q(x_0)$ 的非解析特征，该积分在数学上无法解析求解。因此 $q(x_t)$ 无法写出闭式表达式。
* 轨迹依赖性：在没有给定 $x_0$ 时，反向转移 $q(x_{t-1} | x_t)$ 隐式地依赖于从初始分布开始的整条链的历史信息，破坏了局部“单步”的解析形式。

## 马尔可夫桥 (Markov Bridge) 构造

通过将反向转移概率置于初始状态 $x_0$ 的条件之下，可绕过上述不可积性问题，构建马尔可夫桥 $X_0 \to X_{t-1} \to X_t$。

再次应用贝叶斯定理：

$$
q(x_{t-1} | x_t, x_0) = \frac{q(x_t | x_{t-1}, x_0) q(x_{t-1} | x_0)}{q(x_t | x_0)}
$$

根据前向马尔可夫链的条件独立性质，当给定 $x_{t-1}$ 时，$x_t$ 与历史状态 $x_0$ 独立，即 $q(x_t | x_{t-1}, x_0) = q(x_t | x_{t-1})$。上式化简为：

$$
q(x_{t-1} | x_t, x_0) = q(x_t | x_{t-1}) \cdot \frac{q(x_{t-1} | x_0)}{q(x_t | x_0)}
$$

### 数学推导

由于右侧三项均为已知的高斯密度函数，我们可以忽略常数系数，直接提取其指数部分（Exponential Terms）进行展开与配平方。一维形式下的指数核心项可写为：

$$
\exp \left( -\frac{1}{2} \left[ \frac{(x_t - \sqrt{\alpha_t}x_{t-1})^2}{\beta_t} + \frac{(x_{t-1} - \sqrt{\bar{\alpha}_{t-1}}x_0)^2}{1 - \bar{\alpha}_{t-1}} - \frac{(x_t - \sqrt{\bar{\alpha}_t}x_0)^2}{1 - \bar{\alpha}_t} \right] \right)
$$

因为目标是推导关于 $x_{t-1}$ 的概率分布，我们将上述方括号中与 $x_{t-1}$ 相关的项展开并按 $x_{t-1}$ 的降幂排列（将不含 $x_{t-1}$ 的项归入常数 $C$）：

$$
= \frac{x_t^2 - 2\sqrt{\alpha_t}x_t x_{t-1} + \alpha_t x_{t-1}^2}{\beta_t} + \frac{x_{t-1}^2 - 2\sqrt{\bar{\alpha}_{t-1}}x_0 x_{t-1} + \bar{\alpha}_{t-1}x_0^2}{1 - \bar{\alpha}_{t-1}} + C
$$

$$
= \left( \frac{\alpha_t}{\beta_t} + \frac{1}{1 - \bar{\alpha}_{t-1}} \right) x_{t-1}^2 - 2 \left( \frac{\sqrt{\alpha_t}}{\beta_t}x_t + \frac{\sqrt{\bar{\alpha}_{t-1}}}{1 - \bar{\alpha}_{t-1}}x_0 \right) x_{t-1} + C
$$

对比标准高斯分布指数项 $\frac{(x_{t-1} - \tilde{\mu}_t)^2}{\tilde{\beta}_t} = \frac{1}{\tilde{\beta}_t} x_{t-1}^2 - \frac{2\tilde{\mu}_t}{\tilde{\beta}_t} x_{t-1} + C$，通过待定系数法（Coefficient Matching）：

#### 求解条件方差

$$
\frac{1}{\tilde{\beta}_t} = \frac{\alpha_t}{\beta_t} + \frac{1}{1 - \bar{\alpha}_{t-1}} = \frac{\alpha_t(1 - \bar{\alpha}_{t-1}) + \beta_t}{\beta_t(1 - \bar{\alpha}_{t-1})} = \frac{\alpha_t - \bar{\alpha}_t + 1 - \alpha_t}{\beta_t(1 - \bar{\alpha}_{t-1})} = \frac{1 - \bar{\alpha}_t}{\beta_t(1 - \bar{\alpha}_{t-1})}
$$

$$
\tilde{\beta}_t = \frac{1 - \bar{\alpha}_{t-1}}{1 - \bar{\alpha}_t} \beta_t
$$

#### 求解条件均值

$$
\frac{\tilde{\mu}_t}{\tilde{\beta}_t} = \frac{\sqrt{\alpha_t}}{\beta_t}x_t + \frac{\sqrt{\bar{\alpha}_{t-1}}}{1 - \bar{\alpha}_{t-1}}x_0
$$

将 $\tilde{\beta}_t$ 代入上式：

$$
\tilde{\mu}_t = \tilde{\beta}_t \left( \frac{\sqrt{\alpha_t}}{\beta_t}x_t + \frac{\sqrt{\bar{\alpha}_{t-1}}}{1 - \bar{\alpha}_{t-1}}x_0 \right) = \frac{1 - \bar{\alpha}_{t-1}}{1 - \bar{\alpha}_t}\beta_t \cdot \frac{\sqrt{\alpha_t}}{\beta_t}x_t + \frac{1 - \bar{\alpha}_{t-1}}{1 - \bar{\alpha}_t}\beta_t \cdot \frac{\sqrt{\bar{\alpha}_{t-1}}}{1 - \bar{\alpha}_{t-1}}x_0
$$

$$
\tilde{\mu}_t(x_t, x_0) = \frac{\sqrt{\alpha_t}(1 - \bar{\alpha}_{t-1})}{1 - \bar{\alpha}_t}x_t + \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1 - \bar{\alpha}_t}x_0
$$

由此，反向桥梁条件分布证明为闭式高斯分布：

$$
q(x_{t-1} | x_t, x_0) = \mathcal{N}(x_{t-1}; \tilde{\mu}_t(x_t, x_0), \tilde{\beta}_t\mathbf{I})
$$

## 通用马尔可夫链

此约束并非扩散模型所独有，而是通用马尔可夫过程的底层属性。对于任意形式的马尔可夫链，通过马尔可夫桥参数化的反向单步转移方程普遍成立：

$$
P(X_{t-1} | X_t, X_0) = P(X_t | X_{t-1}) \cdot \frac{P(X_{t-1} | X_0)}{P(X_t | X_0)}
$$

### 对比

| 属性 / 维度 | 通用马尔可夫链 | 扩散模型 (DDPM 前向) |
| --- | --- | --- |
| 前向单步核 $P(X_t \mid X_{t-1})$ | 任意转移矩阵 / 连续状态转移核 | 线性高斯扰动（解析极简） |
| 多步边缘分布 $P(X_t \mid X_0)$ | 需计算 $t$ 步转移矩阵（通常无法解析） | 依靠高斯叠加原理，可通过参数 $\bar{\alpha}_t$ 直接写出闭式 |
| 反向条件桥 $P(X_{t-1} \mid X_t, X_0)$ | 结构上实现解耦，但因组件缺乏解析表达，仍无法写出闭式 | 三个高斯分布乘除，通过指数项系数匹配完美得到高斯闭式解 |

## 结论

在通用马尔可夫链中，若要孤立反向单步转移，必须锚定边界点 $X_0$ 以解耦复杂的任意边缘分布 $P(X_t)$ 的影响。扩散模型精确地利用了这一结构性分解，并通过设计特定的高斯前向核，确保了马尔可夫桥中的每一项都具备精确的代数闭式解，从而满足了神经网络进行函数逼近的工程可行性。