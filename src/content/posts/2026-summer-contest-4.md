---
title: 2026暑期训练赛4
published: 2026-07-17
tags: [Contest]
category: ACM
draft: false
---

# 2026 暑期训练赛 4

Presented By 137QWQ & Powerless。

题目难度简单排序：

| 难度 | 题号       | 涉及算法                     |
| :--- | :--------- | :--------------------------- |
| 入门 | A, G, J, M | 模拟，二分，贪心，字符串     |
| 基础 | C, E, I, K | 数论，构造，并查集，按位运算 |
| 提高 | D, F, H, L | 数论，贪心，树的直径，交互   |
| 困难 | B          | 矩阵快速幂                   |

## Problem A. Lasers

> Quesion By qwq.

### Description

在一个二维坐标平面上，坐标范围为 $$(0,0)$$ 到 $$(x, y)$$。你位于 $$(0,0)$$，目标是到达 $$(x, y)$$。

但是，平面上有 $n$ 条水平激光，第 $i$ 条激光从 $$(0, a_i)$$ 到 $$(x, a_i)$$，还存在 $m$ 条竖直激光，第 $i$ 条激光从 $$(b_i, 0)$$ 到 $$(b_i, y)$$。

你可以沿任意方向移动以到达 $$(x, y)$$，但你的运动轨迹必须是平面内的一条连续曲线。每当你穿越一条水平或一条竖直激光，计为一次穿越。特别地，如果你经过两条激光的交点，则计为两次穿越。

### Analysis

由于所有线都是上下限封死的，所以所有线都一定会被穿过至少一次。

原题等价为 A+B Problem。后继的读入必须做，但是完全无效。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 1e5 + 5;

void solve()
{
    int n, m, x, y;
    cin >> n >> m >> x >> y;
    for (int i = 1; i <= n; i++)
    {
        int t;
        cin >> t;
    }
    for (int i = 1; i <= m; i++)
    {
        int t;
        cin >> t;
    }
    cout << n + m << '\n';
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
    return 0;
}
```

## Problem B. Odd Steps

> Quesion By Powerless.

### Description

求满足以下条件的序列数量（对 998244353 取模）：

- 只由奇数构成。

- 所有数和为 $S$。

- 所有的前缀和中不能出现 $A$ 中的任意数字。

### Analysis

先把第三个限制（即前缀和限制）和数据范围忽略，考虑一个朴素的算法。

设 $f_{i}$ 表示当前和为 $i$ 的序列数量，那么有：

$$f_{i}=f_{i-1}+f_{i-3}+f_{i-5}+\dots$$

（可以取的数只有 $1,3,5,\dots$）

边界有：

$$f_{0}=1$$

这个 dp 的复杂度是 $O(S^2)$ 的，显然我们需要做一些优化。

#### Optimize.1

观察到，$f_{i-1}+f_{i-3}+f_{i-5}+\dots$ 是可以在边求 $f$ 的时候边维护的，我们设 $s_{i}$ 表示 $s_{i}=f_{i}+f_{i-2}+f_{i-4}+\dots$ ，则有：

$$f_{i} = s_{i-1}$$

$$s_{i} = s_{i-2} + f_{i}$$

这样，我们就在 $O(S)$ 的时间内求出了答案。

**但这还不够！** $S$ 的级别为 $10^{18}$ ，这个数据范围似乎在提示我们使用某种优化。

#### Optimize.2

观察到 $f,s$ 的递推式都是不变的，没有额外限制，且数据范围在 $10^{18}$ ，这些信息提示我们使用一种优化：

**矩阵快速幂**。

考虑我们需要维护的量：$f_{i},s_{i-1},s_{i-2}$ ，那么可以构造出向量：

$$
\begin{bmatrix}f_{i}\\ s_{i-1} \\ s_{i-2} \end{bmatrix}
$$


根据之前的递推关系，有系数矩阵：

$$
\begin{bmatrix}1 & 0 & 1 \\ 1 & 0 & 1
\\ 0 & 1 & 0\end{bmatrix} \begin{bmatrix}f_{i-1}\\ s_{i-2} \\ s_{i-3} \end{bmatrix} = \begin{bmatrix}f_{i}\\ s_{i-1} \\ s_{i-2} \end{bmatrix}
$$
用快速幂优化这个过程，我们就在 $O(\log_{2}S)$ 的复杂度下求出了答案。

#### Back to the Origin

接下来考虑前缀和限制。

把 “前缀和不能为 $a_{i}$” 写为形式化的语言，其实就是 $f_{a_{i}} = 0$ 。那么我们只需要在快速幂的中间，将 $f_{i} = 0$ 即可。

时间复杂度 $O(n+\log_{2}S)$。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const LL inf = 0x3f3f3f3f, mod = 998244353;
const LL N = 1e5 + 5, M = 4, si = 3;
LL n, m, a[N];
struct matrix
{
    int a[M][M];
    matrix()
    {
        memset(a, 0, sizeof a);
    }
    inline void build()
    {
        for (int i = 1; i <= si; i++)
        {
            a[i][i] = 1;
        }
    }
};
matrix operator*(const matrix &x, const matrix &y)
{
    matrix res;
    for (int i = 1; i <= si; i++)
    {
        for (int j = 1; j <= si; j++)
        {
            for (int k = 1; k <= si; k++)
            {
                res.a[i][j] += x.a[i][k] * y.a[k][j] % mod;
                res.a[i][j] %= mod;
            }
        }
    }
    return res;
}
matrix operator^(const matrix x, LL y)
{
    matrix base = x, res;
    res.build();
    while (y)
    {
        if (y & 1)
            res = base * res;
        base = base * base;
        y >>= 1;
    }
    return res;
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    
    matrix trans, cur, tmp;
    trans.a[1][1] = 1, trans.a[1][2] = 0, trans.a[1][3] = 1;
    trans.a[2][1] = 1, trans.a[2][2] = 0, trans.a[2][3] = 1;
    trans.a[3][1] = 0, trans.a[3][2] = 1, trans.a[3][1] = 0;
    cin >> n >> m;
    cur.a[1][1] = 1, cur.a[2][1] = 0, cur.a[3][1] = 0;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
        tmp = trans ^ (a[i] - a[i - 1]);
        cur = tmp * cur;
        cur.a[1][1] = 0;
    }
    tmp = trans ^ (m - a[n]);
    cur = tmp * cur;
    cout << cur.a[1][1] << '\n';
    return 0;
}
```

