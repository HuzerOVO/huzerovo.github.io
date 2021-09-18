---
title: 梯度下降算法
toc: true
mathjax: false
categories:
  - - 机器学习
tags:
  - 机器学习
  - 算法
  - 咕咕咕ing
date: 2021-09-16 22:25:42
---


> 介绍 **梯度下降算法 (gradient descent algorithm)** 并简单解释其原理

## 步骤

1. 随机选取一个初始位置 $x_{0}$  
2. 对 $x_{0}$ 进行更新，即：

   $$
       x_{1} = x_{0} - \alpha \frac{\mathrm d}{\mathrm dx}f(x_{0})
   $$

   其中 $\alpha$ 是一个正数，称作 “步长”，
   $\frac{\mathrm d}{\mathrm dx}f(x_{0})$ 是 $f(x)$ 在 $x_{0}$ 处的导数
3. 重复进行 `步骤2` 中的更新操作，最终得到 $x_{n}$ 使得 $f(x_{n})$ 最小

## 数学原理简析

### 基本原理

设函数 $f(x)$ 为二次函数，且在 $x = x_{min}$ 处取得最小值  
在 $x_{n}$ 点处的导数 $\frac{\mathrm d}{\mathrm dx}f(x_{n})$ 为 $k_{n}$  
随机选取一点 $x_{0}$  
1. 若 $k_{0} < 0$ ，则 $x_{0} < x_{min}$，  
   令 $x_{1} = x_{0} - k_{0}$ ，则 $x_{1} < x_{0}$  
   即 $x_{1}$ 向 $x_{0}$ 到 $x_{min}$ 的方向移动 

2. 若 $k_{0} > 0$ ，则 $x_{0} > x_{min}$，  
   令 $x_{1} = x_{0} - k_{0}$ ，则 $x_{1} > x_{0}$  
   即 $x_{1}$ 向 $x_{0}$ 到 $x_{min}$ 的方向移动

可知，更新后的 $x_{n}$ 一定从 $x_{n - 1}$ 向 $x_{min}$ 的方向移动

### 使用条件

在基本原理中可以发现，获得 $x_{n}$ 的过程不依赖于原函数  
只需要知道原函数的一阶导函数  

因此，在原函数为二次函数且存在最小值，导函数比原函数更容易获得时  
可以考虑采取梯度下降算法

### 不足之处

设 $f(x) = x^{2}$ ，则  
$k_{n} = \frac{\mathrm d}{\mathrm dx}f(x_{n}) = 2n$  
函数在 $x_{min} = 0$ 处取得最小值

取初始值 $x_{0} = 1$ ，则 $x_{1} = x_{0} - k_{0} = -1$  
继续对 $x_{1}$ 进行更新，则 $x_{2} = x_{1} - k_{1} = 1 = x_{0}$  

这导致无论如何更新，最终结果都只是 $x = 1$ 和 $x = -1$ 两个值，永远无法得到 $x_{min} = 0$

### 改进

因此，需要引入 $\alpha$ 控制 $x$ 变化的幅度  
$\alpha$ 的值越小，需要进行递减的次数越多，这才能保证得出的结果接近最小值 $x_{min}$  
一般来说， $\alpha$ 是一个小于 1 但大于 0 的值  
具体的值需要根据实际情况进行调整

## 代码示例

在示例中  
设函数为 $f(x) = 3x^{2} + 4x + 5$  
则 $\frac{\mathrm d}{\mathrm dx} f(x) = 6x + 4$  
最低点为 $x_{min} = - \frac{B}{2A} = - \frac{2}{3}$

```python
#!/usr/bin/env python3

import numpy as np

a = 3
b = 4
c = 5

# f(x) = ax^2 + bx + c
def f(x):
    return a*(x**2) + b*x + c

# df(x) = 2ax + b
def df(x):
    return 2*a*x + b

def main():
    # 最低点
    xmin = - b/(2*a)
    # 步长
    alpha = 0.1
    # 随机初始值
    x = np.random.randint(5)
    # 递减 100 次
    for i in range(100):
        x = x - alpha * df(x)
    
    # 输出结果
    print("x_n is:" , x)
    print("x_min is:", xmin)

if __name__ == "__main__":
    main()
```
