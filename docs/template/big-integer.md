## 食用方法

所有有关的东西都在 `Integer` 命名空间下，我们要用的是 `BigInteger` 类。

### 构造函数

1. `BigInteger a;`：构造一个值为 $0$ 的数。
2. `BigInteger a(val);`：构造一个值为 `val` 的数。
3. `BigInteger a(str)`：构造一个值为字符串开头的数。

其中，`val` 为要分配的值，`str` 为字符串（可以是 `char*` 或 `string`）。使用字符串构造高精度数时，会读取**开头**最长的可以成为数的子串（由若干数字构成，开头可以有负号，但只能有一个负号），并忽略之后的字符，若开头并不是数字，则赋值为 $0$。下面是一些使用字符串构造的高精度数的示例：

|`str`|构造出高精度数的值|
|:-:|:-:|
|`114514`|$114514$|
|`-114514`|$-114514$|
|`--114514`|$0$|
|`114514?`|$114514$|
|`?114514`|$0$|
|`114514 1919810`|$114514$|

### IO

可以使用 `istream` 和 `ostream` 进行输入输出。在输入时，会**忽略若干字符**直到找到一个可以成为数的子串，且不会读取这个子串后面的字符。下面是一些示例：

|输入流中的字符串|构造出高精度数的值|输入之后输入流中的字符串|
|:-:|:-:|:-:|
|`114514`|$114514$|无|
|`-114514`|$-114514$|无|
|`--114514`|$-114514$|无|
|`?114514`|$114514$|无|
|`?-114514?`|$-114514$|`?`|

### 异常

#### 数值过大

由于 NTT 的精度限制，本实现只能支持 $6 \times 10^6$ 长度的乘法（即结果的长度最多 $6 \times 10^6$）。如果结果的长度超过了最大长度，则抛出 `OutOfRange` 异常。此外，由于除法过程中用到了一些可能比除数和被除数还大的临时数据，因此除法的最大长度比乘法低（大概是除数的长度不超过 $3 \times 10^6$，但这个估计可能有误差）。

#### 除 0

在除法或取模时，如果除数或模数为 $0$，则抛出 `DivideByZero` 异常。

### 四则运算、比较运算

同 `int`。若两个高精度数分别为 $n$ 和 $m$ 位（对于除法，则 $n$ 为被除数，$m$ 为除数），则时间复杂度为：

|运算|时间复杂度|
|:-:|:-:|
|比较|$O(\min\{n, m\})$|
|加减|$O(\max\{n, m\})$|
|乘|$O((n + m) \log (n + m))$|
|除、取模|设 $k = \max\{n + 2m, 4(n - m)\}$，则复杂度为 $O(k \log k)$（常数极大，约为乘法的 $8$ 倍）|

取模时（以 `a % b` 为例），结果的符号与 $a$ 的符号相同，结果的绝对值为 $|a| \bmod |b|$。

### 乘方

如果要将一个高精度数 $a$ 乘 $k$ 次方（$k$ 为 `int`），可以写成 `a ^ k`，复杂度为 $O(nk \log nk)$。

## 代码

