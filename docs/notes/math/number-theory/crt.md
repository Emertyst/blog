# 线性同余方程组

## 线性同余方程组的定义

$$
\begin{cases}
    x &\equiv r_1 \pmod {m_1} \\
    x &\equiv r_2 \pmod {m_2} \\
    &\vdots                   \\
    x &\equiv r_n \pmod {m_n} \\
\end{cases}
$$

从这一系列线性同余方程中可以解出唯一的 $x \equiv R \pmod M$，其中 $M = \operatorname{lcm}(m_1, m_2, \cdots, m_n)$（也有可能无解）。若 $m_1, m_2, \cdots, m_n$ 两两互质，则必定有解，且可以用公式求出。若不两两互质，则不一定有解。

## 中国剩余定理

中国剩余定理（CRT）可以解决模数两两互质的情况。

### 过程

1. 求出 $M = \displaystyle\prod_{i = 1}^n m_i$；
2. 对于每个 $i$，求出 $m'_i = \dfrac{M}{m_i}$，以及其在模 $m_i$ 意义下的逆元 $n_i$；
3. 解为 $R \equiv \displaystyle\sum_{i = 1}^n r_i m'_i n_i \pmod {M}$。

时间复杂度 $n \log W$。

### 证明

对于任意的 $i$，由于 $m_i \mid M$，所以 $x \equiv R \pmod {m_i}$，我们拆开 $R$ 的求和号：

$$
x \equiv R \equiv \sum_{j = 1}^n r_j m'_j n_j \equiv r_i m'_i n_i + \sum_{j = 1, j \ne i}^n r_j m'_j n_j \pmod {m_i}
$$

- 对于 $j \ne i$，由于 $m'_j$ 的连乘积中有 $m_i$ 这一项，所以 $r_j m'_j n_j \equiv 0 \pmod {m_i}$；
- 对于第 $i$ 项，由于 $n_i \equiv (m'_j)^{-1} \pmod {m_i}$，所以 $m'_i n_i \equiv 1 \pmod {m_i}$。因此最后只剩下了 $r_i$，也就是第 $i$ 项方程。

### 代码

由于 $m_i$ 不一定是质数，所以求模 $m_i$ 意义下的逆元需要使用 exgcd。

!!! note "代码"

    ```cpp
    for (int i = 1; i <= n; ++i)
        M *= m[i];
    for (int i = 1; i <= n; ++i)
        ans = inc(ans, mul(r[i], mul(M / m[i], inv(M / m[i] % m[i], m[i]), M), M), M);
    ```

## 扩展中国剩余定理

对于模数不两两互质的情况，需要使用 exCRT

### 过程

exCRT 通过反复将两个方程合并为一个求出答案。假设两个方程为：

$$
\begin{cases}
    x \equiv r_1 \pmod {m_1} \\
    x \equiv r_2 \pmod {m_2} \\
\end{cases}
$$

令 $M = \operatorname{lcm}(m_1, m_2)$，我们可以将合并之后的方程写为：

$$
x \equiv k_1 m_1 + r_1 \equiv k_2 m_2 + r_2 \pmod M \\
$$

令 $k_1 m_1 + r_1$ 和 $k_2 m_2 + r_2$ 都是 $[0, M)$ 中的数，有

$$
\begin{aligned}
    k_1 m_1 + r_1 &= k_2 m_2 + r_2 \\
    k_1 m_1 - k_2 m_2 &= r_2 - r_1 \\
\end{aligned}
$$

令 $d = \gcd(m_1, m_2)$。根据裴蜀定理，如果 $r_2 - r_1$ 不能被 $d$ 整除，则无解。反之，则能通过 exgcd 构造出一组 $a m_1 + b m_2 = d$，那么：

$$
\begin{aligned}
    k_1 &= \frac{r_2 - r_1}{d} a \\
    k_2 &= \frac{r_1 - r_2}{d} b \\
\end{aligned}
$$

取 $x = k_1 m_1 + r_1$ 或 $x = k_2 m_2 + r_2$ 即可得到合并后的方程。

在实现时，可以将上面的每一步的结果都对 $M$ 取模。

### 代码

!!! note "代码"

    ```cpp
    pii excrt(pii a, pii b) {
        auto [m1, r1] = a;
        auto [m2, r2] = b;
        if (m1 == -1 || m2 == -1)
            return pii(-1, -1);
        int d = gcd(m1, m2), M = lcm(m1, m2);
        if ((r2 - r1) % d)
            return pii(-1, -1);
        int x, y;
        exgcd(m1, m2, x, y, M);
        return pii(M, inc(r1, mul(m1, mul((r2 - r1) / d, x, M), M), M));
    }
    ```