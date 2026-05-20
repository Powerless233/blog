---
title: Solution of CF2225
published: 2026-05-06
tags: [Solution, CF]
category: ACM
draft: false
---

# Educational Codeforces Round 189 (Rated for Div. 2)

这场的感觉就是前 4 题很水，然后开到 E 是个计算几何就力竭了。

## A. A Number Between Two Others

记 y = kx (k > 1)。

则 (x, y) 等效于 (1, k)。

问题等效问是否存在 z 使得 gcd(z, k) = 1。

显然 gcd(k, k - 1) = 1，仅当 k = 2 时不成立。

## B. Alternating String

题目给的操作至多只能使区间两端的位置从相同变为不同。

如果存在 3 个及以上的位置相邻的两个数相等，那一定无解。

如果只有 1 个位置，设 a[x] = a[x + 1]，那么只需要翻转 1 - x 即可，如果区间长度是奇数（即翻转无效），那么进行操作 3（值翻转）即可。

如果只有 2 个位置，分情况讨论：

- 形如 ...abaa ... aaba...，中间的长度一定是奇数，翻转无效，进行操作 3 即可。
- 形如 ...abaa ... bbab...，中间的长度一定是偶数，翻转即可。

综上，如果只有 1 或 2 个位置相邻的两个数相等，则一定有解。

## C. Red-Black Pairs

一眼 dp。

记 string a, b。令 f[i] 表示使前 i 列格子符合要求的最小操作数。

转移就两种：$$A \\ A$$  或者 $$ AA \\ BB$$ 。

f[i] = min(f[i - 1] +  (a[i] != b[i]), f[i - 2] + (a[i] != a[i - 1]) + (b[i] != b[i - 1]))

## D. Exceptional Segments

观察样例发现两个性质：

- 区间可以叠加：若 (x, y), (y+1, z) 符合条件，则 (x, z) 也符合条件。
- 最小区间仅有 2 种情况：4k, 4k + 1, 4k + 2, 4k + 3 或 4k + 2, 4k + 3, 4k + 4, 4k + 5

对于每一种情况，将 x 左侧和右侧分别有几个区间求出来然后相乘即可。

#### Code

```cpp
void solve()
{
    LL n, x;
    n = read(), x = read();

    auto calc = [&](LL k, LL x, LL n) -> LL
    {
        LL l = x, r = x;
        if (l < k)
            return 0;
        while (l % 4 != k)
            l--;
        while (r % 4 != (k + 3) % 4)
            r++;
        if (r > n)
            return 0;
        l = l / 4 + 1;
        r = (n - r) / 4 + 1;
        l %= mod;
        r %= mod;
        return l * r % mod;
    };

    LL ans = calc(0, x, n) + calc(2, x, n);
    ans %= mod;
    cout << ans << '\n';
}
```

