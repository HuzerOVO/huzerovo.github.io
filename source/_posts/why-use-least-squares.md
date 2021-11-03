---
title: 为什么使用最小二乘法
toc: true
mathjax: true
date: 2021-11-04
updated: 2021-11-04
categories:
   - [机器学习]
tags:
   - 最小二乘法
   - 概率论
   - 算法
   - 咕咕咕ing
---

> 解释为什么最小二乘法能减小误差

## 了解一下误差
假设预测结果与实际结果存在误差，即
$$
\begin{equation}
\label{eq1}
y^{(i)} = \theta^{T} x^{(i)} + \varepsilon^{(i)}
\end{equation}
$$
其中 $ \varepsilon^{(i)} $ 就称为误差

如果 $ \varepsilon^{(i)} $ 存在某种分布，
于是我们可以假设这个分布为 **高斯分布** ，或者说是 **正态分布** [^the-reason]  
即：
$$
\varepsilon^{(i)} \sim N(0, \sigma^{2})
$$

于是有：
$$
\begin{equation}
\label{eq2}
P(\varepsilon^{(i)}) = \frac{1}{\sqrt{2 \pi} \sigma} \exp{(- \frac{ ( \varepsilon^{(i)} )^{2} }{2 \sigma^{2} })}
\end{equation}
$$

[^the-reason]: 根据 **中心极限定理** 可知，误差的分布在大多数情况下都成正态分布，这是一个合理的假设

## 最大化预测期望等价于最小化误差
由 $\eqref{eq1}$ 和 $\eqref{eq2}$ 得[^what-is-it]：
$$
\begin{equation}
\label{eq3}
P(y^{i} \mid x^{i} ; \theta)
= \frac{1}{\sqrt{2 \pi} \sigma} \exp{(- \frac{ ( y^{(i)} - \theta^{T} x^{(i)} )^{2} }{2 \sigma^{2} })}
\end{equation}
$$

于是得到：
$
(y^{i} \mid x^{i} ; \theta) \sim N(\theta^{T} x^{(i)}, \sigma^{2})
$

我们又假设， $\varepsilon^{i}$ 之间相互独立，且具有相同的分布，
然后我们使用 **极大似然估计** 将 $\eqref{eq3}$ 最大化[^is-max-not-min]

[^what-is-it]: 这是 **极大似然估计** 里的表示方式， $\theta$ 表示一个参数，
随后求得 $\theta$ 使 $P(y^{i} \mid x^{i})$ 最大

[^is-max-not-min]: 应当注意： $\eqref{eq3}$ 中自然常数的指数为负数

设
$$
\begin{equation}
\begin{split}
L(\theta)
& = P(Y \mid X ; \theta) \\\\
& = \prod_{i=1}^{m} P(y^{i} \mid x^{i} ; \theta) \\\\
& = \prod_{i=1}^{m} \frac{1}{\sqrt{2 \pi} \sigma} \exp{(- \frac{ ( y^{(i)} - \theta^{T} x^{(i)} )^{2} }{2 \sigma^{2} })}
\end{split}
\end{equation}
$$

那么
$$
\begin{equation}
\begin{split}
l(\theta)
& = \log{L(\theta)} \\\\
& = \sum_{i=1}^{m} \log{ \frac{1}{ \sqrt{2 \pi} \sigma } \exp{( - \frac{( y^{(i)} - \theta^{T} x^{(i)} )^{2}}{ 2 \sigma^{2} } )}} \\\\
& = m \log{\frac{1}{\sqrt{2 \pi} \sigma}} + \sum_{i=1}^{m} -( \frac{y^{(i)} - \theta^{T} x^{(i)}}{2 \sigma^{2}} )
\end{split}
\end{equation}
$$

由于 $\sigma^{2} > 0$ 恒成立，
故最大化 $l(\theta)$ 等价于最小化 $\sum_{i=1}^{m} \frac{y^{(i)} - \theta^{T} x^{(i)}}{2}$

这也就是 {% post_link linear-regression %} 一文中提到的 **cost function**