## Problem C. yichen

> Question By Powerless.

### Description

~~喜欢我超长题目背景吗~~

给出一个长为 $n$ 的序列以及 $m$ 次操作，每次操作给出 $l$ 和 $r$，表示对于 $\forall i \in [l,r]$，使 $a[i]$ 变为原来的平方，问最后所有数字的加和除以 $M$ 的余数是多少。（不保证 $M$ 是质数）

### Analysis

#### Step.1 静态处理

静态处理：先读入所有操作，再统一进行处理。

通过差分的方式记录每一个数最终被平方的次数。

具体为：在 $l$ 处打一个 $+1$ 的标记，在 $r+1$ 处打一个 $-1$ 的标记，从前往后遍历一次。

#### Step.2 单点求解

现在我们已经知道了每个数被平方的次数。

现在假设要处理数 a，它最终被平方了 k 次。

$ans = a^{2^k} \mod M$

这里的 k 可能会非常大，即便用快速幂也不行。

引入**欧拉定理**：

$a^{\varphi(m)} \equiv 1 \mod M $，其中 $\varphi(m)$ 为欧拉函数，表示 1 ~ m 中与 m 互质的数。

该定理可通过离散数学中的群论证明，此处略去。

将结论代入：

$ans = a^{2^k \mod \varphi(M) }\mod M$

