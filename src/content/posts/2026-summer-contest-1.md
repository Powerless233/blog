---
title: 2026暑期训练赛1
published: 2026-07-12
tags: [Algorithm]
category: ACM
draft: false
---

# 2026 暑期训练赛 1

## Problem A

给定一个 $N \times N$ 的棋盘，每个格子初始为白色（`.`）或黑色（`#`）。 

你可以改变格子的颜色，目标是使最终状态满足：

1. **每行**从左到右，白格必须在黑格之前（即形如 `...###`）。 
2. **每列**从上到下，白格必须在黑格之前（即形如 `...###`）。 

求最少需要改变颜色的格子数量。

### Analysis

用 0/1 代表 白色/黑色。

容易想到朴素的 $O(n^3)$ dp：

设 $f_{i,j}$ 表示考虑前 $i$ 行，前 $j$ 个位置为 0，第 $j+1 \to n$ 个位置为 1。状态转移方程为：

$$f_{i,j} = \min\{f_{i-1,k} + cost(i,j)\}, k \in [j,n]$$

其中 $\texttt{cost(i,j)}$ 表示第 $i$ 行，将前 $j$ 个位置修改为 0，第 $j+1 \to n$ 个位置修改为 1 的代价。

注意到 $\texttt{cost(i,j)}$ 是不随着位置变化的定值，对于每一个 $(i,j)$，它的值是固定的。

因此可将状态方程改为：

$$f_{i,j} = cost(i,j)+\min\{f_{i-1,k}\} , k \in [j,n]$$

每一行求完之后，处理出后缀最小值即可。

$\texttt{cost(i,j)}$ 用前缀和求解，预处理一个每行前 $j$ 个位置的 1 的个数，然后根据定义计算。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 5000 + 5;
int n;
int a[N][N];
int f[N][N];
int g[N][N];
int c[N][N]; // count of 1
inline int cost(int i, int j)
{
    return c[i][j] + (n - j) - (c[i][n] - c[i][j]);
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    memset(f, 0x3f, sizeof f);
    memset(g, 0x3f, sizeof g);
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        string s;
        cin >> s;
        for (int j = 0; j < n; j++)
        {
            a[i][j + 1] = (s[j] == '#');
            c[i][j + 1] = c[i][j] + a[i][j + 1];
        }
    }
    for (int j = n; j >= 0; j--)
    {
        f[1][j] = cost(1, j);
        g[1][j] = min(f[1][j], g[1][j + 1]);
    }
    for (int i = 2; i <= n; i++)
    {
        for (int j = n; j >= 0; j--)
        {
            f[i][j] = cost(i, j) + g[i - 1][j];
            g[i][j] = min(f[i][j], g[i][j + 1]);
        }
    }

    int ans = 0x3f3f3f3f;
    for (int j = 0; j <= n; j++)
    {
        ans = min(ans, f[n][j]);
    }
    cout << ans << '\n';
    return 0;
}
```

## Problem B

有 $N$ 个排成一行的灯泡（初始全为 `0`），以及两个按钮 A 和 B：

* **按钮 A**：将**从左数第一个** `0` 变为 `1`。 

- **按钮 B**：将**从左数第一个** `1` 变为 `0`。

给定目标状态 $S_1, S_2, \dots, S_N$，请输出任意一个能达到该状态的按钮操作序列（长度需 $\le 10^6$）。题目保证有解。

### Analysis

题目不需要保证最优解，长度不超过 1e6，n 还只有 30，基本是随便做了。

给出一种构造方案：

从右往左（从 $n$ 到 1）看：设当前在处理第 $i$ 个位置，前 $i$ 个位置全是 0 或 全是 1。

若当前位置最终为 1，前 $i$ 个位置为全 0，则进行 $i$ 次操作 A。

若当前位置最终为 1，前 $i$ 个位置为全1，则不操作。

当前位置最终为 0 同理。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 1e5 + 5;
int a[N];
char s[N];
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n;
    cin >> n;
    string str;
    cin >> str;
    for (int i = 1; i <= n; i++)
    {
        s[i] = str[i - 1];
    }
    int st = n;
    string ans = "";
    for (int i = 1; i <= n; i++)
    {
        if (s[i] == '1')
            st = i;
    }
    for (int i = 1; i <= st; i++)
    {
        ans += 'A';
        a[i] = 1;
    }
    for (int i = st - 1; i >= 1; i--)
    {
        if (s[i] == '1')
        {
            if (a[i] == 1)
                continue;
            else
            {
                for (int j = 1; j <= i; j++)
                {
                    ans += 'A';
                    a[j] = 1;
                }
            }
        }
        else
        {
            if (a[i] == 0)
                continue;
            else
            {
                for (int j = 1; j <= i; j++)
                {
                    ans += 'B';
                    a[j] = 0;
                }
            }
        }
    }
    cout << ans.size() << '\n'
         << ans;
    return 0;
}
```

