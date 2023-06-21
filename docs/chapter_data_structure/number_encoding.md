# 数字编码 *

!!! note

    在本书中，标题带有的 * 符号的是选读章节。如果你时间有限或感到理解困难，建议先跳过，等学完必读章节后再单独攻克。

## 原码、反码和补码

从上一节的表格中我们发现，所有整数类型能够表示的负数都比正数多一个。例如，`byte` 的取值范围是 $[-128, 127]$ 。这个现象比较反直觉，它的内在原因涉及到原码、反码、补码的相关知识。在展开分析之前，我们首先给出三者的定义：

- **原码**：我们将数字的二进制表示的最高位视为符号位，其中 $0$ 表示正数，$1$ 表示负数，其余位表示数字的值。
- **反码**：正数的反码与其原码相同，负数的反码是对其原码除符号位外的所有位取反。
- **补码**：正数的补码与其原码相同，负数的补码是在其反码的基础上加 $1$ 。

![原码、反码与补码之间的相互转换](number_encoding.assets/1s_2s_complement.png)

显然，「原码」最为直观，**然而数字却是以「补码」的形式存储在计算机中的**。这是因为原码存在一些局限性。

一方面，**负数的原码不能直接用于运算**。例如，我们在原码下计算 $1 + (-2)$ ，得到的结果是 $-3$ ，这显然是不对的。

$$
\begin{aligned}
& 1 + (-2) \newline
& = 0000 \space 0001 + 1000 \space 0010 \newline
& = 1000 \space 0011 \newline
& = -3
\end{aligned}
$$

为了解决此问题，计算机引入了「反码」。例如，我们先将原码转换为反码，并在反码下计算 $1 + (-2)$ ，并将结果从反码转化回原码，则可得到正确结果 $-1$ 。

$$
\begin{aligned}
& 1 + (-2) \newline
& = 0000 \space 0001 \space \text{(原码)} + 1000 \space 0010 \space \text{(原码)} \newline
& = 0000 \space 0001 \space \text{(反码)} + 1111  \space 1101 \space \text{(反码)} \newline
& = 1111 \space 1110 \space \text{(反码)} \newline
& = 1000 \space 0001 \space \text{(原码)} \newline
& = -1
\end{aligned}
$$

另一方面，**数字零的原码有 $+0$ 和 $-0$ 两种表示方式**。这意味着数字零对应着两个不同的二进制编码，而这可能会带来歧义问题。例如，在条件判断中，如果没有区分正零和负零，可能会导致错误的判断结果。如果我们想要处理正零和负零歧义，则需要引入额外的判断操作，其可能会降低计算机的运算效率。

$$
\begin{aligned}
+0 & = 0000 \space 0000 \newline
-0 & = 1000 \space 0000
\end{aligned}
$$

与原码一样，反码也存在正负零歧义问题。为此，计算机进一步引入了「补码」。那么，补码有什么作用呢？我们先来分析一下负零的补码的计算过程：

$$
\begin{aligned}
-0 = \space & 1000 \space 0000 \space \text{(原码)} \newline
= \space & 1111 \space 1111 \space \text{(反码)} \newline
= 1 \space & 0000 \space 0000 \space \text{(补码)} \newline
\end{aligned}
$$

在负零的反码基础上加 $1$ 会产生进位，而由于 byte 的长度只有 8 位，因此溢出到第 9 位的 $1$ 会被舍弃。**从而得到负零的补码为 $0000 \space 0000$ ，与正零的补码相同**。这意味着在补码表示中只存在一个零，从而解决了正负零歧义问题。

还剩余最后一个疑惑：byte 的取值范围是 $[-128, 127]$ ，多出来的一个负数 $-128$ 是如何得到的呢？我们注意到，区间 $[-127, +127]$ 内的所有整数都有对应的原码、反码和补码，并且原码和补码之间是可以互相转换的。

然而，**补码 $1000 \space 0000$ 是一个例外，它并没有对应的原码**。根据转换方法，我们得到该补码的原码为 $0000 \space 0000$ 。这显然是矛盾的，因为该原码表示数字 $0$ ，它的补码应该是自身。计算机规定这个特殊的补码 $1000 \space 0000$ 代表 $-128$ 。实际上，$(-1) + (-127)$ 在补码下的计算结果就是 $-128$ 。

$$
\begin{aligned}
& (-127) + (-1) \newline
& = 1111 \space 1111 \space \text{(原码)} + 1000 \space 0001 \space \text{(原码)} \newline
& = 1000 \space 0000 \space \text{(反码)} + 1111  \space 1110 \space \text{(反码)} \newline
& = 1000 \space 0001 \space \text{(补码)} + 1111  \space 1111 \space \text{(补码)} \newline
& = 1000 \space 0000 \space \text{(补码)} \newline
& = -128
\end{aligned}
$$

