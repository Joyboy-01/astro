---
title: Why Diffusion Models Don't Memorize--background
publishDate: 2026-04-15T20:30:00+08:00
description: 'background of Diffusion Model'
tags:
  - paper阅读
language: '中文'
---

Why Diffusion Models Don't Memorize: The Role of Implicit Dynamical Regularization in Training (NeurIPS25)

这篇文章揭示并证明了diffusion model的学习过程是先泛化，后记忆，且给出了数学的证明以及量化。

图片生成模型有很多，按照发展来看我们先有了GAN生成网络，随后是diffusion的时代（2020--），Stable diffusion对于diffusion model的改进以及开源让diffusion成为了现在仍在研究演化的重要方向。Diffusion model的发展从DDPM开始，随后科学家们发现DDPM的离散去噪过程可以扩展到连续时间，这背后正得分匹配（Score Matching） 理论， 得分理论仍然是截止目前为止的最重要的发展理论基石以及研究热点。

[diffusion model是什么?](https://blog.csdn.net/m0_59012280/article/details/155040575)可以很好的作为diffusion model的入门，他对Diffusion model的基本理论和进行了解释以及主要讲解了DDPM的数学证明。这里最重要的证明是：“通过预测噪声的方式，就能让模型学得训练数据的分布，进而产生逼真的图片”，博客中给出了详细的证明过程。然而我们可以在证明过程中发现，“单步的 ELBO 扩展为覆盖 T 个时间步的链式 ELBO” 过程中，每一步的计算是离散的，且结论的得出过程不是很自然。**Score-Based Generative Modeling through Stochastic Differential Equations**这篇论文开启了扩散模型的新篇章，通过把 DDPM 的离散加噪过程，写成一个连续时间的 SDE（前向过程），然后使用微分方程描述这个过程，这提供了更坚实的数学理论基础且采样的确定性以及带来了速度提升。[博客1](https://blog.csdn.net/m0_62249876/article/details/134358417)对于此过程进行了理论推导。

DDPM 的单步加噪是：

$$
x_t = \sqrt{1 - \beta_t}\, x_{t-1} + \sqrt{\beta_t}\, \epsilon_t, \quad \epsilon_t \sim \mathcal{N}(0, I)
$$

现在把时间步之间的间隔记作 $\Delta t$，也就是说，从t-1到 t 实际是从时刻 $\tau$到时刻 $\tau + \Delta$ t。

把 $\beta_t = \beta(\tau) \Delta t$（把离散的 $beta$ 改写成连续函数乘以小步长），代入 DDPM 公式：

$$
x_{\tau + \Delta t} = \sqrt{1 - \beta(\tau)\Delta t}\, x_\tau + \sqrt{\beta(\tau)\Delta t}\, \epsilon
$$

用 $\sqrt{1 - y} \approx 1 - y/2$（y 很小）：

$$
x_{\tau + \Delta t} \approx x_\tau - \frac{\beta(\tau)}{2} x_\tau \Delta t + \sqrt{\beta(\tau)\Delta t}\, \epsilon
$$

$$
x_{\tau + \Delta t} - x_\tau \approx -\frac{\beta(\tau)}{2} x_\tau \Delta t + \sqrt{\beta(\tau) \Delta t}\, \epsilon
$$

当 $\Delta t \to 0$，左边趋于微分 dx。右边第二项$\sqrt{\beta(\tau) \Delta t}\, \epsilon$ 在随机过程的语言里就是$\sqrt{\beta(\tau)}\, dB(\tau)$  （dB 是布朗运动的微分，"连续时间版的高斯噪声"）。于是离散 DDPM 的连续极限是：

$$
dx = -\frac{\beta(\tau)}{2} x\, d\tau + \sqrt{\beta(\tau)}\, dB(\tau)
$$
这就是一个 [SDE（随机微分方程)](https://blog.csdn.net/xzs1210652636/article/details/145373585)。学习[SDE](https://zhuanlan.zhihu.com/p/405174311)还要对ODE有一定的了解。（但是我完全不懂，AI解释说SDE 描述了一个粒子如何随时间运动：它被一个"中心力"拉向原点，同时被随机噪声推来推去
跑足够长时间后，粒子分布会**稳定**在某个分布上。）
Anderson 1982的定理解释了随机过程反向运行的公式：如果前向 SDE 是

$$
dx = f(x, \tau)\, d\tau + g(\tau)\, dB(\tau)
$$

那么反向 SDE 是

$$
dx = \left[f(x, \tau) - g(\tau)^2\, \nabla_x \log P_\tau(x)\right] d\tau + g(\tau)\, d\bar{B}(\tau)
$$

其中 $\bar{B}$是反向时间的布朗运动，$P_\tau(x)$ 是前向过程在时刻 $\tau$ 的边际分布。可以发现 $\nabla_x \log P_\tau(x)$ 就是得分，我们只要指导得分，根据前向，我们就可以得知反向怎么计算。博客中给出了证明，直观上我们可以这样来看得分的意义：我们要求一个反向过程，根据贝叶斯公式：$P(x_\tau | x_{\tau + d\tau}) \propto P(x_{\tau + d\tau} | x_\tau) \cdot P(x_\tau)$右边第一项是前向转移（简单的高斯）。右边第二项是粒子在时刻 $\tau$  的边际分布。所以反向过程中的先验部分就是得分的部分，换句话说，得分代表了数据分布本身的信息，得分的梯度就代表了恢复时刻$\tau$速度最快的方向。

回到前向 SDE（OU 过程）。给定初始数据 $x_0$，时刻 $\tau$ 的条件分布是精确可算的（因为 OU 过程是线性的、高斯驱动的）：

$$
x_\tau | x_0 \sim \mathcal{N}(e^{-\tau} x_0, \Delta_\tau I), \quad \Delta_\tau = 1 - e^{-2\tau}
$$

所以含噪样本可以写成：

$$
x_\tau = e^{-\tau} x_0 + \sqrt{\Delta_\tau}\, \epsilon, \quad \epsilon \sim \mathcal{N}(0, I)
$$

由于我们都假定为高斯分布，所以

$$
\nabla_{x_\tau} \log P(x_\tau | x_0) = -\frac{x_\tau - e^{-\tau} x_0}{\Delta_\tau} = -\frac{\epsilon}{\sqrt{\Delta_\tau}}
$$

Hyvärinen (2005)和Vincent (2011) 证明了一个重要等价。最小化

$$
\mathbb{E}_{x_0, \epsilon}\|s_\theta(x_\tau, \tau) - \nabla_{x_\tau} \log P(x_\tau | x_0)\|^2
$$

和最小化

$$
\mathbb{E}_{x_\tau}\|s_\theta(x_\tau, \tau) - \nabla_{x_\tau} \log P_\tau(x_\tau)\|^2
$$

有相同的最优解。所以综合来说我们拟合得分，就相当于拟合$-\frac{\epsilon}{\sqrt{\Delta_\tau}}$，而下面的是常数，所以我们的S通过拟合噪声，变向的拟合了得分，之后就可以进行反向过程。

理论上我们可以完美的拟合到得分，进行反向过程，完美的还原出图片。但是问题在于我们永远不能拟合真实分布，我们拟合的是经验分布：$s_{\text{emp}}(x, t) = \nabla_x \log P_t^{\text{emp}}(x)$
$P_t^{\text{emp}}$是 **经验分布** $P_0^{\text{emp}} = \frac{1}{n}\sum_\nu \delta(x - x^\nu)$前向扩散 t时间后的分布，它的分布是n个训练样本构成的离散分布，可以证明如果反向 SDE 用的得分是 $s_{\text{emp}}$，那生成出来的分布精确等于$P_0^{\text{emp}}$，因为当经验分布固定所以得分固定所以反向过程的数学公式固定所以反向传播到0时刻的分布固定，而这个分布是GMM：反向 SDE 的随机性只是"采样过程"的随机性，不是"结果分布"的随机性。(为什么语言模型有double descent呢？是因为损失的全局最优是真实条件概率不是一个固定分布么？)

然而现实情况是：我们的图片生成模型并不是训练后就只生成训练集中的内容，这说明模型的确泛化了！直觉上的回答可能是在我们使用的图片生成模型中的训练样本数量很大，也许接近了真实分布，所以有良好的泛化性，然而Biroli et al. 2024证明了一个事情，接近真实分布（可以让模型繁华）的数据量是不可能达到的：具体地：n 必须满足 $n > e^{cd}$ （c 是某个常数）才能让 n 个高斯包真的融合成连续分布。

对 CelebA 32×32 灰度图，d=1024。$e^{1024}$是一个不可想象的数。而实际数据只有 20 万张。按这个理论：我们永远达不到 n 足够大的程度。所以模型学到的内容一定不是最终的经验分布得分，而存在着其他分布，在这个想法提出后，有一些论文发表来探讨这一现象，比如：

- Kadkhodaie, Guth, Simoncelli, Mallat (2024), "Generalization in diffusion models arises from geometry-adaptive harmonic representations"（ICLR 2024）
- Kamb & Ganguli (2024), "An analytic theory of creativity in convolutional diffusion models"
- Wu, Marion, Biau, Boyer (2025), "Taking a big step: Large learning rates in denoising score matching prevent memorization"。
- chilli, Ventura, Silvestri, Pham, Raya, Krotov, Lucibello, Ambrogioni (2024), "Losing dimensions: Geometric memorization in generative diffusion"
- Ventura, Achilli, Silvestri, Lucibello, Ambrogioni (2025), "Manifolds, random matrices and spectral gaps: The geometric phases of generative diffusion"

然而和本文相比，他们的研究范围过于局限，他们不能解释在一些范围下的情况，而这篇文章统一了这些文章的说法，并给出了一个更有一般性的结论。

### Hyvärinen & Vincent

Hyvärinen 利用分部积分法在假设数据边界概率为 0 的条件下，证明了：

$$
\mathbb{E}_{P_t} [\|s_\theta(x) - \nabla_x \log P_t(x)\|^2] \equiv \mathbb{E}_{P_t} [\|s_\theta(x)\|^2 + 2\,\text{tr}(\nabla_x s_\theta(x))] + \text{const}
$$

这说明可以完全不需要知道真实得分，只通过优化模型自身的输出（$s_\theta$ 的模长）和模型输出相对于输入的导数（Jacobian 的迹），就能强迫模型学会真实分布的形状。高维下，Hyvärinen 的方法在工程上不可行。

Vincent 证明了最小化 $\mathbb{E}_{x_0, \epsilon}\|s_\theta(x_\tau, \tau) - \nabla_{x_\tau} \log P(x_\tau | x_0)\|^2$和最小化 $\mathbb{E}_{x_\tau}\|s_\theta(x_\tau, \tau) - \nabla_{x_\tau} \log P_\tau(x_\tau)\|^2$有相同的最优解：


$$
\nabla_{x_t} \log P_t(x_t) = \frac{\nabla_{x_t} P_t(x_t)}{P_t(x_t)}
$$

因为 $P_t(x_t) = \int P(x_t | x_0) P(x_0) dx_0$（可微分性和可积性都满足），所以对它求导可以把导数符号移进积分号里：

$$
\nabla_{x_t} P_t(x_t) = \int [\nabla_{x_t} P(x_t | x_0)] P(x_0) dx_0
$$

利用 $\nabla P = P \nabla \log P$，改写积分项：

$$
\nabla_{x_t} P_t(x_t) = \int [P(x_t | x_0) \nabla_{x_t} \log P(x_t | x_0)] P(x_0) dx_0
$$

$$
\nabla_{x_t} \log P_t(x_t) = \frac{\int \nabla_{x_t} \log P(x_t | x_0) P(x_t | x_0) P(x_0) dx_0}{P_t(x_t)}
$$

注意到 $\frac{P(x_t | x_0) P(x_0)}{P_t(x_t)} = P(x_0 | x_t)$（这是给定含噪点后，原始点 $x_0$ 的后验概率）。于是：

$$
\nabla_{x_t} \log P_t(x_t) = \int \nabla_{x_t} \log P(x_t | x_0) P(x_0 | x_t) dx_0 = \mathbb{E}_{x_0 \sim P(x_0|x_t)} [\nabla_{x_t} \log P(x_t | x_0)]
$$

设两个损失函数分别为：

$$
L_1(\theta) = \mathbb{E}_{x_0, \epsilon}|s_\theta(x_t, t) - \nabla_{x_t} \log P(x_t | x_0)|^2
$$

$$
L_2(\theta) = \mathbb{E}_{x_t}|s_\theta(x_t, t) - \nabla_{x_t} \log P_t(x_t)|^2
$$

需要证明 $\arg\min_\theta L_1 = \arg\min_\theta L_2$。
展开 $L_2$：

$$
L_2 = \mathbb{E}_{x_t}|s_\theta - \nabla_{x_t} \log P_t(x_t)|^2
$$

把期望写成对 $(x_0, x_t)$ 联合分布的期望（因为对 $x_t$ 的边际期望可以通过联合分布得到）：

$$
= \mathbb{E}_{x_0, x_t}|s_\theta - \nabla_{x_t} \log P_t(x_t)|^2
$$

$$
L_1 - L_2 = \mathbb{E}_{x_0, x_t}\left[|s_\theta - \nabla \log P(x_t|x_0)|^2 - |s_\theta - \nabla \log P_t(x_t)|^2\right]
$$

$$
= \mathbb{E}_{x_0, x_t}\left[|\nabla \log P_t|^2 - |\nabla \log P(x_t|x_0)|^2 + 2(s_\theta - \nabla \log P_t)^\top(\nabla \log P_t - \nabla \log P(x_t|x_0))\right]
$$

其中**关键**是交叉项中：

$$
\mathbb{E}_{x_0, x_t}\left[(s_\theta - \nabla \log P_t)^\top(\nabla \log P_t - \nabla \log P(x_t|x_0))\right]
$$

利用刚才证明的恒等式 $\nabla \log P_t(x_t) = \mathbb{E}_{x_0|x_t}[\nabla \log P(x_t|x_0)]$，对固定 $x_t$ 先对 $x_0|x_t$ 求期望：

$$
\mathbb{E}_{x_0|x_t}[\nabla \log P_t - \nabla \log P(x_t|x_0)] = \nabla \log P_t - \nabla \log P_t = 0
$$

所以交叉项为零，于是：

$$
L_1 = L_2 + \text{与} \theta \text{ 无关的常数}
$$

因此两者关于 $\theta$ 的梯度相同，最优解一致。
