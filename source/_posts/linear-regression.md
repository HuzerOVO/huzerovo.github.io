---
title: Linear Regression - 线性回归
toc: true
mathjax: true
date: 2021-09-11 15:23:18
categories:
   - [机器学习]
tags:
   - 机器学习
   - 监督学习
   - 线性回归
   - 咕咕咕ing
---

本文基于 Andrew NG 机器学习 公开课程 [^andrew-ng]

[^andrew-ng]: 课程链接 <https://see.stanford.edu/Course/CS229>

线性回归是一种 **监督学习 (supervised leaning)** 的模型
本文简单解释其中的数学原理

## 基础

### 定义 “特征” 与 “标签”

这里的定义是为了方便后面的表达

将监督学习中的一些概念 “重命名”
- 特征，定义为输入空间（特征空间）
- 标签，定义为输出空间

对 “特征” 与 “标签” 的理解  
1. 从算法定义上
   - 特征可理解为输入
   - 标签可理解为输出
   - 算法的目的就是根据输入，给出输出

2. 从直观感觉上
   - 特征指代一个对象的特点
   - 标签指代一个对象
   - 算法的目的即根据特点，猜测对象

### 定义 “特征-标签” 关系

实际生活中，我们知道了 $A, B$ 能够在某种程度上决定 $C$  
且 $A, B$ 各自对 $C$ 的影响有多大，我们并不清楚  

也就是说，我们知道 “特征” 与 “标签” 之间存在某种关系，但无法准确地描述这个关系

此时我们可以对 $A, B, C$ 建立一个关系  
即：能由 $A, B$ 推出 $C$  
使用数学符号表示： $(A, B) \Rightarrow C$  

若将 $A$ 和 $B$ 称做 $C$ 的特征  
并且将 $C$ 定义为标签  
将 $(A, B) \Rightarrow C$ 称为 “特征-标签” 关系  
则称数据 $(A, B, C)$ 存在 “特征-标签” 关系  

### 提出问题

我们希望，给出一组存在 “特征-标签” 的关系的数据  
计算机能够根据这组数据，建立一个存在 “特征-标签” 关系的模型  
随后，给出 “特征” ，计算机能够根据模型，给出比较可靠的、推测的 “标签”

### 使用数学语言描述问题

接下来将问题抽象成为一个数学问题

设所有的特征的集合为 $(x_{1}, x_{2}, \cdots, x_{n})$，其中，$n$ 表示特征的数量  

在比较简单的情况下  
假定 “特征-标签” 是线性关系  
并且这个线性关系为函数 $h$  
则称 $h$ 为 **假设函数 (hypotheses)**  

于是得到的函数

$$
\begin{equation}
   h_{\theta}(x) = \theta_{0} + \theta_{1} \cdot x_{1} + \theta_{2} \cdot x_{2} + \cdots + \theta_{n} \cdot x_{n}
\end{equation}
$$

其中， $\theta_{0}, \theta_{1}, \cdots, \theta_{n}$ 表示 **参数 (parameters)**，或者说是 **权值 (weights)**  
这是我们需要求得的值

### 引入向量进行表达

若用向量表示 $\theta$ ：

$$
\vec{\theta} = 
\begin{pmatrix}
   \theta_{0}\\\\
   \theta_{1}\\\\
   \vdots\\\\
   \theta_{n}
\end{pmatrix}
$$

并将特征集合重新定义为特征向量：

$$
\vec{x} =
\begin{pmatrix}
   x_{0}\\\\
   x_{1}\\\\
   \vdots\\\\
   x_{n}
\end{pmatrix}
$$

其中 $x_{0} = 1$  
**为简化符号，记 $\vec{\theta}$ 为 $\theta$ ，$\vec{x}$ 为 $x$**  
则函数可表示为：

$$
\begin{equation}
   h_{\theta}(x) = \sum_{i=0}^{n}{\theta_{i} x_{i}} = \theta^{T}x
\end{equation}
$$

此时我们的问题就是，求出一个 $\theta$ ，使得函数 $h$ 能够很好地描述 “特征-标签” 关系  
即建立一个合理的模型

## 进一步分析问题

### 引入数据集与训练集的概念

