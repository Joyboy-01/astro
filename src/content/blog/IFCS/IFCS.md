---
title: Introduction and Frontiers of Cognitive Science (Updating)
publishDate: 2026-09-12T20:30:00+08:00
description: '课程笔记'
tags:
    - cogintive science
---

[课程网页链接](https://clcs-sustech.github.io/zh/course/cse5026/)

## Course Introduction & Overview

### background

Psychology, Gestalt, Behaviorism, S-R theory, Information Processing Theory

### Methodology

- Internalism

    心智状态完全由大脑内部的物理状态决定，过于主观
    1. Reductionism
    2. Functionalism
    3. Introspectionism
    4. Cognitivism / Symbolism

- Externalism

    心智状态部分由外部世界决定
    1. Behaviorism
    2. connectionism

#### Technology
 - Brain-related Technology
 - Behavioral Studies
 - Tools; Experimental designs; Math and statistics.

### Supplementar: CPM & Hypothesis Testing

对于两类假设
- $H_0$：原假设（null hypothesis）
- $H_A$：备择假设（alternative hypothesis）

以及显著性水平（significance level）
- $\alpha$

假设检验通常包括：
- Type I error（$α$）：$H_0$ 为真，却错误拒绝 $H_0$；
- Type II error（$β$）：$H_A$ 为真，却没有拒绝 $H_0$；
- Power：$H_A$ 为真，并成功拒绝$H_0$。

$\alpha$ ,$\beta$, $power$ 的关系是：

- $\beta$：在计算Type I error 使用 $\alpha$ 确定临界值后，用 $H_A$ 的分布算出没有在拒绝区域的概率：
$$
\beta=P_{\mu_A}(Z\le 1.645) \text{   OR   } 
\beta=P_{\mu_A}\left(\bar X\le c_\alpha\right)
$$
- $power=1−β$
- 增加样本量会提高 power：标准误变小，样本均值的分布变窄，因此更容易超过拒绝阈值。
- 一般性的样本量与 power 关系，适用于许多正态近似的假设检验：在给定噪声大小、显著性水平、目标 power 和最小可检测效应后，需要多少样本。
    
    $\Delta$：希望检测到的真实效应大小
$$
n\ge
\frac{\sigma^2\left(z_{1-\beta}+z_{1-\alpha}\right)^2}{\Delta^2}
$$
