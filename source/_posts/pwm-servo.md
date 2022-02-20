---
title: PWM 舵机控制
date: 2021-03-18
updated: 2021-10-29
toc: true
mathjax: true
categories:
   - [树莓派]
tags:
   - Python
   - Servo
   - PWM
---

> 介绍如何在树莓派上控制舵机(servo)

### PWM 基础知识

> PWM（Pulse Width Modulation）即脉宽调制  
> 是一种利用数字信号对模拟电路进行控制的一种方法  
> 详情自行搜索，~~主要是我也不会~~

#### 频率

指 $ 1s $ 内，脉冲信号变化（高电平 $ \to $ 低电平 $ \to $ 高电平）的次数

#### 周期

脉冲信号完成一次变化的时间称作一个周期

#### 频率与周期的关系

$$
f = \frac{1}{T}
$$

一个周期为 $ 20ms $ 脉冲信号，其频率为 $ \frac{1}{20 \times 10^{-3}s} $，即 $ 50Hz $

#### 占空比

在一个周期内，高电平持续的时间与整个周期的总时间的比例为占空比  
设整个周期的时间为 $ T $，高电平持续的时间为 $ t $，则占空比 $ D $ 为：

$$
D = \frac{t}{T} \times 100\\%
$$

或

$$
D = t \cdot f \times 100\\%
$$

一个周期为 $ 20ms $ 的脉冲信号，其中高电平持续时间为 $ 1.5ms $，
则占空比为 $ \frac{1.5ms}{20ms} \times 100\\% = 7.5\\% $

### PWM 控制舵机

使用 PWM 控制舵机时，一般使用频率为 $ 50Hz $ 的 PWM 信号  
在 $ 180^{\circ} $ 的舵机中，一般以 $ 1.5ms $ 的高电平持续时间作为基准  
高电平持续时间 $ 1.5ms\pm1ms $ 作分别为舵机旋转 $ \mp90^{\circ} $ 的控制信号

据此可得出相应的占空比，并通过设置 PWM 信号的占空比来控制舵机转动的角度
| 高电平持续时间 |   占空比    | 角度（顺时针为正） |
|:---------------|:------------|:-------------------|
|  $ 0.5ms $     | $ 2.5\\% $  | $ +90^{\circ} $    |
|  $ 1.5ms $     | $ 7.5\\% $  | $ 0^{\circ} $      |
|  $ 2.5ms $     | $ 12.5\\% $ | $ -90^{\circ} $    |

### 使用 Python 实现 PWM 控制

树莓派上的 GPIO 可实现 PWM，~~虽然抖动很大~~  
可以通过 Python 来对这些 GPIO 接口进行控制

#### 使用 RPi.GPIO 库

参考 [PRi.GPIO PyPI](https://pypi.org/project/RPi.GPIO)

##### 导入 GPIO 包并初始化 GPIO 接口

``` python
import RPi.GPIO as GPIO

# 使用编号 18 的 GPIO 接口
pin = 18

# 设置 GPIO 的编号模式，详情参考树莓派官网
GPIO.setmode(GPIO.BCM)

# 关闭奇奇怪怪的警告
GPIO.setwarnings(False)

# 初始化接口
GPIO.setup(pin, GPIO.OUT, initial=False)
```

##### 创建 PWM 实例

``` python
# 创建实例
hz = 50
p = GPIO.PWM(pin, hz)

# 设置占空比
# 单位 ms
t = 1.5
# 乘以0.1,单位修正为 s，与 Hz 单位匹配
dc = t * hz * 0.1

# 启动 PWM
p.start(dc)

# 更改占空比
p.ChangeDutyCycle(dc)

# 停止 PWM
p.stop()

# 在程序结束时要释放对 GPIO 接口的控制
GPIO.cleanup()
```

##### 会产生的问题

正如前面所说，使用 PRi.GPIO 实现的 PWM 抖动很大，
因为这属于软件控制的 PWM，在频繁地更改占空比时会更明显，
在使用 `p.ChangeDutyCycle(dc)` 函数时也可能发生抖动，导致舵机不能转到指定角度  
在 `p.start(dc)` 后，即使没有任何操作，也仍可能发生抖动  
同时控制多个舵机时，不同接口还会产生干扰

##### 解决方法

一种解决的方法是使用 `time` 模块中的 `sleep()` 函数进行延时

``` python
p.start(dc)
sleep(0.5)
for i in range(0.5, 2.5 + 0.1, 0.1):
    dc = i * hz * 0.1
    p.ChangeDutyCycle(dc)
    sleep(0.5)
p.stop()
```

但这种方法并不能解决在空闲时出现的抖动  
也不能去除其他端口的干扰

另一种方法则是在启动 PWM 后，进过延时就将其关闭

``` python
p.start(dc)
sleep(0.5)
for i in range(0.5, 2.5 + 0.1, 0.1):
    dc = i * hz * 0.1
    p.start(dc)
    sleep(0.5)
    p.stop()
p.stop()
```

这种方法需要频繁的开启关闭 PWM，且仍不稳定，但在空闲时不会发生抖动

还有一种方法：使用 pigpio 库

#### 使用 pigpio

使用 pigpio 库，可以实现硬件方式控制的 PWM，
能显著地提高 PWM 的抗干扰性  

参考 [pigpio library](http://abyz.me.uk/rpi/pigpio/python.html)

##### 启动服务

使用 pigpio 库需要启动 pigpiod 服务

```shell
sudo systemctl start pigpiod.service
```

##### 导入包并初始化

```python
import pigpio

# 使用编号 18 的接口
# 注意：使用接口的 BCM 编号
pin = 18

# 创建连接 pigpio 连接
pi = pigpio.pi()

# 设置 GPIO 接口输出模式
pi.set_mode(pin, pigpio.OUTPUT)
```

##### PWM 设置

```python
# 设置 PWM 频率为 50 Hz
pi.set_PWM_frequency(pin, 50)

# 设置 PWM 范围 0~100
# 与占空比相结合，范围 0~100
pi.set_PWM_range(pin, 100)

# 设置占空比设置为 25 即 25/100 = 1/4
pi.set_PWM_dutycycle(25)

# 关闭 PWM
pi.set_PWM_dutycycle(0)
```

##### 优缺点

使用 pigpio 时必须启动 pigpiod 服务，
但使用这个方式输出的 PWM 是硬件方式控制的，比较稳定，不易受到干扰