上面的函数仅表示单个数据中的 “特征与标签” 关系  
若存在多个数据，即 **数据集 (dataset)**  
且所有的数据都存在 “特征-标签” 的关系  
即可以构建出一个 **训练集 (training set)**  
我们可以使用 **训练集** 对算法进行 **训练** ，最终得出 $\theta$

定义下面的符号，方便之后的表示：  
设 **训练集大小** 为 $m$ ，令：  
$x_{i}^{(j)}$ 表示训练集中，**第 $j$ 项数据** 的 **第 $i$ 个特征**  
$x^{(j)}$ 表示训练集中 **第 $j$ 项数据** 的特征向量

### 引入最小二乘法转化问题

回顾我们的问题：  
我们希望得到一个函数 $h$ 能够很好地描述训练集中的每个数据中 “特征-标签” 的关系  
换句话说，给出 “特征” ，若 “特征” 存在于训练集中，那么
由 **函数 $h$** 得出的 “标签”，与训练集中的 “标签”，它们之间的误差最小  
根据 **最小二乘法** ，得：

$$
\begin{equation}
   J(\theta) = \frac{1}{2} \sum_{i=1}^{m} \left( h_{\theta}(x^{(i)}) - y^{(i)} \right)^{2}
\end{equation}
$$

函数 $J$ 也称为 **cost function**

于是问题转化为：  
需要得到一个 $\theta$ ，使得 $J(\theta)$ 的值最小

## 解决问题

接下来介绍解决这个问题的可行方法

### 梯度下降

为了得到 $\theta$ 使函数值 $J(\theta)$ 最小，可以采取 **梯度下降 (gradient descent)** 的方法：  
随机选取一个 $\theta$ 值，随后不断地更新 $\theta$ ，使 $J(\theta)$ 不断减小  
当 $J(\theta)$ 的值达到最小时，得到的 $\theta$ 即是问题的解

问题在于，如何更新 $\theta$ 才能使得 $J(\theta)$ 的值不断减小  
我们可以这样做[^why]：  

[^why]: 其中的原理在于：
求导后的值为函数斜率，其正负代表方向，当一个数减去这个值时，这个数会沿 $x$ 轴往函数值减小的方向移动

$$  
\begin{equation}
   \theta_{j} := \theta_{j} - \alpha \frac {\partial}{\partial \theta_{j}} (\theta)
\end{equation}
$$

其中 $:=$ 表示赋值，将右边的值赋予左边，而 $=$ 是真值判断，表示等号两边的值或结果是相同的  
$\alpha$ 表示 **学习率 (learning rate)**，或者称 **步长**，是自定义的常数  

对 $J(\theta)$ 进行偏导：
$$
\begin{equation}
\begin{split}
   \frac{\partial}{\partial \theta_{j}} J(\theta)&= \frac{\partial}{\partial \theta_{j}} \frac {1}{2} (h_{\theta}(x) - y)^2 \\\\&= 2 \cdot \frac{1}{2}(h_{\theta}(x) - y) \cdot \frac{\partial}{\partial \theta_{j}} (h_{\theta}(x) - y) \\\\&= (h_{\theta}(x) - y) \cdot \frac{\partial}{\partial \theta_{j}} \left( \sum_{i=0}^{n} \theta_{i} x_{i} - y \right) \\\\&= (h_{\theta}(x) - y) x_{j}
\end{split}
\end{equation}
$$

代入 **式 $(4)$** ，得：

$$
\begin{equation}
   \theta_{j} := \theta_{j} - \alpha (h_{\theta}(x) - y) x_{j}
\end{equation}
$$

这是单个数据的表示，引入上面介绍的训练集  

$$
\begin{equation}
   \theta_{j} := \theta_{j} - \alpha \sum_{i=1}^{m} \left(h_{\theta}(x^{(i)}) - y^{(i)}\right) x_{j}^{(i)}
\end{equation}
$$

对每一个在向量 $\theta$ 中的分量 $\theta_{i}$ 都进行上述运算  
最终我们会得到一个 $\theta$ ，这个值就是我们希望求得的值

### 牛顿法

咕咕咕

### 解析解法

咕咕咕
