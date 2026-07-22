---
title: Elements_of_Causal_Inference
publishDate: 2026-7-2 15:29:00
description: '部分章节略读'
tags:
  - causal inference
language: '中文'
---

[Elements_of_Causal_Inference](https://mitp-content-server.mit.edu/books/content/sectbyfn?collid=books_pres_0&id=11283&fn=11283.pdf) opensource

这本书和之前Brady Neal课程的视角有一些区别，Neal的课程倾向于**通过多个假设控制其他变量，当我改变单一变量的时候真正影响结果的是因**，也就是在确定的SCM上通过机制来确定因果带来的影响。这本书的视角更倾向于**因果是不跟随环境变化的，在不同的环境中稳定保持不变的才是因果**，这个观点的关键在于我们如何通过不同的环境找到真正的SCM。我感觉干预和环境变化其实没什么大的差别，只不过是解释的方式不同。

## chapter2 Assumptions for Causal Inference

### ICM
类似之前的局部马尔科夫假设，即 $p(x_j\mid x_{PA_j})$ 

对于二变量 $p(c,e)=p(c)p(e∣c)$ ICM认为 $P_C$ 与 $P(E∣C)$ 是独立选择的
也就是说机制和输入无关（即和环境无关）
$$
p^e(a,t)=p^e(a)p(t∣a)
$$

机制变化具有独立性，机制之间使用的信息也应该具有独立性，SCM方程中的噪声也应该具有独立性。

改变系统的一部分时，未被干预的机制仍然保持不变。

**SMS**

当切换环境、施加外部干预时，只有很少一部分机制会发生改变，绝大多数机制保持不变。由ICM导出

### multi-environment

多环境指的是改变系统的一部分时，未被干预的机制仍然保持不变，但是对某些变量进行干预。可以通过环境变化找到真正的因果关系。

多环境分为两种
1. 干预目标已知：知道每个环境修改了哪些节点；
2. 干预目标未知：只知道数据来自不同环境，但不知道具体哪里发生了变化

## chapter7 Learning Multivariate Causal Models

### ICP
Invariant Causal Prediction
主要解决干预目标未知的问题，循环早某个目标变量的直接原因

ICP 的理论起点：真正父节点给出的条件预测关系应跨环境保持不变
$$
p^e(Y \mid X_{PA(Y)})=p^f(Y \mid X_{PA(Y)})
$$

在不知道真实父节点的情况下，枚举后选集和，并对每个集合验证，最后取交集。可以用线性模型建模，检验的是是否存在同一个回归方程和同一个残差分布，可以同时解释所有环境。准确性取决于多个环境是否提供了不同的有效变化

### discovery方法

**Constraint-based discovery**

使用条件独立关系

典型假设：
- causal Markov；
- faithfulness；
- 无隐藏混杂或对隐藏混杂进行额外处理。

通常只能识别 Markov equivalence class，而不能完全定向。

**Functional causal models**

加入函数形式限制，例如：

$$
Y=f(X)+N,N⊥X.
$$

包括：
- additive noise models；
- LiNGAM；
- nonlinear ANM；
- equal error variance models。

利用“反方向通常不能满足同样简单结构”来确定方向。

一般 SCM 不可识别；非线性 ANM、非线性 causal additive models 以及某些等误差方差模型可以获得更强的 DAG 可识别性。

**Multi-environment discovery**

利用：

$$
P^e(Y∣X_S)
$$

是否跨环境稳定。

其优势是：

不一定需要知道干预目标；
可以利用自然实验和异质数据；
能针对一个目标变量寻找稳定的直接原因。

其限制是：

需要多个具有实质差异的环境；
通常是 target-wise，而不是直接恢复整个 DAG；
invariance 不等于充分识别；
若 Y 的机制本身发生变化，基本假设失效。

## chapter8 Connections to Machine Learning, II

类似与transparency：因果机制稳定，因此基于因果机制的预测更可能跨分布泛化。

本章提到了一些利用因果关系来帮助或者调整机器学习的机制，比如半监督学习由结果反推原因时，无标签数据往往更有用。反过来可能用处不大。训练数据和测试数据不一样的时候要区分是数据比例不一样还是系统规律不一样（个人感觉这个对DT比较有帮助）。以及如何利用共享的噪声去噪等等。	​

一个有趣的视角是RL的policy选action其实是干预，因为神经网络只根据固定的状态输入输出动作，因此自然的排除了其他隐藏的混杂变量。

本章是一些比较实用的算法之类的，因果结构告诉机器学习：什么应该被去除，什么可以被压缩，什么在策略改变后需要重新估计，以及什么能够跨环境复用。主要是以下几点：
1. 因果知识可以添加进入机器学习的回归式子，让回归对某个变量更精确
2. off-policy evaluation 是机制替换问题。
3. 好的状态表示应保留对奖励和未来状态有因果影响的信息。
4. 跨域泛化的关键不是寻找所有环境中相关性最高的变量，而是寻找条件机制稳定的变量。