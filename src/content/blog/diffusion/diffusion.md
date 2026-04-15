---
title: Why Diffusion Models Don't Memorize
publishDate: 2026-04-15T20:30:00+08:00
description: 'Diffusion Model'
tags:
  - paper阅读
language: '中文'
---
## Background
Why Diffusion Models Don't Memorize: The Role of Implicit Dynamical Regularization in Training (NeurIPS25)

这篇文章揭示并证明了diffusion model的学习过程是先泛化，后记忆，且给出了数学的证明以及量化。

图片生成模型有很多，按照发展来看我们先有了GAN生成网络，随后是diffusion的时代（2020--），Stable diffusion对于diffusion model的改进以及开源让diffusion成为了现在仍在研究演化的重要方向。Diffusion model的发展从DDPM开始，随后科学家们发现DDPM的离散去噪过程可以扩展到连续时间，这背后正得分匹配（Score Matching） 理论， 得分理论仍然是截止目前为止的最重要的发展理论基石以及研究热点。

[diffusion model是什么?](https://blog.csdn.net/m0_59012280/article/details/155040575)可以很好的作为diffusion model的入门，他对Diffusion model的基本理论和进行了解释以及主要讲解了DDPM的数学证明。这里最重要的证明是：“通过预测噪声的方式，就能让模型学得训练数据的分布，进而产生逼真的图片”，博客中给出了详细的证明过程。然而我们可以在证明过程中发现，“单步的 ELBO 扩展为覆盖 T 个时间步的链式 ELBO” 过程中，每一步的计算是离散的，且结论的得出过程不是很自然。Score-Based Generative Modeling through Stochastic Differential Equations这篇论文开启了扩散模型的新篇章，通过把 DDPM 的离散加噪过程，写成一个连续时间的 SDE（前向过程），然后使用微分方程描述这个过程，这提供了更坚实的数学理论基础且采样的确定性以及带来了速度提升。[博客1](https://blog.csdn.net/m0_62249876/article/details/134358417)对于此过程进行了理论推导。

DDPM 的单步加噪是：

$$
x_t = \sqrt{1 - \beta_t}\, x_{t-1} + \sqrt{\beta_t}\, \epsilon_t, \quad \epsilon_t \sim \mathcal{N}(0, I)
$$

现在把时间步之间的间隔记作 $\Delta t$，也就是说，从t-1到 t 实际是从时刻 $\tau$到时刻 $\tau + \Delta t$ 。

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
跑足够长时间后，粒子分布会稳定在某个分布上。）
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
$P_t^{\text{emp}}$是 经验分布 $P_0^{\text{emp}} = \frac{1}{n}\sum_\nu \delta(x - x^\nu)$前向扩散 t时间后的分布，它的分布是n个训练样本构成的离散分布，可以证明如果反向 SDE 用的得分是 $s_{\text{emp}}$，那生成出来的分布精确等于$P_0^{\text{emp}}$，因为当经验分布固定所以得分固定所以反向过程的数学公式固定所以反向传播到0时刻的分布固定，而这个分布是GMM：反向 SDE 的随机性只是"采样过程"的随机性，不是"结果分布"的随机性。(为什么语言模型有double descent呢？是因为损失的全局最优是真实条件概率不是一个固定分布么？)

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

其中关键是交叉项中：

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


## Experiemnt
论文用一系列实验从不同角度消除了可能存在的疑问。
### 主实验

主实验在 CelebA 数据集上训练 U-Net，改变训练集大小，观察 FID 和 $f_{\text{mem}}$ 随训练时间 $\tau$ 的演化（论文 Figure 2 左面板）。

- 所有不同 $n$ 的 FID 曲线几乎重合。这直接说明 $\tau_{\text{gen}}$ 不依赖 $n$。
- $f_{\text{mem}}$ 曲线开始上升的时间随 $n$ 推后。

关键证据是"塌缩"：把横轴换成 $\tau / n$，所有 $f_{\text{mem}}$ 曲线塌缩成一条曲线。为什么塌缩这么有说服力？因为如果 $\tau_{\text{mem}}$ 只是笼统地"随 $n$ 增大"，曲线在任何变换下都不会完美重合。但 $\tau_{\text{mem}} = c \cdot n$这个精确的线性关系，正好对应"横轴除以 $n$ 后曲线重合"。塌缩是标度律的几何化验证——它比直接画 $\tau_{\text{mem}}$ 对 $n$ 的散点图更有说服力，因为它验证了标度律在整个曲线形状上都成立。

视觉上的直观证据来自 Figure 2 右面板：同一个 $n $ 训练的模型，在 $\tau = 100K$ 步生成的人脸和训练集的最近邻明显不同；但训到 $\tau = 1.62M$ 步，生成的人脸和最近邻几乎一模一样。

### 控制实验

观察到 $\tau_{\text{mem}} \propto n$ 后，最自然的解释会浮现出来：

> "$n$ 大时每个样本被 SGD 采样的频率更低（每个 batch 里它出现的概率更小），所以要让模型记住每个样本，需要更多步数——于是 $\tau_{\text{mem}} \propto n$。"

如果这个解释是对的，论文的发现就只是优化过程的表面效应，谈不上深刻。所以论文必须排除这个解释。

排除的方法是一个漂亮的控制实验：用全批次梯度下降（$B = n$，每步用所有训练样本）。这意味着每个样本在每一步都被"看到"——采样频率不再依赖 $n$。如果 $\tau_{\text{mem}} \propto n$ 来自采样频率效应，全批次下应该消失。但论文 Figure 7 左图显示：全批次下 $\tau_{\text{mem}} \propto n$ 仍然成立。

这个控制实验把论文的核心发现从"优化过程的副产物"提升到"损失景观本身的性质"。这是论文方法论的一个亮点——主动反驳最自然的替代假说，让结论更有力。

### 鲁棒性验证

证明了核心标度律之后，下一个问题是：这个现象有多普适？论文从多个维度做扩展。

模型容量维度（论文 Figure 3）：改变 U-Net 的基础宽度，对应参数量 $p$ 的改变。结果是另一组漂亮的标度律：

$$
\tau_{\text{gen}} \propto \frac{1}{W}, \quad \tau_{\text{mem}} \propto \frac{n}{W}
$$

两个时间尺度对 $W$ 的依赖相同——所以两者的比值 $\tau_{\text{mem}} / \tau_{\text{gen}} \propto n$，只依赖于 $n$，不依赖于模型大小。这意味着泛化窗口的相对长度对所有模型尺寸都是一样的。

#### 扩散时间的影响

附录 D（Figure 11）测试了不同扩散时间 下的标度律。结果：所有 t 下重新缩放时间后曲线都塌缩，标度律对扩散时间稳健。同时还揭示了一个新现象，这个新现象论文没有理论解释，留作开放问题。

#### 数据方差的影响

附录 D（Figure 12）测试了不同数据方差 。Theorem 3.1 实际上对一般数据协方差成立。Figure 12 验证了这一点。

#### 整合

把模型容量和数据量两个变量整合，论文画出了一个 $(n, p)$ 相图（Figure 3 右）：横轴是参数量 $p$，纵轴是数据量 $n$，多条曲线对应不同训练时间下"刚好不记忆"的边界。

这张图把整个扩散模型理论的版图整合到一起：

- 记忆区：$n$ 太小，记忆不可避免
- 架构正则化区：$n > n^*(p)$，模型表达不出 $s_{\text{emp}}$，永远不会记忆
- 动力学正则化区：模型有能力记忆，但训练时间不够——这正是本文新发现的区域

最重要的观察是：随着训练时间 $\tau$ 增大，"刚好不记忆"的边界向上移动，最终收敛到架构正则化的边界 $n^*(p)$。这意味着新理论包含旧理论作为极限情况——前人的架构正则化是 $\tau \to \infty$ 极限下的行为。本文的动力学正则化在有限 $\tau$ 下接管。两套理论在极限下严格一致。这扩展了旧理论，而不是推翻它。

### 普适性验证

到目前为止所有实验都用 CelebA + U-Net + SGD。但读者会问：这个标度律是这一特定配置的产物，还是扩散模型的普遍性质？论文用一系列跨系统实验回答这个问题。
优化器维度（论文附录 A.3，Figure 7 右）：用 Adam 替代 SGD。结果同样的两个时间尺度结构存在，$\tau_{\text{mem}} \propto n$ 同样成立。唯一差别是 Adam 总训练步数少一个数量级。这说明现象不是 SGD 特有的——是损失景观的内在性质。

#### 数据分布维度

换一个完全不同的数据分布——8 维高斯混合（GMM），用残差 MLP 而不是 U-Net 训练。这里数据不再是图像，没有空间结构；网络也不再是 CNN。但 $\tau_{\text{mem}} \propto n$ 仍然成立（Figure 9）。这说明现象不依赖于"图像"这个特殊数据类型，也不依赖于 CNN 这个特殊架构。

#### 生成任务维度

换成条件生成（classifier-free guidance）。$\tau_{\text{mem}} \propto n$ 在条件设定下也成立（Figure 10）。

每一个验证都让结论从"特定配置下成立"变成"更普遍成立"。当四个维度都验证后，结论被强化为"这是扩散模型本身的性质，不是任何特定选择的副产物"。

## Theory

理论证明占了大部分文章的篇幅，然而证明性的内容很多，但其实思路比较简单。证明用到了一系列分析工具比如复本方法等统计物理学中的方法。

理论证明的目标是在一个可解析分析的简化模型（RFNN + 高斯数据）中，严格推导出两个时间尺度的存在，并证明：$\tau_{\text{gen}} \sim O(1)$,$\tau_{\text{mem}} \sim \psi_n = n/d$：

1. 写下 RFNN 的训练动力学方程
2. 证明动力学是线性的，有闭式解
3. 闭式解显示时间尺度 = 某个矩阵 $U$ 的特征值倒数
4. 用随机矩阵理论计算 $U$ 的特征值谱
5. 证明这个谱在高维极限下分裂成两个 bulk
6. 大 bulk 对应 $\tau_{\text{gen}}$，小 bulk 对应 $\tau_{\text{mem}}$​
7. 小 bulk 的位置正比于 $\psi_p/\psi_n = p/n$，给出 $\tau_{\text{mem}} \propto n$
整个证明的核心是谱分裂这一现象。其他步骤都是为了把动力学的时间尺度问题归结为谱分裂问题。

### 证明

RFNN模型的表示如下

$$s_A(x) = \frac{A}{\sqrt{p}} \sigma\left(\frac{Wx}{\sqrt{d}}\right)$$

- $W \in \mathbb{R}^{p \times d}$：第一层权重，随机初始化后冻结
- $A \in \mathbb{R}^{d \times p}$：第二层权重，唯一被训练的参数
- $\sigma$：非线性激活函数（论文用 tanh）
论文只考虑固定噪声时刻 $t$ 的损失，而不是对所有 $t$ 积分。真实扩散模型的损失是对 $t \in [0, T]$ 积分的。理论分析中对每个 $t$ 单独分析，结论会更清晰。而且实验显示在一个代表性的小 $t$（比如 $t = 0.01$）下分析已经抓住了主要现象。

固定 $t$ 时，训练损失（论文公式 4）是：

$$
\mathcal{L}_{\text{train}}(A) = \frac{1}{n} \sum_{\nu=1}^{n} \mathbb{E}_\xi \left| \sqrt{\Delta_t}, s_A(x^\nu_t) + \xi \right|^2
$$

其中 $x^\nu_t = e^{-t} x^\nu + \sqrt{\Delta_t}, \xi$ 是加了噪声的训练样本。

把 $s_A$ 的定义代入并展开平方：

$$
\mathcal{L}_{\text{train}}(A) = 1 + \frac{\Delta_t}{d}\text{Tr}\left( \frac{A^T A}{p} U \right) + \frac{2\sqrt{\Delta_t}}{d}\text{Tr}\left( \frac{A}{\sqrt{p}} V \right)
$$

损失是 $A$ 的二次函数。这里出现了两个矩阵：

$$
U = \frac{1}{n} \sum_\nu \mathbb{E}_\xi\left[ \sigma(Wx^\nu_t/\sqrt{d}), \sigma(Wx^\nu_t/\sqrt{d})^T \right]
$$

$$
V = \frac{1}{n} \sum_\nu \mathbb{E}_\xi\left[ \sigma(Wx^\nu_t/\sqrt{d}), \xi^T \right]
$$

$$
\nabla_A L_{train} = \frac{2\Delta_t}{dp} A U + \frac{2\sqrt{\Delta_t}}{d\sqrt{p}} V^T
$$

$U$ 是训练样本经过随机特征映射后得到的 $p$-维特征向量的协方差矩阵（在数据和噪声的平均意义下）。训练动力学的二次项 $\text{Tr}(A^T A U / p)$ 完全由 $U$ 决定。梯度下降在 $A$ 方向上的动力学速度 = $U$ 在对应特征向量方向上的特征值。

因为损失是 $A$ 的二次函数，梯度 $\nabla_A \mathcal{L}_{\text{train}}$ 是 $A$ 的线性函数。

$$
A^{(k+1)} - A^{(k)} = -\eta \nabla_A L_{train}
$$

$$
\frac{dA}{d\tau} = \frac{A^{(k+1)} - A^{(k)}}{\eta / d^2} = - d^2 \nabla_A L_{train}
$$

$$
\dot{A}(\tau) = -2\Delta_t \frac{d}{p} A U - \frac{2d\sqrt{\Delta_t}}{\sqrt{p}} V^T
$$

这是关于 $A(\tau)$ 的线性常微分方程，可以精确求解。初始条件 $A(0) = 0$ 下的解：

$$
\frac{A(\tau)}{\sqrt{p}} = -\frac{1}{\sqrt{\Delta_t}}V^T U^{-1} \left( 1 - e^{-\frac{2\Delta_t}{\psi_p} U \tau} \right)
$$

矩阵指数 $e^{-\frac{2\Delta_t}{\psi_p} U \tau}$ 可以在 $U$ 的特征基上对角化。设 $U$ 的特征值分解为 $U = \sum_\lambda \lambda, v_\lambda v_\lambda^T$。那么：

$$
e^{cU} = \sum_\lambda \left[ 1 + c\lambda + \frac{(c\lambda)^2}{2!} + \frac{(c\lambda)^3}{3!} + \dots \right] v_\lambda v_\lambda^T
$$

$$
e^{-\frac{2\Delta_t}{\psi_p} U \tau} = \sum_\lambda e^{-\frac{2\Delta_t \lambda}{\psi_p} \tau}v_\lambda v_\lambda^T
$$

每个特征向量 $v_\lambda$ 方向上的动力学是独立的指数衰减，衰减率为 $\lambda / \psi_p$，时间尺度为：

$$
\tau_\lambda \sim \frac{\psi_p}{\Delta_t \lambda}
$$

结论：时间尺度 = 特征值的倒数（乘上常数 $\psi_p / \Delta_t$）。
所以大特征值 $\lambda$ → 小 $\tau_\lambda$ → 快速学习的方向，小特征值 $\lambda$ → 大 $\tau_\lambda$ → 慢速学习的方向。如果 $U$ 的谱分成两个分离的 bulk一个大值，一个小值），那训练动力学就有两个分离的时间尺度。这正是论文要证明的谱分裂。

$$
U = \frac{1}{n} \sum_\nu \mathbb{E}_\xi\left[ \sigma(W x^\nu_t / \sqrt{d}), \sigma(W x^\nu_t / \sqrt{d})^T \right]
$$

$U$ 里有非线性函数 $\sigma$（比如 tanh）。这让随机矩阵理论无法直接处理——它的标准工具都是对线性随机矩阵设计的。

高斯等价原理（Gerace et al. 2020，Hu & Lu 2023）说：在高维极限下 $(d, n, p \to \infty)$，非线性随机特征模型 $U$ 的谱等价于某个线性高斯模型的谱。

具体地（论文 Lemma C.1），$U$ 的谱等于如下矩阵的谱：

$$
U \equiv \frac{G G^T}{n} + b_t^2\frac{W W^T}{d} + s_t^2I_p
$$

其中：

- $G = e^{-t} a_t\frac{W}{\sqrt{d}} X' + v_t, \Omega$ 是一个高斯矩阵
- $X'$ 是协方差为 $\Sigma$ 的高斯数据矩阵
- $\Omega$ 是独立的高斯矩阵
- $a_t, b_t, v_t, s_t$ 是依赖于 $t$ 和 $\sigma$ 的常数（论文公式 12-14）

也就是说：把 $\sigma(Wx/\sqrt{d})$ 等价替换为 $a_t, Wx/\sqrt{d} + v_t \cdot \text{noise}$——非线性被替换为线性加独立噪声。

$\sigma(Wx/\sqrt{d})$ 在高维下输入是接近高斯的（因为 $Wx$ 是很多独立项的和，中心极限定理）。所以我们可以把 $\sigma$ 的输出做Hermite 多项式展开：

$$
\sigma(z) = \sum_k \frac{\alpha_k}{k!} He_k(z)
$$

其中 $He_k$ 是 Hermite 多项式。在高维下，只有前两个系数（线性项和常数项）贡献到谱的主要部分。高阶项的贡献是 $O(1/d)$ 的小量，可以忽略。

线性项的系数 $a_t = \alpha_1$ 给出了"等价线性模型"的斜率，常数项加上高阶剩余给出了"等价噪声"的方差 $v_t^2, s_t^2$。

这就是为什么非线性可以被等价为"线性 + 高斯噪声"。
$U$ 简化后的结构：

$$
U = \underbrace{\frac{G G^T}{n}}_{\text{依赖数据 } X'} + \underbrace{b_t^2 \frac{W W^T}{d}}_{\text{只依赖随机权重 } W} + \underbrace{s_t^2 I_p}_{\text{常数}}
$$

Theorem 3.1 给出了计算 $U$ 的谱密度 $\rho(\lambda)$ 的工具——三个耦合方程。它们定义了 $U$ 的 Stieltjes 变换，而 Stieltjes 变换可以反推出谱密度。

在过参数化极限 $\psi_p > \psi_n \gg 1$ 下，$U$ 的谱密度是：

$$\rho(\lambda) = \left(1 - \frac{1 + \psi_n}{\psi_p}\right) \delta(\lambda - s_t^2) + \frac{\psi_n}{\psi_p}\rho_1(\lambda) + \frac{1}{\psi_p}\rho_2(\lambda)$$

### 三个分量

1. $\delta(\lambda - s_t^2)$：位于 $s_t^2$ 的 delta 峰，权重约 $1 - (1+\psi_n)/\psi_p \sim 1$
  
2. $\rho_1$：一个分布在以下区间的连续谱：

$$
\left[ s_t^2 + v_t^2(1 - \sqrt{\psi_p/\psi_n})^2s_t^2 + v_t^2(1 + \sqrt{\psi_p/\psi_n})^2 \right]
$$

Marchenko-Pastur 定律（经典随机矩阵结果）：$p \times n$ 高斯矩阵的 $G G^T / n$ 的特征值分布在区间 $[(\sqrt{p/n}-1)^2, (\sqrt{p/n}+1)^2]$。当 $p \gg n$，这个区间的尺度是 $p/n$。所以 $\rho_1$ 的典型特征值 $\sim p/n$。

$\rho_1$ 来自 $U$ 分解中的第一项 $G G^T / n$。$G$ 是一个 $p \times n$ 的（等价）高斯矩阵，来自训练数据。所以这个区间的尺度是 $\psi_p/\psi_n \sim p/n$。

我们可以理解当参数量远大于数据量时，数据只能"填满"参数空间的一小部分。$GG^T/n$ 的谱结构反映了"数据 vs 参数"的维度对比。这个对比决定了记忆时间尺度。

3. $\rho_2$：另一个分布在尺度 $\psi_p$ 的连续谱。它和数据无关（只依赖 $\Sigma$），尺度远大于 $\rho_1$。

### 总结
现在回看整个逻辑：
1. 设置：RFNN 是两层网络，只训第二层。训练损失是参数 $A$ 的二次函数。
2. 动力学解：梯度下降的闭式解表明时间尺度由矩阵 $U$ 的特征值决定。
3. GEP 简化：高斯等价原理把非线性 $U$ 等价为线性高斯矩阵，使随机矩阵理论可以应用。
4. 谱计算：用复本方法（或自由概率）计算 $U$ 的 Stieltjes 变换，得到 Theorem 3.1 的三个方程。
5. 谱分裂：在过参数化极限下谱分裂成两个 bulk，尺度比约为 $\psi_n = n/d$。
6. 时间尺度分离：大 bulk $\rho_2$ → $\tau_{\text{gen}} \sim O(1)$；小 bulk $\rho_1$ → $\tau_{\text{mem}} \sim O(n)$。
7. 中间状态：Proposition C.2 证明在这两个时间尺度之间，测试损失 = 训练损失，即模型泛化。
8. 额外发现：得分误差在泛化窗口内按 $\psi_n^{-\eta}$ 幂律衰减。
$U$ 的谱分裂到两个尺度比为 $n/d$ 的 bulk。这一件事推出所有其他结论——两个时间尺度的存在、各自的 $n$-依赖性、泛化窗口的存在、窗口长度 $\propto n$。

## Thinking 
Early stopping 不是技巧而是必需。在监督学习里 early stopping 是"防止过拟合的可选技巧"。在扩散模型里它是绕开损失函数本身坏性质的必需手段。

这个结论不能推广到语言模型。语言模型（LLM）也会记忆训练数据，但机制不同：扩散模型的损失函数最优解本身就是记忆解——这是数学必然。而 LLM 训练的是下一 token 预测，全局最优是真实条件概率 $P(x_{t+1} | x_1, ..., x_t)$——这是一个泛化的目标，不是记忆。LLM 的记忆主要来自训练数据的重复——出现多次的序列被记住。所以 LLM 的记忆防护策略是数据去重、差分隐私训练等，和本文的 early stopping 策略不同。本文的 $\tau_{\text{mem}} \propto n$ 标度律和 $(n, p)$ 相图只对扩散模型成立。