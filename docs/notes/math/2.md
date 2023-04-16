# Miller Rabin

## 朴素 ~~暴力~~ 算法

如果我们要判断 $n$ 是否是素数，可以枚举 $[2, \sqrt n]$ 的每一个数判断能否整除 $n$，如果存在 $x$ 使 $x \mid n$，则 $n$ 不是素数，否则是素数。

## 费马素性测试

!!! abstract "费马小定理"

    若 $p$ 为质数，则：

    $$
    i^{p - 1} \equiv 1 \pmod p, \gcd(i, p) = 1 \\
    $$

或许可以通过这个性质来判断一个数是否是质数。我们可以任选一个底数 $b$，判断 $b^n \equiv 1 \pmod n$ 是否成立。但是有一些合数是可以通过这个测试的，比如 $2^{340} \equiv 1 \pmod{341}$。于是乎我们可以多选几个底数判断。但存在一类合数，对于任意 $i \in [1, n-1], \gcd(i, n) = 1$ 都满足上述关系，这类合数叫卡迈克尔数。比如 $561 = 3 \times 11 \times 17$ 就是一个卡迈克尔数。

如果随机选中的底数是一个卡迈克尔数的因子，卡迈克尔数也不满足上面的性质，但这样的概率还是太低了。也就是说，费马素性测试会有一定的概率误判。

## 二次探测定理

!!! abstract "二次探测定理"

    若 $p$ 为奇素数，则 $x^2 \equiv 1 \pmod p$ 的解为 $x \equiv 1 \pmod p$ 或 $x \equiv p - 1 \pmod p$。

!!! info "证明"

    由平方差公式：

    $$
    (x - 1)(x + 1) \equiv 0 \pmod p \\
    $$

    也就是说 $(x - 1)(x + 1)$ 为 $p$ 的倍数，由于 $p$ 是一个素数，$x - 1$ 和 $x + 1$ 中至少有一个是 $p$ 的倍数（如果 $p$ 是合数，则可能 $x - 1$ 和 $x + 1$ 都是 $p$ 的因数的倍数）。

## Miller-Rabin 素性测试

如果将费马素性测试与二次探测定理结合使用，即可提高测试成功的概率。

具体的流程是：

1. 对于每个底数，先进行 Fermat 素性测试（即判断 $b^{n - 1} \equiv 1 \pmod n$）。
2. 令 $d \leftarrow n - 1$，重复将 $d \leftarrow \frac{d}{2}$ 并计算 $b^d \mod n$，直到 $d$ 为奇数。如果循环结束时还没有判断出 $p$ 不是质数，则换下一个底数测试。
3. 如果 $b^d \mod n = n - 1$，直接结束循环，换下一个底数测试；如果 $1 \lt b^d \mod n \lt n - 1$，则 $p$ 不是质数。

如果选 $k$ 个数，那么时间复杂度是 $O(k \log^2 n)$。

如果我们预处理出来 $n - 1 = d \times 2^t, d \mod 2 = 1$，可以计算出 $b^d \mod p$ 并不断平方，如果结果一直是 $1$ 或者出现 $n - 1$，则换下一个底数测试；如果结果在没有出现 $n - 1$ 的情况下就出现了 $1$，说明 $p$ 一定不是质数。这时，时间复杂度优化成 $O(k \log n)$。

### 固定底数与确定性检验

如果广义黎曼猜想成立，我们只需要选取所有 $[2, \min(n - 2, \lfloor 2 \ln^2 n\rfloor)]$ 作为作为底数进行测试，即可**确定** $n$ 是不是质数，时间复杂度 $\log^3 n$。

但对于 $2^{64}$ 内的数，可以选取 $2, 325, 9375, 28178, 450775, 9780504, 1795265022$ 作为底数，这样判断的正确率为 $100 \%$。

```plain
2, 325, 9375, 28178, 450775, 9780504, 1795265022
```
~~便于复制~~

如果记不住，则可以使用前 $12$ 个质数作为底数检验。

这样就可以实现 $O(\log n)$ 的判素了。

## 代码

!!! note "代码"

    ```cpp
    #include <iostream>
    using namespace std;
    typedef long long ll;
    typedef long double ld;
    typedef unsigned long long ull;
    const ll base[7] = {2, 325, 9375, 28178, 450775, 9780504, 1795265022};
    ll mul(ll a, ll b, ll mod) {
        ull c = (ld)a / mod * b + 0.5L, r = (ull)a * b - (ull)c * mod;
        return r < mod ? r : r + mod;
    }
    ll power(ll a, ll b, ll mod) {
        if (!(a %= mod))
            return 1;
        ll ans = 1;
        for (; b; b >>= 1, a = mul(a, a, mod))
            if (b & 1)
                ans = mul(ans, a, mod);
        return ans;
    }
    bool check(ll x) {
        if (x <= 2)
            return x == 2;
        ll b = x - 1, t = 0;
        while ((b & 1) == 0)
            b >>= 1, ++t;
        for (int i = 0; i < 7; ++i) {
            if (power(base[i], x - 1, x) != 1)
                return false;
            for (ll tmp = power(base[i], b, x), nxt; t--; tmp = nxt) {
                nxt = mul(tmp, tmp, x);
                if (nxt != 1)
                    continue;
                else if (tmp == x - 1 || tmp == 1)
                    break;
                else
                    return false;
            }
        }
        return true;
    }
    ll x;
    int main() {
        ios::sync_with_stdio(false), cin.tie(0);
        while (cin >> x)
            cout << "NY"[check(x)] << "\n";
    }
    ```