预处理 $\varphi(M)$ 和 $2^i \mod \varphi(m), i\in [1,n]$ 即可。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 1e5 + 5;
LL a[N], c[N], MOD;
LL phi(LL x)
{
    LL ans = x;
    for (LL i = 2; i <= sqrt(x); i++)
        if (x % i == 0)
        {
            ans = ans / i * (i - 1);
            while (x % i == 0)
                x /= i;
        }
    if (x > 1)
        ans = ans / x * (x - 1);
    return ans;
}
LL k[N], v[N], t[N];
LL qpow(LL x, LL y, LL MOD)
{
    if (y == 0)
        return 1;
    if (y == 1)
        return x % MOD;
    LL u = qpow(x, y / 2, MOD) % MOD;
    return u * u % MOD * (y % 2 == 0 ? 1 : x) % MOD;
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m >> MOD;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
    }
    for (int i = 1; i <= m; i++)
    {
        int l, r;
        cin >> l >> r;
        c[l]++, c[r + 1]--;
    }
    for (int i = 1; i <= n; i++)
        c[i] += c[i - 1];
    LL phi_MOD = phi(MOD), ans = 0;
    k[0] = v[0] = 1;
    for (int i = 1; i <= m; i++)
    {
        k[i] = k[i - 1] * 2 % phi_MOD;
    }
    for (int i = 1; i <= m; i++)
    {
        v[i] = v[i - 1] * 2;
        if (v[i] > phi_MOD)
            break;
        t[i] = 1;
    }
    for (int i = 1; i <= n; i++)
    {
        if (t[c[i]] == 0)
            ans = (ans + qpow(a[i], k[c[i]] + phi_MOD, MOD)) % MOD;
        else
            ans = (ans + qpow(a[i], v[c[i]], MOD)) % MOD;
    }
    cout << ans << '\n';
    return 0;
}
```

## Problem D. Rational Bubble Sort

> Quesion By qwq.

### Description

给定一个由有理数构成的序列 $a_1,a_2,\ldots,a_n$，最初每个 $a_i$ 都是 $0$ 到 $10^6$（包含）之间的整数。

你可以执行如下操作任意多次（可以为零次）：

- 选择一个下标 $i$，使得 $1 \le i < n$，令 $z = \frac{1}{2}(a_{i} + a_{i+1})$。
- 然后，将 $a_i$ 和 $a_{i+1}$ 都赋值为 $z$。

例如，设 $a = [0,15,0]$。如果你对 $i=2$ 执行一次操作，那么 $a$ 变为 $[0,\color{red}{7.5},\color{red}{7.5}]$。

请判断是否有可能使 $a$ 变为非递减排序的序列。

### Analysis

设 $S_i$ 为 $a_{1 \sim i}$ 的平均值，注意到取平均只会改变 $S_i$ 的值、而 $S_{i+1}$ 保持不变，故考虑用 $S_i$ 描述问题：

$$
a_i \le a_{i+1} \Leftrightarrow S_i - S_{i-1} \le S_{i+1} - S_i
$$

差分使人联想到**凸性**：即 $(i, S_i)$ 在 $(i-1, S_{i-1}), (i+1, S_{i+1})$ 之下。

每次操作会使 $S_i - S_{i-1} = S_{i+1} - S_i$，即将折线 $(i-1, S_{i-1}), (i, S_i), (i+1, S_{i+1})$ **拉直**。

设坐标系中的基准线为 $(0,0)$ 到 $(n,S_n)$ 的直线，当所有 $(i,S_i)$ 都不低于这条线，若两个在线上的点夹着一个不在线上的点，则进行一次拉直即可把这个点拉回到线上；而对于一个长度 $\geq$ 2 的不在线上的点的连续段，任何操作都无法减少段内不在线上的点的数量，此时无解。

### Code

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int N = 2e5 + 50;
void solve()
{
    int n, sum = 0;
    cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; ++i)
    {
        cin >> a[i];
        sum += a[i];
    }
    int rag = 0, tag = 0, cnt = 0;
    for (int i = n - 1; i >= 0; --i)
    {
        rag += a[i];
        if (rag * n > sum * (n - i))
        {
            cout << "Yes" << endl;
            return;
        }
        else if (rag * n == sum * (n - i))
        {
            cnt = 0;
        }
        else if (rag * n < sum * (n - i))
        {
            ++cnt;
            tag |= (cnt >= 2);
        }
    }
    cout << (tag ? "No" : "Yes") << endl;
}
signed main()
{
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
}
```

## Problem E. Another Array Problem

> Quesion By Powerless.

### Description

给定一个长度为 $n$ 的整数数组 $a$。你可以对其进行如下操作任意多次（可以为 $0$ 次）：

- 选择两个下标 $i$、$j$，满足 $1 \le i < j \le n$，然后将所有 $i \leq k \leq j$ 的 $a_k$ 替换为 $|a_i - a_j|$。

请输出通过上述操作后，最终数组所有元素之和的最大值。

### Analysis

构造类结论题。

#### Step.1 n > 3

首先讨论 n > 3 的情况：

假设 a[p] 为数组 a 中的最大值，a[x] = m。

```text
a[]: ....m....
```

记 $|a[1]-a[2]| = x$，则对 (1, 2) 进行操作：

```text
a[]: xx..m....
```

再对 (1, 2) 进行操作：a[1] = a[2], a[1] - a[2] = 0。

```text
a[]: 00..m....
```

对 (1, p) 进行操作（p 为最大值 m 的下标）：

```text
a[]: mmmmm....
```

同理，对 (n - 1, n) 进行两次操作：

```text
a[]: mmmmm..00
```

再对 (p, n) 进行操作：

```text
a[]: mmmmmmmmm
```

**整个数组都变成了最大值 m**，和为 n*m。

由于 a 中所有元素都是正数，答案不会比 n*m 更大。

当最大值位于 1, 2 / n - 1, n 时也可以构造出全 m 的情况，请读者自行尝试构造。

#### Step.2 n = 2

当 n = 2 时，a 中只有两个元素，不妨记为 a, b。

只有操作 / 不操作两种情况：