## Problem C

给定一个长度为 $N$、仅由 `A`、`B`、`C` 组成的字符串 $S$。

统计有多少个非空连续子串满足：其中 `A` 的数量严格大于 `B` 的数量。

### Analysis

对于子串的讨论，我们可以将其转化为区间问题来讨论。

对于 A 的数量严格大于 B，我们可以抽象为 A = 1，B = -1。

那么原问题就转化为：给定一个序列，求有多少个区间，它的区间和大于 0。

```text
例如 ABA 可以抽象为 1 -1 1
可行的字串有 (1,1) (3,3) (1,3) 它们的区间和都为 1
```

区间和用前缀和表示，对于区间 $(l,r)$，区间和 $sum = c_r - c_{l-1} > 0$。

那么对于一个特定的 $l$ 讨论，满足条件的 $r$ 有：$c_r > c_l-1$。

即统计 $(l,n)$ 上有多少个元素大于 $c_{l-1}$，使用树状数组求解即可。

### Code

实现上的细节：

- 需要避免树状数组的下标为负数，空间允许的情况下直接全部 +2n 即可。
- 答案会爆 int。

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const int N = 2e6 + 5;
int a[N];
int c[N];
int tr[N];
inline int lowbit(int x) { return x & -x; }
void update(int x, int z)
{
    while (x < N)
        tr[x] += z, x += lowbit(x);
}
int query(int x)
{
    int res = 0;
    while (x)
        res += tr[x], x -= lowbit(x);
    return res;
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n;
    cin >> n;
    string s;
    cin >> s;
    int ans = 0;
    for (int i = 1; i <= n; i++)
    {
        if (s[i - 1] == 'A')
            a[i] = 1;
        if (s[i - 1] == 'B')
            a[i] = -1;
        c[i] = c[i - 1] + a[i];
        ans += query(c[i] + 2 * n - 1);
        ans += (c[i] > 0);
        update(c[i] + 2 * n, 1);
        cout << '\n';
    }
    cout << ans << '\n';
    return 0;
}
```

## Problem D

给定一棵 $N$ 个节点的带权树（边编号 $1 \sim N-1$），初始边 $i$ 连接 $u_i, v_i$，权值为 $w_i$。

需处理 $Q$ 次操作： 

1. `1 i w`：将边 $i$ 的权值改为 $w$。 
2. `2 u v`：输出节点 $u$ 到 $v$ 的路径长度（边权和）。

### Analysis

树剖板子。

用 $a[x]$ 表示 $x - fa[x]$ 之间的边权，计算路径时减去 $2\times a[lca]$ 即可。

### Code

```cpp 
#include <bits/stdc++.h>
#define int long long
#define LL long long
using namespace std;
const LL inf = 0x3f3f3f3f;
const LL N = 2e5 + 5;
int n, m, rt, tot = 0;
int a[N];
int si[N], dep[N], fa[N], son[N], id[N], sp[N], top[N];
vector<int> e[N];
#define ls (k << 1)
#define rs (k << 1 | 1)
#define mid ((l + r) >> 1)
#define len (r - l + 1)
#define val(k) tr[k].val
#define lz(k) tr[k].lz
struct node
{
    int val, lz;
} tr[N << 2];
inline void dfs_son(int x, int f, int d)
{
    si[x] = 1;
    fa[x] = f;
    dep[x] = d;
    int maxson = 0;
    for (int i = 0; i < e[x].size(); i++)
    {
        int y = e[x][i];
        if (y != f)
        {
            dfs_son(y, x, d + 1);
            si[x] += si[y];
            if (si[y] > maxson)
                maxson = si[y], son[x] = y;
        }
    }
}
inline void dfs_chain(int x, int topf)
{
    id[x] = ++tot;
    sp[tot] = a[x];
    top[x] = topf;
    if (!son[x])
        return;
    dfs_chain(son[x], topf);
    for (int i = 0; i < e[x].size(); i++)
    {
        int y = e[x][i];
        if (y != fa[x] && y != son[x])
        {
            dfs_chain(y, y);
        }
    }
}
inline void pushup(int k)
{
    val(k) = (val(ls) + val(rs));
}
inline void pushdown(int k, int l, int r)
{
    if (!lz(k))
        return;
    lz(ls) += lz(k);
    lz(rs) += lz(k);
    val(ls) += (mid - l + 1) * lz(k);
    val(rs) += (r - mid) * lz(k);
    lz(k) = 0;
}
inline void build(int k, int l, int r)
{
    if (l == r)
    {
        val(k) = sp[l];
        return;
    }
    build(ls, l, mid);
    build(rs, mid + 1, r);
    pushup(k);
}
inline void update(int k, int l, int r, int ql, int qr, int z)
{
    if (l >= ql && r <= qr)
    {
        val(k) += z * len;
        lz(k) += z;
        return;
    }
    pushdown(k, l, r);
    if (ql <= mid)
        update(ls, l, mid, ql, qr, z);
    if (qr > mid)
        update(rs, mid + 1, r, ql, qr, z);
    pushup(k);
}
inline int query(int k, int l, int r, int ql, int qr)
{
    if (l >= ql && r <= qr)
        return val(k);
    pushdown(k, l, r);
    int res = 0;
    if (ql <= mid)
        res += query(ls, l, mid, ql, qr);
    if (qr > mid)
        res += query(rs, mid + 1, r, ql, qr);
    return res;
}
inline void range_update(int x, int y, int z)
{
    while (top[x] != top[y])
    {
        if (dep[top[x]] < dep[top[y]])
            swap(x, y);
        update(1, 1, n, id[top[x]], id[x], z);
        x = fa[top[x]];
    }
    if (dep[x] > dep[y])
        swap(x, y);
    update(1, 1, n, id[x], id[y], z);
}
inline int range_query(int x, int y)
{
    int res = 0;
    while (top[x] != top[y])
    {
        if (dep[top[x]] < dep[top[y]])
            swap(x, y);
        res += query(1, 1, n, id[top[x]], id[x]);
        x = fa[top[x]];
    }
    if (dep[x] > dep[y])
        swap(x, y);
    res += query(1, 1, n, id[x], id[y]);
    return res;
}
inline void tree_update(int x, int z)
{
    update(1, 1, n, id[x], id[x] + si[x] - 1, z);
}
inline int tree_query(int x)
{
    return query(1, 1, n, id[x], id[x] + si[x] - 1);
}
inline int lca(int x, int y)
{
    while (top[x] != top[y])
    {
        if (dep[top[x]] < dep[top[y]])
            swap(x, y);
        x = fa[top[x]];
    }
    if (dep[x] > dep[y])
        swap(x, y);
    return x;
}
struct pEdge
{
    int x, y, w;
} pe[N];
signed main()
{
    cin >> n;
    for (int i = 1; i < n; i++)
    {
        int x, y, w;
        cin >> x >> y >> w;
        e[x].push_back(y);
        e[y].push_back(x);
        pe[i] = {x, y, w};
    }
    dfs_son(1, 0, 0);
    for (int i = 1; i < n; i++)
    {
        auto [x, y, w] = pe[i];
        if (y == fa[x])
        {
            a[x] = w;
        }
        else
        {
            a[y] = w;
        }
    }
    dfs_chain(1, 1);
    build(1, 1, n);
    int q;
    cin >> q;
    for (int i = 1; i <= q; i++)
    {
        int opt, c, d;
        cin >> opt >> c >> d;
        if (opt == 1)
        {
            auto [x, y, w] = pe[c];
            if (y == fa[x])
            {
                range_update(x, x, d - w);
                a[x] = d;
            }
            else
            {
                range_update(y, y, d - w);
                a[y] = d;
            }
            pe[c].w = d;
        }
        else
        {
            int lc = lca(c, d);
            int res = range_query(c, lc) + range_query(d, lc) - 2 * a[lc];
            cout << res << '\n';
        }
    }
    return 0;
}
```

## Problem E

给定 $N \times N$ 棋盘（`.`为空地，`#`为障碍），玩家从 $(N, C)$ 出发，尝试向上移动最多 $N-1$ 步到达第一行。

