---
title: engineering optimization method (Updating)
publishDate: 2026-09-12T20:30:00+08:00
description: '课程笔记'
tags:
    - 运筹学
---

## INTRO

运筹学（Operations Research, OR）利用数学方法对决策问题建模，在资源约束下寻找最优或较优方案。

$$
\min_{\boldsymbol{x}} \text{ or } \max_{\boldsymbol{x}} \; f(\boldsymbol{x})
$$
$$
\text{s.t.}
\begin{cases}
g_i(\boldsymbol{x}) \le 0, & i=1,2,\dots,m \\
h_j(\boldsymbol{x}) = 0, & j=1,2,\dots,p \\
\boldsymbol{x} \in X
\end{cases}
$$

- $\boldsymbol{x}$：决策变量向量
- $f(\boldsymbol{x})$：目标函数
- $g_i(\boldsymbol{x}) \le 0$：不等式约束
- $h_j(\boldsymbol{x}) = 0$：等式约束
- $\text{s.t.}$ = subject to，受约束于
- $X$：决策变量的可行集合

运筹学模型的最优解仅对这个特定的模型有效。对问题的抽象简化能力是建模的关键

### 解决问题的步骤

1. 定义问题并收集数据
2. 构造运筹学数学模型
3. 模型求解
4. 模型验证
5. 应用准备和具体措施

## Linear Programming (线性规划)

可行解：满足约束条件
最优解：最好的可行解

线性规划总可以在可行解空间的角点处找到最优解

### LPM 线性规划（标准型）

$$
\max \quad \boldsymbol{c}^T \boldsymbol{x}  \ 
$$

$$
\text{s.t.}
\begin{cases}
A\boldsymbol{x} = \boldsymbol{b} \ \text{ where }A \in \mathbb{R}^{m\times n} \\
\boldsymbol{x} \ge \boldsymbol{0}
\end{cases}
$$

- $\boldsymbol{c}$：价值系数向量
- $\boldsymbol{x}$：决策变量向量
- $A$：约束系数矩阵，一个良好的模型还会满足 $rank(A) = m$（保证约束唯一）
- $\boldsymbol{b}$：右端常数向量，要求 $\boldsymbol{b}\ge \boldsymbol{0}$
- $\boldsymbol{x}\ge \boldsymbol{0}$：变量非负约束

例子可以是网络的最大流问题

#### 等式约束

如果碰到不等式，使用：
1. 松弛变量
2. 剩余变量
规划为等式

#### 决策变量

单纯形法要求所有决策变量非负，定义：
$$
x = x^+ - x^-
$$
其中右侧变量均大于等于0，但是不能同时取正值，也就是至少一个为0

### Simplex method 单纯形法

一般线性问题规划问题的代数方法，方法的几何解释是发现最优的角点

$A\boldsymbol{x}=\boldsymbol{b}$ 的解 $x$ 至少有 $n-m$ 个零变量 (非基变量)，剩下需要求解的变量叫做基变量，如果基变量没有唯一解，则需要重新选择非基变量（$A$ 满秩的情况下总有唯一解）。

如果有唯一解，则称解为基解(basic solution), basic solution 是corner point的代数表示。基解（角点）的最大数目是$\binom{n}{m}$

**单纯形法之需要考察部分基解，就能找到问题的最优解**

> Q: 为什么我们认为线性规划求出来的是全局最优而不是局部呢？

> A：LP 可行域是有限个半空间的交集；每一个半空间都是凸集；凸集相交仍然是凸集，所以最后得到的集合是凸集。而且目标是线性的。（隐约记得上学期退掉的凸优化讲过）

- entering variable (进基变量)
- leaving variable (离基变量)



