---

title: 数位 DP
published: 2026-01-27
tags: [Algorithm]
category: ACM
draft: false
---

# 数位 DP

数位 DP 是一种用于解决**与数字的每一位相关**的计数或判定问题的动态规划技巧。它常用于在给定区间（如 $[L, R]$）内，统计满足某些“数字结构”性质的整数个数。

### T1: [烦人的数学作业 - 洛谷](https://www.luogu.com.cn/problem/P4999)

题目要求从 L 到 R 的所有数字的 数字和 的总和。

运用前缀和的思想，将 L~R 转化为 (1~R) - (1~L)，将原问题拆分为 1~L 和 1~R 两个独立的问题。

我们来看 1~2137 这个例子。

![img0](../../assets/images/digit_dp/digit_dp_0.png)

首先我们对它进行了拆分，这个过程其实就是从高位向低位进行了递归。我们根据上面这个图的思路写一个暴力：

```cpp
LL dfs(int dep, int sum, bool lim)
{
    if (!dep)
        return sum;
    LL res = 0, up = (lim ? a[dep] : 9);
    for (int i = 0; i <= up; i++)
    {
        res += dfs(dep - 1, sum + i, lim && (i == up));
    }
    return res;
}
LL Calc(int k)
{
    int len = 0;
    while (k)
    {
        a[++len] = k % 10;
        k /= 10;
    }
    return dfs(len, 0, 1) % mod;
}
```

其中，dep 表示当前枚举的位数（最高位是 len，最低为是 1）；

sum 表示前面枚举过位置的数字和；

lim 表示当前数位是否有上限，为 1 对于上图中每个括号的最后一行，例如只能枚举到 0~2 而非 0~9。

---

这个暴力的复杂度是 $O(n)$ 的，跟枚举一样慢。我们来考虑怎么优化。

![img2](../../assets/images/digit_dp/digit_dp_2.png)

注意观察我们枚举的逻辑：在从高位向地位枚举的过程中，**对后继答案有影响的只有 当前的位置 和 当前的数字和**，与其它因素无关。

例如 2000~2099，我们在求答案的时候只关注：在前面枚举的两位上，数字和为 2。至于这个 2 到底是 2+0 还是 1+1，我们并不关心。因此，我们可以根绝这一点进行记忆化。

记 $f[dep][sum]$ 表示枚举到第 dep 位时，前面所有数字和为 sum 时的答案。

根据这个定义：

$f[4][1]$ 代表 1000~1999 这个范围所有数的数字和。

$f[3][2]$ 代表 2000~2099 这个范围所有数的数字和，同样可以代表 1100~1199 这个范围，但这里我们用不到，也不关心。

对于一组 dep, sum，我们在求解时如果第一次遇到，那么就通过暴力先求解；后面再遇到这组 dep, sum 时，就可以通过记忆化直接得到答案。这样，我们对于每一组 dep, sum 至多只会求解一次。

时间复杂度 $O(digit\times Maxsum)$， digit 代表位数（18），Maxsum 代表可以达到的最大数字和（9*18）。

#### Code

```cpp
LL a[20], f[20][200];
LL dp(int dep, LL sum, bool lim)
{
    if (!dep)
        return sum;
    if (!lim && f[dep][sum] != -1)
        return f[dep][sum] % mod;
    LL up = lim ? a[dep] : 9, res = 0;
    for (int i = 0; i <= up; i++)
    {
        res = (res + dp(dep - 1, (sum + i) % mod, lim && (i == up))) % mod;
    }
    if (!lim)
        f[dep][sum] = res % mod;
    return res % mod;
}
LL Calc(LL k)
{
    int len = 0;
    while (k)
    {
        a[++len] = k % 10;
        k /= 10;
    }
    return dp(len, 0, 1);
}
int main()
{
    memset(f, -1, sizeof f);
    int T = read();
    while (T--)
    {
        LL l, r;
        l = read(), r = read();
        cout << (Calc(r) - Calc(l - 1) + mod) % mod << '\n';
    }
    return 0;
}
```
> 有些人实现的时候喜欢把 lim 也放在状态里面（定义为 f[dep\][sum\][lim\]），我觉得没必要。因为对于一个 dep，它在 lim=1 （即达到数字上限）的时候，sum 是唯一的，本来就只会求一次，根本不会减少计算量。
---

数位 dp 有很多种写法，但记忆化搜索的写法最为通用且思考量低。

别的我不会。

做数位 dp 题的核心思路就是递归从高位向低位枚举，然后看到底有什么东西会影响答案，比如这道题就是前面位数的数字和。