**移动规则**： 

每次向上一行（左上、正上或右上）移动一格： 

- 若目标为空地，直接移动。 
- 若目标有障碍，仅当该格**正下方所有格子均为空**时，消除障碍并移动；否则**移动失败且立即终止**。

### Analysis

这题赛时的榜歪了，实际爆搜就可以了。

在正常从下往上广搜的基础上，多加一个是否可以将这一列障碍清除的标记即可。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 3000 + 5;
int n, m;
int a[N][N], c[N][N];
int topc[N];
bool vis[N][N];
int ans[N];
void init(int n)
{
    for (int i = 1; i <= n + 1; i++)
    {
        for (int j = 1; j <= n + 1; j++)
        {
            vis[i][j] = 0;
        }
        c[n + 1][i] = 0;
        ans[i] = topc[i] = 0;
    }
}
void solve()
{
    cin >> n >> m;
    init(n);

    string s;
    for (int i = 1; i <= n; i++)
    {
        cin >> s;
        for (int j = 1; j <= n; j++)
        {
            a[i][j] = (s[j - 1] == '#');
        }
    }
    for (int j = 1; j <= n; j++)
    {
        for (int i = n; i >= 1; i--)
        {
            c[i][j] = c[i + 1][j] + a[i][j];
        }
    }
    queue<pair<int, int>> q;
    q.push(make_pair(n, m));
    while (!q.empty())
    {
        auto [x, y] = q.front();
        q.pop();
        if (vis[x][y])
            continue;
        vis[x][y] = 1;
        if (x == 1)
        {
            ans[y] = 1;
            continue;
        }
        for (int i = -1; i <= 1; i++)
        {
            int dx = x - 1, dy = y + i;
            if (dy < 1 || dy > n)
                continue;
            if (!a[dx][dy])
                q.push(make_pair(dx, dy));
            else if (!c[dx + 1][dy])
            {
                q.push(make_pair(dx, dy));
                topc[dy] = 1;
            }
            else if (topc[dy])
            {
                q.push(make_pair(dx, dy));
            }
        }
    }
    for (int i = 1; i <= n; i++)
    {
        cout << ans[i];
    }
    cout << '\n';
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T;
    cin >> T;
    while (T--)
    {
        solve();
    }
    return 0;
}
```

## Problem F

桌上有 $N$ 个杯子，容量分别为 $A_i$。其中恰有 $K$ 杯是清酒，其余是水（具体分布未知）。 

选择若干个杯子喝掉，要求**无论哪 $K$ 杯是清酒**，都能保证至少喝到 $X$ 毫升清酒。 

求满足条件的最少选择杯数；若无法满足，输出 $-1$。

### Analysis

这类问题需要考虑最坏情况。

本题的最坏情况是：你没选的全是酒，你选的当中最少的那几杯是酒。

贪心，选最大的若干杯，然后统计最坏情况下是否满足条件。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const int N = 3e5 + 5;
int n, k, x;
int a[N], c[N];
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> k >> x;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
    }
    sort(a + 1, a + 1 + n);
    for (int i = 1; i <= n; i++)
    {
        c[i] = c[i - 1] + a[i];
    }
    for (int l = n; l >= 1; l--)
    {
        // pick l->n
        int len = n - l + 1;
        int t = k - (n - len);
        if (t <= 0)
            continue;
        if (c[l + t - 1] - c[l - 1] >= x)
        {
            cout << len << '\n';
            return 0;
        }
    }
    cout << "-1\n";
    return 0;
}
```