- 操作：ans = $2\times|a-b|$
- 不操作：ans = a + b

两者取 max 即可。

#### Step.3 n = 3

对于最大值为 a[1], a[3] 的情况，我们仍然可以用 step.1 中的方案构造出全最大值。

对于最大值为 a[2] 的情况，我们来模拟进行操作会如何变化：

假设先操作 (2,3)， 再操作 (1,2) x 2，最后操作 (1, 3)

```text
a1          a2          a3
a1          a2-a3       a2-a3
|a1-a2+a3|  |a1-a2+a3|  a2-a3
0           0           a2-a3
a2-a3       a2-a3       a2-a3
```

最后得到和 3*(a2-a3)，

同理可以得到 3*(a2-a2)。

最后答案取最大值。

如果你对答案不放心，此处也可以多模拟几种不同情况。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 2e5 + 5;
LL n, a[N];
void solve()
{
    cin >> n;
    LL mx = 0;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
        mx = max(mx, a[i]);
    }
    LL ans = 0;
    if (n == 2)
    {
        ans = max(a[1] + a[2], 2 * abs(a[1] - a[2]));
    }
    else if (n == 3)
    {
        ans = a[1] + a[2] + a[3];
        ans = max(ans, 3 * a[1]);
        ans = max(ans, 3 * a[3]);
        ans = max(ans, 3 * abs(a[2] - a[1]));
        ans = max(ans, 3 * abs(a[2] - a[3]));
    }
    else
    {
        ans = n * mx;
    }
    cout << ans << '\n';
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
    return 0;
}
```

## Problem F. Bucket Bonanza

> Quesion By qwq.

### Description

给定 $n$ 个水桶，第 $i$ 个水桶能盛 $v_i$ 的水，每秒漏 $l_i$ 的的水。

第 $0$ 秒开始前你可以 **任意次数 **地将 **任意个数** 的水桶合并为一个，合并出的**水桶容量**取**最大值**，**漏水速度**取**最小值**。

给出 $q$ 组询问，每次问第 $0$ 秒加满水后第 $t_i$ 秒时所有水桶中水量和的最大值。

### Analysis

先不考虑有的水桶会漏完的情况。

合并两个水桶等价于 **删除两者间较小的水桶容量和较大的漏水速度**。

> 将 (a, d), (b, d) 合并为 (a, b) 等价于在 (a, b), (c, d) 中删去了 (c, d)。
>
> 忽略对应关系，而将替换转化为删除元素，是非常好用的 trick。

可以发现 **答案和** 水桶容量——漏水速度 **对应关系无关**，只和 所有水桶容量的集合 和 所有漏水速度的集合 有关。

对于每次询问，考虑贪心，每次尝试合并容量最小的水桶和漏水最多的水桶，直到最小容量大于最多漏水量。如果容量最小和漏水最多是同一个水桶，这个水桶不可能储水，等价于删除。

接下来考虑有的水桶会漏完的情况，此时把漏完的水桶和其它桶合并，答案不会变得更差。所以至少存在一个最优答案是水漏不完的（或只保留0个水桶），直接用之前的贪心解决。

每次询问二分删多少个水桶即可。

### Code

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int N = 2e5 + 50;
int v[N], d[N], sum_v[N], sum_d[N], n, q;
void solve()
{
    cin >> n;
    for (int i = 1; i <= n; ++i)
        cin >> v[i];
    for (int i = 1; i <= n; ++i)
        cin >> d[i];

    sort(v + 1, v + n + 1, [](int x, int y)
         { return x > y; });
    sort(d + 1, d + n + 1, [](int x, int y)
         { return x < y; });

    sum_v[0] = sum_d[0] = 0;

    for (int i = 1; i <= n; ++i)
        sum_v[i] = sum_v[i - 1] + v[i];
    for (int i = 1; i <= n; ++i)
        sum_d[i] = sum_d[i - 1] + d[i];

    cin >> q;
    while (q--)
    {
        int t;
        cin >> t;
        int l = 1, r = n + 1, mid;
        while (l < r)
        {
            mid = l + r >> 1;
            if (v[mid] > d[mid] * t)
            {
                l = mid + 1;
            }
            else
            {
                r = mid;
            }
        }
        cout << sum_v[l - 1] - sum_d[l - 1] * t << " ";
    }
    cout << endl;
}
signed main()
{
    int t = 1;
    cin >> t;
    while (t--)
    {
        solve();
    }
}
```

## Problem G. Manga

> Quesion By Powerless.

### Description

高桥准备阅读一套名为《Snuke-kun》的漫画，该系列共有 $10^9$ 卷。

