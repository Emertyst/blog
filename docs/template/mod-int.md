## 食用方法

### 构造函数

1. `ModInt<mod> a;`：构造一个模 `mod` 意义下，值为 $0$ 的数。
2. `ModInt<mod> a(val);`：构造一个模 `mod` 意义下，值为 `val` 的数。

其中，`mod` 为模数，`val` 为要分配的值。

### IO

可以使用 `istream` 和 `ostream` 进行输入输出。

### NaN（Not a Number）

`xxx.nan()`：用于检查一个数是不是 NaN，如果是 NaN 则返回 `true`，否则返回 `false`。

一个为 NaN 的数在输出时，将会输出 `NaN`。如果一次运算中有任何一个数为 NaN，结果也将为 NaN。

### 四则运算

与 `int` 用法相同。在除法时，如果除数在模 `mod` 意义下无逆元，那么会返回 NaN。

### 逆元

`inv(a);`：返回 `a` 在模 `mod` 意义下的逆元，若无逆元则返回 NaN。

### 乘方

`power(a, b)`：返回 `a` 在模 `mod` 意义下的 `b` 次方。

## 代码

!!! notes "模意义下 int 类的封装"

    ```cpp
    #include <iostream>

    template <const int MOD>
    class ModInt {
    private:
        typedef long long ll;
        typedef ModInt<MOD> Type;
        int val;

        static Type getNan() {
            Type ans;
            ans.val = -1;
            return ans;
        }
        static int exgcd(int a, int b, int &x, int &y) {
            if (b == 0)
                return x = 1, y = 0, a;
            int ans = exgcd(b, a % b, y, x);
            return y -= a / b * x, ans;
        }

    public:
        ModInt() { val = 0; }
        template <class T>
        ModInt(T x) { val = (x % MOD + MOD) % MOD; }

        bool nan() const { return val == -1; }

        friend std::istream &operator>>(std::istream &input, Type &x) {
            int tmp;
            input >> tmp;
            x = Type(tmp);
            return input;
        }
        friend std::ostream &operator<<(std::ostream &output, const Type &x) {
            if (x.val == -1)
                output << "NaN";
            else
                output << x.val;
            return output;
        }

        Type &operator+=(const Type &a) {
            if (nan() || a.nan())
                *this = getNan();
            else
                val = val + a.val >= MOD ? val + a.val - MOD : val + a.val;
            return *this;
        }
        friend Type operator+(Type a, const Type &b) { return a += b; }

        Type &operator-=(const Type &a) {
            if (nan() || a.nan())
                *this = getNan();
            else
                val = val < a.val ? val - a.val + MOD : val - a.val;
            return *this;
        }
        friend Type operator-(Type a, const Type &b) { return a -= b; }

        Type &operator*=(const Type &a) {
            if (nan() || a.nan())
                *this = getNan();
            else
                val = 1ll * val * a.val % MOD;
            return *this;
        }
        friend Type operator*(Type a, const Type &b) { return a *= b; }

        friend Type power(Type a, int b) {
            if (a.nan())
                return a;
            Type ans = 1;
            for (; b; b >>= 1, a *= a)
                if (b & 1)
                    ans *= a;
            return ans;
        }
        friend Type inv(const Type &a) {
            if (a.nan())
                return a;
            int x, y, gcd = Type::exgcd(a.val, MOD, x, y);
            Type ans;
            if (gcd != 1)
                ans = Type::getNan();
            else
                ans = Type(x);
            return ans;
        }

        Type &operator/=(const Type &a) {
            if (nan() || a.nan())
                *this = getNan();
            else
                *this *= inv(a);
            return *this;
        }
        friend Type operator/(Type a, const Type &b) { return a /= b; }
    };
    ```