## Problem G

给定长度为 $N$ 的非递减数组 $A$ 和 $Q$ 次询问。

每次询问给出区间 $[L, R]$，求该区间内最多能组成多少对 $(a, b)$，满足小元素 $a \le \frac{b}{2}$（每个元素仅能用一次）。

### Analysis

首先考虑对于一个特定的区间，最优方案的形式一定是对半切，然后逐一配对：

```text
1 2 3 ... | ... 4 5 6
```

因为至多只会匹配 $len / 2$ 组，所以前半部分的元素一定是匹配到后半的元素。

假设可以匹配 $k$ 对，那么形如上图的匹配方案一定成立（取最前面 $k$ 个和最后面 $k$ 个）。

记 $b_x$ 为满足 $ 2\times a_y \le a_x $ 的最大的 $y$，即小于 $x$ 的，能与 $x$ 配对的最大下标。

设当前区间为 $(l,r)$，那么对于每一个 $x \in [r-k+1,r]$，都有 $b_x \geq x + k - (r-l)$，

即： $b_x - x \geq k - r + l$

对于同一个 $k$，右式的结果是定值，左式通过预处理与 $k$ 无关；因此，可以通过预处理 st 表 O(1) 判断上式是否成立。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 2e5 + 5;
int n, a[N], b[N];
int f[N][32];
inline int rmq(int x, int y)
{
    int t = log2(y - x + 1);
    return min(f[x][t], f[y - (1 << t) + 1][t]);
}
inline bool check(int L, int R, int k)
{
    return rmq(R - k + 1, R) >= L - R + k - 1;
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n;
    memset(f, 0x3f, sizeof f);
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
    }
    for (int l = n, r = n; r >= 1 && l >= 1; r--)
    {
        while (l >= 1 && 2 * a[l] > a[r])
            l--;
        b[r] = l;
    }
    for (int i = 1; i <= n; i++)
        f[i][0] = b[i] - i;
    for (int k = 1; k <= 30; k++)
    {
        for (int i = 1; i + (1 << k) - 1<= n; i++)
        {
            f[i][k] = min(f[i][k - 1], f[i + (1 << k - 1)][k - 1]);
        }
    }
    int q;
    cin >> q;
    while (q--)
    {
        int L, R;
        cin >> L >> R;
        int l = 0, r = (R - L + 1) / 2, k;
        while (l < r)
        {
            k = (l + r + 1) >> 1;
            if (check(L, R, k))
                l = k;
            else
                r = k - 1;
        }
        cout << l << '\n';
    }
    return 0;
}
```

## Problem H

有 $N$ 个人和 $N$ 个容器（编号 $1 \sim N$）。初始时第 $i$ 个人持有第 $i$ 个容器（容器为空）。 

每轮所有人**同时**执行以下操作： 

1. 向当前持有的**所有**容器中倒入等于自己编号 $i$ 的水量。
2.  将这些容器全部交给第 $A_i$ 个人。 

给定 $Q$ 次查询，每次给出 $T_i, B_i$，求第 $T_i$ 轮结束后，容器 $B_i$ 中的总水量。

### Analysis

将所有 $i$ 向  $a_i$ 连边，得到的图是基环树森林。对于树上跳跃 k 次求路径和，没有比 $\log n$ 更快的做法。

那么只需要考虑带 $\log$ 的做法，本题用倍增和矩阵快速幂都可做，不过倍增的码量会少很多。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const int N = 2e5 + 5;
int n, q, f[N][32], sum[N][32];
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> q;
    for (int i = 1; i <= n; i++)
    {
        cin >> f[i][0];
        sum[i][0] = i;
    }
    for (int k = 1; k <= 30; k++)
    {
        for (int i = 1; i <= n; i++)
        {
            f[i][k] = f[f[i][k - 1]][k - 1];
            sum[i][k] = sum[f[i][k - 1]][k - 1] + sum[i][k - 1];
        }
    }
    while (q--)
    {
        int t, b;
        cin >> t >> b;
        int ans = 0;
        for (int i = 0; i <= 30; i++)
        {
            if (t & (1 << i))
                ans += sum[b][i], b = f[b][i];
        }
        cout << ans << '\n';
    }
    return 0;
}
```