最初，高桥拥有该系列的 $N$ 本书。第 $i$ 本书是第 $a_i$ 卷。

高桥可以**仅在他开始阅读之前**重复以下操作任意次（可能是零次）：

*   如果他只有 1 本或更少的书，则什么也不做；否则，卖掉他拥有的两本书，并买入一本任意卷数的书作为代替。

然后，高桥按顺序阅读第 1 卷、第 2 卷、第 3 卷……。然而，当他没有下一卷要读的书时，他就会停止阅读该系列（无论他拥有的其他卷数如何）。

求他能读到的该系列的最新卷数。如果他一本也读不了，则答案为 0。

### Analysis

简单的模拟题。

使用 set, vector, 二分均可。

#### Sol.1 set / vector

如果是 vector 就先排序， set 直接插入元素。

从 1 开始遍历，如果当前这本没有，就删掉**最大 / 多余**的两个，直到无法删去。

#### Sol.2 二分

读入时处理前 i 本中缺失几本，然后二分答案，看当前**大于答案**和**多余**的书够不够填补小于答案部分**缺失**的书。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const LL N = 6e5 + 5, inf = 0x3f3f3f3f;
int n, a[N], cnt[N], vis[N];
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
        if (a[i] < N)
            cnt[a[i]]++;
    }
    for (int i = 1; i < N; i++)
        vis[i] = vis[i - 1] + (!cnt[i]);
    sort(a + 1, a + 1 + n);
    int l = 1, r = N - 1, mid, res;
    while (l <= r)
    {
        mid = (l + r) / 2;
        if (mid + vis[mid] <= n)
            res = mid, l = mid + 1;
        else
            r = mid - 1;
    }
    cout << res << '\n';
    return 0;
}
```

## Problem H. Exactly K Steps

> Quesion By Powerless.

### Description

给定一棵包含 $N$ 个顶点的树。顶点编号为 $1, \dots, N$，第 $i$ 条边 $(1 \le i \le N-1)$ 连接顶点 $A_i$ 和 $B_i$。

我们定义树上顶点 $u$ 和 $v$ 之间的**距离**为从顶点 $u$ 到顶点 $v$ 的最短路径上的边数。

给定 $Q$ 个询问。在第 $i$ 个 $(1 \le i \le Q)$ 询问中，给定整数 $U_i$ 和 $K_i$，输出任意一个距离顶点 $U_i$ 恰好为 $K_i$ 的顶点编号。如果不存在这样的顶点，输出 `-1`。

### Analysis

设 $L$ 和 $R$ 为树直径的两个端点。那么，对于任意顶点 $u$，$L$ 或 $R$ 中必有一个是离 $u$ 最远的点。我们将通过反证法来证明这一点：

我们用 $d(u, v)$ 表示顶点 $u$ 和 $v$ 之间的距离。假设存在顶点 $A$ 和 $B$，使得 $d(A, B) > \max(d(A, L), d(A, R))$。

设 $a$ 为当树被视为以 $A$ 为根时，$L$ 和 $R$ 的 LCA（最近公共祖先）。类似地，为 $B$ 定义 $b$。

我们可以假设 $L, a, b, R$ 按此顺序出现在直径上。那么，
$$
d(A, a) + d(a, b) + d(b, B) \ge d(A, B) > d(A, R) = d(A, a) + d(a, R)
$$
所以，
$$
d(a, b) + d(b, B) > d(a, R)
$$
从而， 
$$
d(L, B) = d(L, a) + d(a, b) + d(b, B) > d(L, a) + d(a, R) = d(L, R)
$$
这与（$L, R$ 是直径端点）相矛盾。

因此，对于每个查询，我们可以将答案的候选范围缩小到从 $U_i$ 到 $L$ 的最短路径上的点，或者是从 $U_i$ 到 $R$ 的最短路径上的点。

通过预先读取查询，我们可以按如下方式解决问题：

从根节点开始进行 dfs（深度优先搜索）以管理从根节点到当前顶点的路径上的顶点。这可以通过在进入一个顶点时将其追加到一个数组中，并在退出时移除数组的最后一个元素来实现。

深度为 $d$ 的 $u$ 的祖先，就是进入 $u$ 时该维护数组的第 $d$ 个元素。

总时间复杂度为 $O(N + Q)$。

### Code

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2e5 + 50;
vector<int> e[N], path;
vector<pair<int, int>> qry[N];
int n, d[N], res[N];
int get_far(int u, int fa)
{
    d[u] = d[fa] + 1;
    int ans = u;
    for (auto v : e[u])
    {
        if (v == fa)
            continue;
        int p = get_far(v, u);
        if (d[p] > d[ans])
            ans = p;
    }
    return ans;
}
void dfs(int u, int fa)
{
    for (auto [id, k] : qry[u])
    {
        if (k <= path.size())
        {
            res[id] = path[path.size() - k];
        }
    }
    path.push_back(u);
    for (auto v : e[u])
    {
        if (v == fa)
            continue;
        dfs(v, u);
    }
    path.pop_back();
}
int main()
{
    cin >> n;
    for (int i = 1; i < n; ++i)
    {
        int u, v;
        cin >> u >> v;
        e[u].push_back(v);
        e[v].push_back(u);
    }
    int q;
    cin >> q;
    for (int i = 1; i <= q; ++i)
    {
        int u, k;
        cin >> u >> k;
        qry[u].push_back({i, k});
    }

    d[0] = 0;
    int L = get_far(1, 0), R = get_far(L, 0);

    memset(res, -1, sizeof(res));

    path.clear();
    dfs(L, 0);
    path.clear();
    dfs(R, 0);

    for (int i = 1; i <= q; ++i)
    {
        cout << res[i] << endl;
    }
    return 0;
}
```

