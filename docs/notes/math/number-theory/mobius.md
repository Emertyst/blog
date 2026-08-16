# 莫比乌斯反演

## 狄利克雷卷积

定义两个数论函数 $f(x), g(x)$ 的狄利克雷卷积（记作 $f * g$）：

$$
(f * g)(x) = \sum_{i \mid x} f(i) g\left( \frac{x}{i} \right) \\
$$

狄利克雷卷积满足交换律、结合律、分配律，并且满足 $f * \epsilon = f$。

!!! tip

    $$
    \epsilon(x) =
    \begin{cases}
        1, x = 1 \\
        0, x \ne 1 \\
    \end{cases}
    $$

## 莫比乌斯反演

### 定义

!!! abstract "莫比乌斯反演"

    $$
    f = g * 1 \iff g = \mu * f \\
    $$

### 证明

先证明一个结论：

!!! abstract "结论"

    $$
    \mu * 1 = \epsilon \\
    $$

!!! info "证明"

    设 $n$ 有 $k$ 个质因子，则 $n = 1$ 时（即 $k = 0$ 时），$(\mu * 1)(n) = \epsilon(n)$；$n \ne 1$ 时（即 $k \ne 0$ 时），有：

    $$
    \begin{aligned}
        (\mu * 1)(n) &= \sum_{i = 0}^k (-1)^i \binom{k}{i} \\
        &= (1 - 1)^k \\
        &= 0 \\
        &= \epsilon(n) \\
    \end{aligned}
    $$

    !!! warning

        因为二项式定理不适用于指数为 $0$ 的情况，因此需要分类讨论。

接下来证明莫比乌斯反演

!!! info "证明"

    $$
    \begin{aligned}
        f &= g * 1 \\
        f * \mu &= g * 1 * \mu \\
        f * \mu &= g * \epsilon \\
        f * \mu &= g \\
    \end{aligned}
    $$

    因为变换均等价，因此得证。

### 结论

!!! abstract "常用结论"

    $$
    \varphi * 1 = Id \iff \varphi = \mu * Id \\
    $$

!!! info "证明"

    先证明左半部分：

    $$
    \begin{aligned}
        Id(x) &= x \\
        &= \sum_{i \mid x} \sum_{j = 1}^x [\gcd(j, x) = i] \\
        &= \sum_{i \mid x} \sum_{j = 1}^{\frac{x}{i}} \left[\gcd\left(j, \frac{x}{i} \right) = 1 \right] \\
        &= \sum_{i \mid x} \varphi\left(\frac{x}{i} \right) \\
        &= \sum_{i \mid x} \varphi(i) \\
        &= (\varphi * 1)(x) \\
    \end{aligned}
    $$

    其中，第二行变换的依据是：$1 \sim x$ 中任何一个数与 $x$ 的 $\gcd$ 一定是 $x$ 的因数，所以统计 $1 \sim x$ 中所有与 $x$ 的 $\gcd$ 为 $x$ 因数的数的数量一定是 $x$。

    右半部分可由莫比乌斯反演推出。