## Problem I

给定一个 $N$ 个顶点的完全有向图，边 $(i, j)$ 的代价为 $C_{i,j}$。另给定正整数 $K$。 

对于每个起点 $s \in [1, N]$，求一条**恰好经过 $K$ 条边**并**回到 $s$** 的回路，使得路径总代价最小。

### Analysis

本题有着过于独特的数据范围：K = 1e9，这留给我们的时间复杂度只可能是 $\log K$ 了。

这种问题被称作 min-plus 矩阵快速幂，也成为距离积。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const LL inf = 0x3f3f3f3f3f3f3f3f;
const LL N = 105;
LL n, k;
struct matrix
{
    LL a[N][N];
    matrix()
    {
        memset(a, 0, sizeof a);
    }
    void setMax()
    {
        for (int i = 1; i <= n; i++)
        {
            for (int j = 1; j <= n; j++)
            {
                a[i][j] = inf;
            }
        }
    }
};
matrix operator*(const matrix &x, const matrix &y)
{
    matrix res;
    res.setMax();
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n; j++)
        {
            for (int k = 1; k <= n; k++)
            {
                res.a[i][j] = min(res.a[i][j], x.a[i][k] + y.a[k][j]);
            }
        }
    }
    return res;
}
matrix operator^(const matrix &x, LL y)
{
    matrix res = x, base = x;
    y--;
    while (y)
    {
        if (y & 1)
            res = res * base;
        base = base * base;
        y >>= 1;
    }
    return res;
}
signed main()
{
    matrix a;
    cin >> n >> k;
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n; j++)
        {
            cin >> a.a[i][j];
        }
    }
    matrix b = a;
    if (k)
        b = a ^ k;
    for (int i = 1; i <= n; i++)
    {
        cout << b.a[i][i] << '\n';
    }
    return 0;
}
```

## Problem J

### Analysis

本题的难点在于排序的重载。

一个比较好想（但写起来比较垃圾）的写法：将四个想象和四条坐标轴按顺时针编号 1~8，排序时先按照这个编号排序，若编号相同（即在同一象限 / 坐标轴）则按照斜率比较。

同一根线上会有多个点，所以再手写一个去重。

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const LL N = 2e5 + 5, inf = 0x3f3f3f3f;
struct Node
{
    int id, x, y, fl;
} a[N], b[N];
int getfl(int x, int y)
{
    if (x < 0 && y > 0)
        return 1;
    if (x == 0 && y > 0)
        return 2;
    if (x > 0 && y > 0)
        return 3;
    if (x > 0 && y == 0)
        return 4;
    if (x > 0 && y < 0)
        return 5;
    if (x == 0 && y < 0)
        return 6;
    if (x < 0 && y < 0)
        return 7;
    if (x < 0 && y == 0)
        return 8;
}
bool cmp(Node a, Node b)
{
    if (a.fl != b.fl)
        return a.fl < b.fl;
    LL xa = a.x, ya = a.y, xb = b.x, yb = b.y;
    if (a.fl == 1)
    {
        xa = -xa, xb = -xb;
        return xb * ya < xa * yb;
    }
    if (a.fl == 3)
    {
        return xb * ya > xa * yb;
    }
    if (a.fl == 5)
    {
        ya = -ya, yb = -yb;
        return xb * ya < xa * yb;
    }
    if (a.fl == 7)
    {
        return xb * ya > xa * yb;
    }
}
bool idd(Node a, Node b)
{
    LL xa = a.x, ya = a.y, xb = b.x, yb = b.y;
    return a.fl == b.fl && xb * ya == xa * yb;
}
int n, q, rid[N], cnt[N], c[N];
int tot = 0;
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> q;
    for (int i = 1; i <= n; i++)
    {
        int x, y;
        cin >> x >> y;
        a[i] = {i, x, y, getfl(x, y)};
    }
    sort(a + 1, a + 1 + n, cmp);
    b[++tot] = a[1];
    cnt[tot] = 1;
    rid[a[1].id] = 1;
    for (int i = 2; i <= n; i++)
    {
        if (idd(a[i], a[i - 1]))
        {
            cnt[tot]++;
            rid[a[i].id] = tot;
        }
        else
        {
            b[++tot] = a[i];
            cnt[tot] = 1;
            rid[a[i].id] = tot;
        }
    }
    for (int i = 1; i <= tot; i++)
    {
        c[i] = c[i - 1] + cnt[i];
    }
    while (q--)
    {
        int a, b;
        cin >> a >> b;
        int st = rid[a], en = rid[b];
        if (st <= en)
        {
            cout << c[en] - c[st - 1] << '\n';
        }
        else
        {
            cout << n - c[st - 1] + c[en] << '\n';
        }
    }
    return 0;
}
```