## Problem I. Blackout 2

> Quesion By Powerless.

### Description

一个国家有 $N$ 个城市和 $M$ 个发电厂，我们将它们统称为“地点”。

这些地点的编号为 $1, 2, \dots, N + M$。其中：

-   地点 $1, 2, \dots, N$ 是**城市**。
-   地点 $N + 1, N + 2, \dots, N + M$ 是**发电厂**。

这个国家共有 $E$ 条输电线。第 $i$ 条输电线（$1 \le i \le E$）双向连接地点 $U_i$ 和地点 $V_i$。

如果一个城市可以通过若干条输电线到达至少一个发电厂，则称该城市是**通电的**。

现在会发生 $Q$ 个事件。在第 $i$ 个事件（$1 \le i \le Q$）中，第 $X_i$ 条输电线会断裂并无法使用。一旦输电线断裂，它在后续的所有事件中都将保持断裂状态。

请计算在每一个事件发生后，处于通电状态的城市数量。

### Analysis

图论套路题。

看到删边，果断离线倒序处理。

每次加一条边，对其端点所在连通块进行讨论。

- 一个点所在连通块中有电厂，另一个没有，就统计答案。
- 两个点所在连通块都没有电厂，合并连通块。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const LL N = 5e5 + 5, inf = 0x3f3f3f3f;
int T, n, m, k, now = 0;
int p[N], q[N], ans[N], si[N];
bool vis[N];
struct Edge
{
    int x, y;
} a[N];
inline int find(int x)
{
    if (p[x] == x)
        return x;
    return p[x] = find(p[x]);
}
inline void AddEdge(int x, int y)
{
    int px = find(x), py = find(y);
    if (px > py)
        swap(x, y), swap(px, py);
    if (px != py)
    {
        p[px] = py;
        if (px <= n && py > n)
            now += si[px];
        if (px <= n && py <= n)
            si[py] += si[px];
    }
}
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    
    int x, y;
    cin >> n >> k >> m;
    for (int i = 1; i <= n + k; i++)
        p[i] = i, si[i] = 1;
    for (int i = 1; i <= m; i++)
    {
        cin >> x >> y;
        a[i] = {x, y};
    }
    cin >> T;
    for (int i = 1; i <= T; i++)
    {
        cin >> q[i];
        vis[q[i]] = 1;
    }
    for (int i = 1; i <= m; i++)
    {
        if (!vis[i])
        {
            x = a[i].x, y = a[i].y;
            AddEdge(x, y);
        }
    }
    for (int i = T; i >= 1; i--)
    {
        x = a[q[i]].x, y = a[q[i]].y;
        ans[i] = now;
        AddEdge(x, y);
    }
    for (int i = 1; i <= T; i++)
        cout << ans[i] << '\n';
    return 0;
}