!!! notes "高精度整数"

    ```cpp
    #include <algorithm>
    #include <cstring>
    #include <iostream>
    #include <string>
    #include <tuple>
    #include <vector>

    namespace Integer {
    typedef long long ll;
    typedef unsigned long long ull;
    typedef __int128 lll;
    typedef long double ld;

    namespace Math {
    constexpr int G = 3, MOD[2] = {998244353, 1004535809};
    constexpr ll M = 1002772198720536577ll, N[2] = {334257240187163831ll, 668514958533372747ll};
    template <int MOD>
    int inc(int a, int b) { return a + b >= MOD ? a + b - MOD : a + b; }
    template <int MOD>
    int dec(int a, int b) { return a < b ? a - b + MOD : a - b; }
    template <int MOD>
    int mul(int a, int b) { return 1ll * a * b % MOD; }
    template <int MOD>
    int power(int a, int b) {
        int ans = 1;
        for (; b; b >>= 1, a = mul<MOD>(a, a))
            if (b & 1)
                ans = mul<MOD>(ans, a);
        return ans;
    }
    template <int MOD>
    int inv(int a) { return power<MOD>(a, MOD - 2); }
    ll mul(ll a, ll b) {
        ull c = (ld)a / M * b + 0.5L, ans = 1ull * a * b - c * M;
        return ans < 1ull * M ? ans : ans + M;
    }
    } // namespace Math
    using namespace Math;
    class BigInteger : std::vector<int> {
    private:
        const static int LEN = 1e6, DIG = 6, MAX = 1e6, LG = 32 - __builtin_clz(LEN), N = 1 << LG;
        const static bool NEG = false, POS = true;

        bool tag;

        class OutOfRange : public std::exception {
            const char *what() const throw() { return "Out of range!"; }
        };
        class DivideByZero : public std::exception {
            const char *what() const throw() { return "Divide by 0!"; }
        };

        class Rev {
        private:
            int rev[N];

        public:
            Rev() {
                for (int i = 1; i < N; ++i)
                    rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (LG - 1));
            }
            const int &operator[](int a) const { return rev[a]; }
        };
        template <int I>
        class W {
        private:
            int w[N];

        public:
            W() {
                int u = power<MOD[I]>(G, (MOD[I] - 1) >> LG);
                w[N >> 1] = 1;
                for (int i = (N >> 1) + 1; i < N; ++i)
                    w[i] = Math::mul<MOD[I]>(w[i - 1], u);
                for (int i = (N >> 1) - 1; i; --i)
                    w[i] = w[i << 1];
            }
            const int &operator[](int a) const { return w[a]; }
        };
        static inline const Rev rev;
        static inline const std::tuple<W<0>, W<1>> w;

        void skip() {
            int n = (int)size() - 1;
            while (n && !at(n))
                --n;
            resize(n + 1);
        }
        void add(const BigInteger &a) {
            resize(std::max(size(), a.size()));
            for (int i = 0; i < (int)a.size(); ++i)
                at(i) += a[i];
            for (auto itr = begin(); itr != end() - 1; ++itr)
                if (*itr >= MAX)
                    ++*(itr + 1), *itr -= MAX;
            if (*(end() - 1) >= MAX)
                *(end() - 1) -= MAX, push_back(1);
        }
        void subtract(const BigInteger &a) {
            resize(std::max(size(), a.size()));
            bool nice = false;
            for (int i = 0; i < (int)a.size(); ++i)
                at(i) -= a[i], nice = at(i) < 0;
            if (size() == a.size() && nice) {
                tag = !tag;
                for (int &i : *this)
                    i *= -1;
            }
            for (auto itr = begin(); itr != end() - 1; ++itr)
                if (*itr < 0)
                    --*(itr + 1), *itr += MAX;
            skip();
        }

        static int getLen(int n) { return 32 - __builtin_clz(n); }
        template <int I>
        static void ntt(vector<int> &a, int lg, bool inv) {
            int n = 1 << lg;
            for (int i = 1; i < n; ++i)
                if (i < (rev[i] >> (LG - lg)))
                    std::swap(a[i], a[rev[i] >> (LG - lg)]);
            for (int l = 1; l < n; l <<= 1)
                for (int i = 0; i < n; i += (l << 1))
                    for (int j = i; j < i + l; ++j) {
                        int tmp1 = a[j], tmp2 = mul<MOD[I]>(std::get<I>(w)[l + j - i], a[j + l]);
                        a[j] = inc<MOD[I]>(tmp1, tmp2), a[j + l] = dec<MOD[I]>(tmp1, tmp2);
                    }
            if (inv) {
                std::reverse(a.data() + 1, a.data() + n);
                for (int i = 0, inv = Math::inv<MOD[I]>(n); i < n; ++i)
                    a[i] = mul<MOD[I]>(a[i], inv);
            }
        }

        template <int I>
        static void multiply(const BigInteger &a, const BigInteger &b, vector<ll> &c, int lg) {
            vector<int> x(a.begin(), a.end()), y(b.begin(), b.end());
            x.resize(1 << lg, 0), y.resize(1 << lg, 0);
            ntt<I>(x, lg, false), ntt<I>(y, lg, false);
            for (int i = 0; i < (int)x.size(); ++i)
                x[i] = mul<MOD[I]>(x[i], y[i]);
            ntt<I>(x, lg, true);
            for (int i = 0; i < (int)x.size(); ++i)
                (c[i] += mul(1ll * x[i], Math::N[I])) %= M;
            if constexpr (I)
                multiply<I - 1>(a, b, c, lg);
        }

        BigInteger &operator<<=(int a) { return insert(begin(), a, 0), *this; }
        friend BigInteger operator<<(BigInteger a, int b) { return a <<= b; }
        BigInteger &operator>>=(int a) {
            if (a >= (int)size())
                clear(), push_back(0);
            else
                erase(begin(), begin() + a);
            return *this;
        }
        friend BigInteger operator>>(BigInteger a, int b) { return a >>= b; }
        friend BigInteger inv(const BigInteger &a) {

            int n = (int)a.size();
            if (n <= 2) {
                lll b = 1, c = 0;
                for (auto itr = a.rbegin(); itr != a.rend(); ++itr)
                    b *= MAX, b *= MAX, c *= MAX, c += *itr;
                return BigInteger(b / c);
            }
            int m = (n >> 1) + 1;
            BigInteger ans = inv(a >> (n - m)) << (n - m);
            return ans * ((BigInteger(2) << n * 2) - ans * a) >> n * 2;
        }

    public:
        BigInteger() { tag = POS, push_back(0); }
        template <class T>
        BigInteger(T a) {
            tag = POS;
            if (a == 0) {
                push_back(0);
                return;
            }
            if (a < 0)
                tag = NEG, a = -a;
            while (a)
                push_back(a % MAX), a /= MAX;
        }
        BigInteger(const char *s) {
            tag = POS;
            if (*s == '-')
                tag = NEG, ++s;
            int n = 0;
            while (isdigit(*s))
                ++n, ++s;
            if (!n) {
                tag = POS, push_back(0);
                return;
            }
            resize((n + DIG - 1) / DIG);
            for (int i = 0, j = 1; i < n; ++i) {
                --s, at(i / DIG) += (*s - '0') * j, j *= 10;
                if ((i + 1) % DIG == 0)
                    j = 1;
            }
        }
        BigInteger(const std::string &s) { BigInteger(s.c_str()); }

        friend std::istream &operator>>(std::istream &input, BigInteger &a) {
            char c = 0;
            while (!isdigit(input.peek()))
                c = input.get();
            std::string s;
            s.clear();
            while (isdigit(input.peek()))
                s.push_back(input.get());
            if (c == '-')
                a.tag = NEG;
            else
                a.tag = POS;
            a.clear(), a.resize((s.size() + DIG - 1) / DIG);
            auto itr = s.rbegin();
            for (int i = 0, j = 1; itr != s.rend(); ++itr, ++i) {
                a[i / DIG] += (*itr - '0') * j, j *= 10;
                if ((i + 1) % DIG == 0)
                    j = 1;
            }
            a.skip();
            return input;
        }
        friend std::ostream &operator<<(std::ostream &output, const BigInteger &a) {
            if (a.tag == NEG && !(a.size() == 1 && a[0] == 0))
                output << '-';
            output << *a.rbegin();
            for (auto itr = a.rbegin() + 1; itr != a.rend(); ++itr) {
                int tmp = *itr, num[DIG];
                for (int i = 0; i < DIG; ++i)
                    num[i] = tmp % 10, tmp /= 10;
                for (int i = DIG; i; --i)
                    output << num[i - 1];
            }
            return output;
        }

        friend bool operator==(const BigInteger &a, const BigInteger &b) {
            if (a.size() != b.size())
                return false;
            for (int i = 0; i < (int)a.size(); ++i)
                if (a[i] != b[i])
                    return false;
            return true;
        }
        friend bool operator!=(const BigInteger &a, const BigInteger &b) { return !(a == b); }

        friend bool operator<(const BigInteger &a, const BigInteger &b) {
            if (a.tag != b.tag)
                return a.tag == NEG;
            if (a.size() != b.size())
                return (a.size() < b.size()) ^ (a.tag == NEG);
            for (int i = (int)a.size() - 1; i >= 0; --i)
                if (a[i] > b[i])
                    return a.tag == NEG;
                else if (a[i] < b[i])
                    return a.tag == POS;
            return false;
        }
        friend bool operator>=(const BigInteger &a, const BigInteger &b) { return !(a < b); }

        friend bool operator>(const BigInteger &a, const BigInteger &b) {
            if (a.tag != b.tag)
                return a.tag == POS;
            if (a.size() != b.size())
                return (a.size() < b.size()) ^ (a.tag == POS);
            for (int i = (int)a.size() - 1; i >= 0; --i)
                if (a[i] > b[i])
                    return a.tag == POS;
                else if (a[i] < b[i])
                    return a.tag == NEG;
            return false;
        }
        friend bool operator<=(const BigInteger &a, const BigInteger &b) { return !(a > b); }

        friend BigInteger operator-(BigInteger a) { return a.tag = !a.tag, a; }

        BigInteger &operator+=(const BigInteger &a) { return (tag != a.tag) ? subtract(a) : add(a), *this; }
        friend BigInteger operator+(BigInteger a, const BigInteger &b) { return a += b; }

        BigInteger &operator-=(const BigInteger &a) { return (tag != a.tag) ? add(a) : subtract(a), *this; }
        friend BigInteger operator-(BigInteger a, const BigInteger &b) { return a -= b; }

        BigInteger &operator*=(const BigInteger &a) {
            tag ^= (a.tag == NEG);
            int n = size(), m = a.size(), lg = getLen(n + m);
            if (n + m - 1 > LEN)
                throw OutOfRange();
            vector<ll> tmp(1 << lg, 0);
            multiply<1>(*this, a, tmp, lg);
            resize(n + m);
            for (int i = 0; i < n + m; ++i)
                tmp[i + 1] += tmp[i] / MAX, at(i) = tmp[i] % MAX;
            if ((int)size() > 1 && *(end() - 1) == 0)
                erase(end() - 1);
            return *this;
        }
        friend BigInteger operator*(BigInteger a, const BigInteger &b) { return a *= b; }

        BigInteger &operator/=(BigInteger a) {
            if (a == 0)
                throw DivideByZero();
            bool tmp = (tag ^ a.tag) ? NEG : POS;
            tag = a.tag = POS;
            int n = size(), m = a.size();
            if (n > m * 2)
                *this <<= (n - m * 2), a <<= (n - m * 2), m = n - m, n = m * 2;
            BigInteger b = *this * inv(a) >> m * 2;
            *this -= a * b, *this = *this < a ? b : b + 1;
            tag = tmp;
            return *this;
        }
        friend BigInteger operator/(BigInteger a, const BigInteger &b) { return a /= b; }

        BigInteger &operator%=(BigInteger a) {
            if (a == 0)
                throw DivideByZero();
            bool tmp = tag;
            tag = a.tag = POS;
            int n = size(), m = a.size();
            if (n > m * 2)
                *this <<= (n - m * 2), a <<= (n - m * 2), m = n - m, n = m * 2;
            BigInteger b = *this * inv(a) >> m * 2;
            *this -= a * b;
            if (*this >= a)
                *this -= a;
            tag = tmp;
            return *this;
        }
        friend BigInteger operator%(BigInteger a, const BigInteger &b) { return a %= b; }

        friend BigInteger operator^(BigInteger a, int b) {
            BigInteger ans = 1;
            for (; b; b >>= 1, a *= a)
                if (b & 1)
                    ans *= a;
            return ans;
        }
    };
    } // namespace Integer
    ```