## Problem K

给定一个 $N \times N$ 的 01 矩阵 $A$。

构建一个 $NK \times NK$ 的有向图，其邻接矩阵由 $K \times K$ 个 $A$ 的副本平铺而成（即分块矩阵）。 

给定 $Q$ 次询问，每次给出起点 $s_i$ 和终点 $t_i$，求两点间的**最短路径长度**。若不可达，输出 `-1`。

### Analysis

整个大图由 $K \times K$ 个完全相同的子图块平铺而成,这意味着所有子图块在拓扑结构上是同构的，因此，我们将任意节点 $(x,y)$ 映射为其在基础块 A 中的相对坐标 $(x \mod N, y \mod N)$。我们可以直接在原图上求最短路径问题。

```text
以下根据样例讨论
从 (1,1) 到 (1,5) 的长度将会等于 (1,1) 到 (1,2) 的长度。
为什么？邻接图如下：
1 1 1 1 1 1
1 1 0 1 1 0
0 1 0 0 1 0
1 1 1 1 1 1
1 1 0 1 1 0
0 1 0 0 1 0
(1,2) 之间存在连边，因此 (1,5) 之间也存在连边。
因为节点 (1,5) 在 A 中的映射即为 (1,2)，在复制的分块图中也必定会有相应的边。
```