```

## Problem J. Optimal Shifts

> Quesion By qwq.

### Description

给定一个二进制字符串 $s_1s_2 \ldots s_n$，其中至少包含一个 $1$。你需要将其变为同样长度且全部由 $1$ 组成的二进制字符串。为此，你可以进行如下任意次操作：

选择一个整数 $d$（$1 \le d \le n$），将字符串 $s$ 进行循环右移 $d$ 位得到新字符串 $t$，形式化地表示为：$t = s_{n-d+1} \ldots s_{n} s_1 \ldots s_{n-d}$。之后，对所有 $j$ 满足 $t_j = 1$，执行 $s_j := 1$。该操作的花费为 $d$ 个金币。

注意，原本 $s_j=1$ 的位置即便 $t_j=0$ 也会维持为 $1$。

请你计算将 $s$ 变为全是 $1$ 的字符串所需的最小金币总数。

### Analysis

显然， 要让整个串都变为 1，肯定是修改越多越好。

贪心。假设我们移动了 2 位，花费为 2，进行一次修改。我们不如做 2 次如下操作：移动 1 位，花费为 1，进行一次修改。

因此，最优解的操是 **每移动一位，就修改一次**。

那么原题就等价为 **求原串中距离最远的两个 1 之间的距离**。

### Code

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int N = 1e6 + 50;
int n;
string s;
inline void solve()
{
    cin >> n;
    cin >> s;
    int st = 0, ans = 0, cnt = 0;
    for (int i = 0; i < s.size(); ++i)
        if (s[i] == '1')
        {
            st = i;
            break;
        }
    for (int i = st + 1; i != st; ++i)
    {
        if (i >= n)
            i -= n;
        if (i == st)
            break;
        if (s[i] == '0')
        {
            ++cnt;
            ans = max(ans, cnt);
        }
        else
        {
            cnt = 0;
        }
    }
    cout << ans << endl;
}
signed main()
{
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
    return 0;
}
```

## Problem K. Exceptional Segments

> Quesion By qwq.

### Description

给定两个整数 $n$ 和 $x$。

考虑序列 $[1, 2, 3, \dots, n]$。你需要找出其中**包含 $x$** 且**异或和（XOR）等于 0** 的子段（subsegments）的数量。换句话说，你需要统计满足以下条件的数对 $(l, r)$ 的数量：
$$1 \le l \le x \le r \le n \quad \text{且} \quad l \oplus (l + 1) \oplus \dots \oplus r = 0$$
其中 $\oplus$ 表示按位异或运算。

例如，如果 $n = 7$ 且 $x = 6$，则以下子段符合条件：

-   $(4, 7)$，因为 $x$ 位于该子段内，且 $4 \oplus 5 \oplus 6 \oplus 7 = 0$；
-   $(1, 7)$，因为 $x$ 位于该子段内，且 $1 \oplus 2 \oplus 3 \oplus 4 \oplus 5 \oplus 6 \oplus 7 = 0$。

由于答案可能非常大，请输出其对 $998\,244\,353$ 取模的结果。

### Analysis

观察样例和暴力程序输出后发现两个性质：

- 区间可以叠加：若 $(x, y), (y+1, z)$ 符合条件，则$ (x, z) $也符合条件。
- 最小区间仅有 2 种情况：$4k, 4k + 1, 4k + 2, 4k + 3$ 或 $4k + 2, 4k + 3, 4k + 4, 4k + 5$。

对于每一种情况，将 x 左侧和右侧分别有几个区间求出来然后相乘即可。

### Code

暴力的代码也贴在这里了。

```cpp
#include <bits/stdc++.h>
#define int long long
#define LL long long
#define pii pair<int, int>
using namespace std;
const LL N = 2e5 + 5, inf = 0x3f3f3f3f, mod = 998244353;
void solve()
{
    LL n, x;
    cin >> n >> x;

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
void bruteforce()
{
    int n, x;
    int ans = 0;
    cin >> n >> x;
    for (int l = 1; l <= n; l++)
    {
        for (int r = l; r <= n; r++)
        {
            if (l <= x && x <= r)
            {
                int x = 0;
                for (int i = l; i <= r; i++)
                    x ^= i;
                if (!x)
                {
                    ans++;
                    cout << l << ' ' << r << '\n';
                }
            }
        }
    }
    cout << ans << '\n';
}
signed main()
{
    int T;
    cin >> T;
    while (T--)
    {
        solve();
        // bruteforce();
    }
    return 0;
}
```

## Problem L. Interactive Graph (Hard Version)

> Quesion By qwq.

### Description

这是一个交互式问题。

命题人想出了一个没有环和重边的有向无环图（DAG），该图有 $n$ 个点和 $m$ 条边。

你的任务是确定图中具体有哪些边。为此，你可以提出形式为：“图中所有路径按字典序排序后的第 $k$ 条路径长什么样？”的问题。

图中的一条路径指的是一组顶点序列 $u_{1}, u_{2}, \dots, u_{l}$，使得对于任意 $i < l$，图中存在一条有向边 $(u_{i}, u_{i+1})$。

你需要通过不超过 $n+m$ 次的提问来完成任务。

$^{\ast}$ 序列 $a$ 在字典序上小于序列 $b$ 当且仅当满足下列之一：

- $a$ 是 $b$ 的前缀，且 $a \ne b$；
- 在 $a$ 和 $b$ 的第一个不同位置，$a$ 的对应元素小于 $b$ 的对应元素。

