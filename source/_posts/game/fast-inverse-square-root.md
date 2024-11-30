---
title: 快速求平方根的倒数
date: 2023-11-23
categories: ["game"]
tags: ["Algorithm"]
references:
	- '[Fast inverse square root - Wikipedia](https://en.wikipedia.org/wiki/Fast_inverse_square_root)'
	- '[Fast Inverse Square Root — A Quake III Algorithm - YouTube](https://www.youtube.com/watch?v=p8u_k2LIZyo)'
---

# Fast inverse square root

快速平方根算法，是一种在 IEEE 754 浮点格式下估计 $\frac{1}{\sqrt{x}}$ 的方法。

## Most famous case

该算法最出名的是 1999 年 [约翰·卡马克](https://zh.wikipedia.org/wiki/約翰·卡馬克) 在《[Quake III Arena](https://en.wikipedia.org/wiki/Quake_III_Arena)》中的实现，其代码如下：

```cpp
float Q_rsqrt( float number )
{
	long i;
	float x2, y;
	const float threehalfs = 1.5F;

	x2 = number * 0.5F;
	y  = number;
	i  = * ( long * ) &y;						// evil floating point bit level hacking
	i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
	y  = * ( float * ) &i;
	y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
//	y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed

	return y;
}
```

以上代码中起关键作用的代码如下：

- `i  = * ( long * ) &y;` ：这是在将浮点数转换为 long 整型。
- `i  = 0x5f3759df - ( i >> 1 );` ：通过一番魔鬼操作，得到 $\frac{1}{\sqrt{x}}$ 近似解，下面会讲到。
- `y  = * ( float * ) &i;` ：这是在将整型转换回浮点型。
- `y  = y * ( threehalfs - ( x2 * y * y ) );` ：用牛顿法迭代更精确的结果，上面代码中只用了一次牛顿法就得到了符合 float 类型精度要求的结果，因此注释掉了后面的代码。

## Motivation

浮点数的平方根倒数通常用于对矢量进行归一化，这在计算机图形学中有着广泛的应用，但浮点数平方根和除法相对于乘法成本较高，快速平方根倒数算法绕过了这两个步骤，从而具有性能优势。

## Algorithm

### IEEE 754 Float

要理解这个算法，首先要了解 **IEEE 754** 规范下的 32位（单精度）浮点型存储方式。

- 符号位（sign bit）：0 表示正数，1 表示负数，通常在最高位。科学计数法存储，整数位永远为1。
- 指数位（exponent）：有8位（double float 为11位），可以为值 0~255（$2^{8} - 1$），值127表示指数为0，因此能表示 $2^{-126}$ 到 $2^{127}$。
- 尾数位（mantissa）：存储数值的实际有效数字，有 23位，尾数用来表示一个介于 1~2（不包括 2） 之间的数，但最高位（隐含的”1”）通常不存储，因为除了在特殊情况（如0、infinities、NaN）外，它总是1。

![IEEE 754 Float](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0d/Float_w_significand_2.svg/1180px-Float_w_significand_2.svg.png)

该浮点数的值计算公式如下：

$$
x = (-1)^{Si} \cdot (1 + \frac{M}{L}) \cdot 2^{(E - B)} \tag{1} % Float
$$

其中常数 $L = 2^{23}$ （即尾数 23 位能表示的最大值 $+ 1$），$B = 127$（即指数位能表示的最大值 $/ 2 - 1$）。

对于图中的浮点数，符号位 $Si = 0_2 = 0$，指数位$E = 0111\ 1100_2 = 124$，尾数位 $M = 010\ 0000\ 0000\ 0000\ 0000\ 0000_2 = 0.25 \times 2^{23}$

带入公式得当前浮点数 $x =  1 \cdot (1 + 0.25) \cdot 2^{-3} = 0.15625$。

当我们对这浮点数作为 int 进行类型重解释时，得如下结果：

$$
I_{x} = (-1)^{Si} \cdot (E \times L + M) \tag{2} % Int
$$

### Aliasing to an integer as an approximate logarithm

对于想求的浮点数 $y = \frac{1}{\sqrt{x}}$ 的 x，由于应用场景限制，不会出现需要对平方根进行解析延拓的情况，因此可以认为 x、y 中标志位 $Si = 0$ 恒成立。

再对公式 1 的 $x = (1 + \frac{M}{L}) \cdot 2^{(E - B)}$ 取对数，得到 $\log_{2}{x} = E - B + \log_{2}{(1 + \frac{M}{L})}$

然后，由于 $\frac{M}{L} \in [0,1)$ 时，$\log_{2}{(1 + \frac{M}{L})} \approx \frac{M}{L} + \sigma$ 即：

$$
\log_{2}{x} \approx E - B + \frac{M}{L} + \sigma \tag{3} \approx \frac{I_x}{L} - B + \sigma % Approximate
$$

那，如果我们跳过直接求 $y = \frac{1}{\sqrt{x}}$，改成求 y 被视为 Int 类型后的近似表示 $I_y$，可以得到

$$
\begin{align*}
y = \frac{1}{\sqrt{x}}\\
\log_{2}{y} = -\frac{1}{2}\log_{2}{x}\\
\frac{I_y}{L} - B + \sigma \approx -\frac{1}{2}(\frac{I_x}{L} - B + \sigma)\\
I_y \approx \frac{3}{2}L(B - \sigma) - \frac{1}{2}I_x
\end{align*}
$$

对应的代码就是

```
i  = 0x5f3759df - ( i >> 1 );
```

再讨论一下 $\sigma$，上面代码中实际 $\sigma \approx 0.0450466$，如果为 0，易得$\frac{3}{2}L(B - \sigma) = 0x5f400000$，$\sigma$ 的取值关乎 float 被视为 int 后 $I_x$ 与 $L(B + \log_{2}{x} - \sigma)$ 实际的接近程度，如果基于数学计算， $\sigma$ 理论最优值为[^1]

![Log by aliasing to int](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2f/Log_by_aliasing_to_int.svg/440px-Log_by_aliasing_to_int.svg.png)

$$
\sigma = \frac{1}{2} − \frac{1 +\ln(\ln(2))}{2\ln(2)} \approx 0.0430357
$$

[^1]:  这个公式咋来的大概率当时没搞明白，过了快一年了再看还是卡在这里没看明白。（2024.10.06）

### Newton's method

牛顿法可以在有根的近似解的情况下提高精度。要求要有第一个近似值，且附近连续可导。

求 $y = \frac{1}{\sqrt{x}}$ 可以视为求 $f(y) = \frac{1}{y^2} - x$ 的根。用上面方法找到第一个近似值，对于近似值 $y_n$ 总能找到更好的近似值 $y_{n + 1} = y_n - \frac{f(y_n)}{f'(y_n)}$，即

$$
y_{n + 1} = y_n(\frac{3 - xy^2_n)}{2})
$$
对应的就是

```
y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
```

### Additional

了解算法即可，硬件上的优化以至于它现在没什么用了。