同时要处理特殊情况：起点与终点的映射节点相同。

若起点与终点本身相同，则最短路径为 0。

若起点与终点本身不同，则需要求解环路。

本题的 N = 100，直接在 A 上跑一边 Floyd 即可。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const int N = 100 + 5, inf = 0x3f3f3f3f3f3f3f3f;
int n, k;
int dis[N][N];
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    memset(dis, 0x3f, sizeof dis);

    cin >> n >> k;
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n; j++)
        {
            int x;
            cin >> x;
            if (x)
                dis[i][j] = 1;
        }
    }
    for (int k = 1; k <= n; k++)
    {
        for (int i = 1; i <= n; i++)
        {
            for (int j = 1; j <= n; j++)
            {
                if (i == k || j == k)
                    continue;
                dis[i][j] = min(dis[i][j], dis[i][k] + dis[k][j]);
            }
        }
    }
    int q;
    cin >> q;
    while (q--)
    {
        int a, b;
        cin >> a >> b;
        if (a == b)
        {
            cout << "0\n";
            continue;
        }
        a = (a - 1) % n + 1;
        b = (b - 1) % n + 1;
        if (dis[a][b] < inf)
            cout << dis[a][b] << '\n';
        else
            cout << "-1\n";
    }
    return 0;
}
```

## Problem L

有 $N$ 种商品，第 $i$ 种价格为 $P_i$，价值为 $V_i$（各一件）。

给定预算 $M$（保证 $P_i \le M$），求在总价格不超过 $M$ 的前提下，能获得的最大总价值。 

在此基础上，对每种商品 $i$ 进行分类： 

- **Type A**：出现在**所有**最优购买方案中。
- **Type B**：出现在**部分**最优购买方案中。
- **Type C**：**不出现**在任何最优购买方案中。

### Analysis

其实本做法有点极限。赛后跟 qwq 交流，qwq 也有想过用 bitset，但最后怕超时就没用。

用长度为 n 的 01 串表示当前预算下最优解第 $i$ 个物品是否出现在方案中。

然后用两个 bitset f,g 分别代表方案的交和并集。

若一个物品出现在**交集**中，则说明**所有方案**都有它。

若一个物品出现在**并集**中，则说明**部分方案**中有它。

时间复杂度 $O(\frac{1}{\omega} \times nm^2)$，赛时以 576ms 通过。

### Code

求答案的时候别忘了最优解不一定把背包用完。

```cpp
#include <bits/stdc++.h>
#define int long long
#define LL long long
using namespace std;
const int N = 1005, M = 5e4 + 5;
int n, m, w[N], c[N];
int dp[M];
bitset<N> g[M];
bitset<N> f[M];
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
    {
        cin >> w[i] >> c[i];
    }
    for (int i = 1; i <= n; i++)
    {
        for (int j = m; j >= w[i]; j--)
        {
            if (dp[j] < dp[j - w[i]] + c[i])
            {
                dp[j] = dp[j - w[i]] + c[i];
                g[j] = g[j - w[i]];
                g[j][i] = 1;
                f[j] = f[j - w[i]];
                f[j][i] = 1;
            }
            else if (dp[j] == dp[j - w[i]] + c[i])
            {
                g[j][i] = 1;
                g[j] |= g[j - w[i]];
                f[j] &= f[j - w[i]];
            }
        }
    }
    bitset<N> ansa, ansb;
    int ans = 0, st = 0;
    bool fl = 1;
    for (int i = 0; i <= m; i++)
    {
        if (dp[i] > ans)
            ans = dp[i], st = i;
    }
    ansb = g[st];
    ansa = f[st];
    for (int i = st + 1; i <= m; i++)
    {
        ansb |= g[i];
        ansa &= f[i];
    }
    for (int i = 1; i <= n; i++)
    {
        if (ansa[i])
            cout << 'A';
        else if (ansb[i])
            cout << 'B';
        else
            cout << 'C';
    }
    return 0;
}

```