你可能已经发现，上述的所有计算都是加法运算。这暗示着一个重要事实：**计算机内部的硬件电路主要是基于加法运算设计的**。这是因为加法运算相对于其他运算（比如乘法、除法和减法）来说，硬件实现起来更简单，更容易进行并行化处理，从而提高运算速度。

然而，这并不意味着计算机只能做加法。**通过将加法与一些基本逻辑运算结合，计算机能够实现各种其他的数学运算**。例如，计算减法 $a - b$ 可以转换为计算加法 $a + (-b)$ ；计算乘法和除法可以转换为计算多次加法或减法。

现在，我们可以总结出计算机使用补码的原因：基于补码表示，计算机可以用同样的电路和操作来处理正数和负数的加法，不需要设计特殊的硬件电路来处理减法，并且无需特别处理正负零的歧义问题。这大大简化了硬件设计，并提高了运算效率。

补码的设计非常精妙，由于篇幅关系我们先介绍到这里。建议有兴趣的读者进一步深度了解。

## 浮点数编码

细心的你可能会发现：`int` 和 `float` 长度相同，都是 4 bytes，但为什么 `float` 的取值范围远大于 `int` ？这非常反直觉，因为按理说 `float` 需要表示小数，取值范围应该变小才对。

实际上，这是因为浮点数 `float` 采用了不同的表示方式。根据 IEEE 754 标准，32-bit 长度的 `float` 由以下部分构成：

- 符号位 $\mathrm{S}$ ：占 1 bit ；
- 指数位 $\mathrm{E}$ ：占 8 bits ；
- 分数位 $\mathrm{N}$ ：占 24 bits ，其中 23 位显式存储；

设 32-bit 二进制数的第 $i$ 位为 $b_i$ ，则 `float` 值的计算方法定义为：

$$
\text { val } = (-1)^{b_{31}} \times 2^{\left(b_{30} b_{29} \ldots b_{23}\right)_2-127} \times\left(1 . b_{22} b_{21} \ldots b_0\right)_2
$$

转化到十进制下的计算公式为

$$
\text { val }=(-1)^{\mathrm{S}} \times 2^{\mathrm{E} -127} \times (1 + \mathrm{N})
$$

其中各项的取值范围为

$$
\begin{aligned}
\mathrm{S} \in & \{ 0, 1\} , \quad \mathrm{E} \in \{ 1, 2, \dots, 254 \} \newline
(1 + \mathrm{N}) = & (1 + \sum_{i=1}^{23} b_{23-i} 2^{-i}) \subset [1, 2 - 2^{-23}]
\end{aligned}
$$

![IEEE 754 标准下的 float 表示方式](number_encoding.assets/ieee_754_float.png)

以上图为例，$\mathrm{S} = 0$ ， $\mathrm{E} = 124$ ，$\mathrm{N} = 2^{-2} + 2^{-3} = 0.375$ ，易得

$$
\text { val } = (-1)^0 \times 2^{124 - 127} \times (1 + 0.375) = 0.171875
$$

现在我们可以回答最初的问题：**`float` 的表示方式包含指数位，导致其取值范围远大于 `int`** 。根据以上计算，`float` 可表示的最大正数为 $2^{254 - 127} \times (2 - 2^{-23}) \approx 3.4 \times 10^{38}$ ，切换符号位便可得到最小负数。

**尽管浮点数 `float` 扩展了取值范围，但其副作用是牺牲了精度**。整数类型 `int` 将全部 32 位用于表示数字，数字是均匀分布的；而由于指数位的存在，浮点数 `float` 的数值越大，相邻两个数字之间的差值就会趋向越大。

进一步地，指数位 $E = 0$ 和 $E = 255$ 具有特殊含义，**用于表示零、无穷大、$\mathrm{NaN}$ 等**。

<div class="center-table" markdown>

| 指数位 E           | 分数位 $\mathrm{N} = 0$ | 分数位 $\mathrm{N} \ne 0$ | 计算公式                                                               |
| ------------------ | ----------------------- | ------------------------- | ---------------------------------------------------------------------- |
| $0$                | $\pm 0$                 | 次正规数                  | $(-1)^{\mathrm{S}} \times 2^{-126} \times (0.\mathrm{N})$              |
| $1, 2, \dots, 254$ | 正规数                  | 正规数                    | $(-1)^{\mathrm{S}} \times 2^{(\mathrm{E} -127)} \times (1.\mathrm{N})$ |
| $255$              | $\pm \infty$            | $\mathrm{NaN}$            |                                                                        |

</div>

特别地，次正规数显著提升了浮点数的精度，这是因为：

- 最小正正规数为 $2^{-126} \approx 1.18 \times 10^{-38}$ ；
- 最小正次正规数为 $2^{-126} \times 2^{-23} \approx 1.4 \times 10^{-45}$ ；

双精度 `double` 也采用类似 `float` 的表示方法，此处不再详述。