### Analysis

让我们从 2 开始依次询问路径，因为路径 1 包含一个顶点 1。如果我们一条一条地询问路径（即随后的每一条），那么在询问过程中，路径会以两种方式之一发生变化：

-   前一条路径上增加了一个顶点；
-   在前一条路径上删除了一个顶点后缀，并添加了一个新的顶点后缀。

请注意，当出现第二种情况时，对于从后缀中移除的所有顶点，我们可以确定从该顶点出发并以其他顶点为终点的路径数，我们将其记为 $C_v$ 。

因此，如果我们在某一时刻第二次访问一个顶点，我们可以立即跳过这些 $C_v$ 路径，并按词典顺序进入下一个顶点。

因此，对于每次查询，我们要么了解一条边，要么移动到另一个相连的部分，所以我们需要的问题正好是 $n+m$ 个。

## Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
const int N=35;
int n,m,d[N];
vector<int> e[N];
int dfs(int u)
{
    if(!d[u]){
        d[u]=1;
        for(auto v:e[u]){
            d[u]+=dfs(v);
        }
    }
    return d[u];
}
void solve()
{
    int k=2,tot=0;
    cin>>n;
    for(int i=0;i<=n;++i)
        e[i].clear();
    while(1){
        memset(d,0,sizeof(d));
        cout<<"? "<<k<<endl;
        int m,u,v;
        cin>>m;
        if(m==0){
            break;
        }
        else if(m==1){
            cin>>u;
            k+=dfs(u);
        }
        else{
            for(int i=1;i<m-1;++i)
                cin>>u;
            cin>>u>>v;
            e[u].push_back(v);
            ++tot;
            k+=dfs(v);
        }
    }
    cout<<"! "<<tot<<endl;
    for(int u=1;u<=n;++u){
        for(auto v:e[u]){
            cout<<u<<" "<<v<<endl;
        }
    }
}
signed main()
{
    int t; cin>>t;
    while(t--)
    {
        solve();
    }
}
```

## Problem M. Manipulating History

> Quesion By Powerless.

### Description

上白泽慧音具有操作历史程度的能力。   

幻想乡的历史，一开始是一个长度为 $1$ 的字符串 $s$。为了修复由于八云紫造成的历史错乱，她需要完成 $n$ 次操作。对于第 $i$ 次操作：

- 她会选择字符串 $s$ 中的一个非空子串 $t_{2i-1}$；
- 她会将 $t_{2i-1}$ 替换为 $t_{2i}$，注意这两者的长度可能是不一样的。

注意，如果 $t_{2i-1}$ 在字符串 $s$ 中出现多次，也仅仅只替换其中恰好一个。    

例如，如果有一个字符串 $s=\texttt{marisa}$，$t_{2i-1}=\texttt a$，$t_{2i}=\texttt z$，那么在一次操作后，字符串 $s$ 将会变成 $\texttt{marisz}$ 或者 $\texttt{mzrisa}$。    

在经过 $n$ 次这样的操作之后，慧音得到了最后的字符串以及 $2n$ 个 $t_i$。正当慧音觉得她完成了这项任务的时候，八云紫又一次出现并且打乱了所有的 $t_i$。更糟糕的是，慧音忘记了幻想乡最一开始的历史。     

请你帮助慧音求出幻想乡最一开始的历史。

### Analysis

第一眼是不是感觉是道后期题？实际上结论题 nanoda！

我们来考虑每一个字母。每一次操作，不妨记作将 $A$ 替换为 $B$。

那么有：

- $A$ 中所有字母在当前串中的次数 +1
- $B$ 中所有字母在当前串中的次数 -1

题目最后给出了最终串 $S$，我们可以理解成将 $S$ 替换为空。

那么最后所有字母出现的次数为 0，即：**除了第一个字母外，所有字母出现的次数都经过若干次 +1 / -1，最后变为 0**。

也就是说：**除了初始的第一个字母，其他所有字母都是成对出现的**。

那么我们只需要读入 2n+1 个字符串，统计出现次数为奇数的字母即可。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 1e5 + 5;
int n;
int cnt[26];
void solve()
{
    memset(cnt, 0, sizeof cnt);
    cin >> n;
    for (int i = 1; i <= 2 * n + 1; i++)
    {
        string s;
        cin >> s;
        for (int j = 0; j < s.size(); j++)
        {
            cnt[s[j] - 'a']++;
        }
    }
    for (int i = 0; i < 26; i++)
    {
        if (cnt[i] % 2)
        {
            cout << char(i + 'a') << '\n';
        }
    }
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
    return 0;
}
```

