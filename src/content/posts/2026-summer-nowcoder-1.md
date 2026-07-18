---
title: 2026牛客暑期多校1
published: 2026-07-18
tags: [Contest]
category: ACM
draft: false
---

# 2026 牛客暑期多校 1

基本都是没见过的题型，跪下了。

## Problem G

### Description

给定一个整数 $n$，你需要构造一个集合 $S$，该集合由三维欧几里得空间中的不超过 $(2n + 2)$ 个不同的点组成。要求对于集合 $S$ 中的每个点 $P$，恰好存在 $n$ 个不同的点 $Q \in S$，使得 $\text{dis}(P, Q) = 1$，其中 $\text{dis}(P, Q)$ 表示 $P$ 和 $Q$ 之间的欧几里得距离。
*图注：正二十面体，这是 $n=5$ 时的一个可行解。*

为了容忍**精度误差**，设 $\epsilon = 0.01$。当且仅当满足以下条件时，你的答案将被视为正确：

-   对于任意两个不同的点 $P, Q \in S$，满足 $\text{dis}(P, Q) > \epsilon$；
-   对于每个点 $P \in S$，恰好有 $n$ 个不同的点 $Q \in S$，满足 $1 - \epsilon < \text{dis}(P, Q) < 1 + \epsilon$。

$1 \le n \le 100$， $-100 \le x, y, z \le 100$。

可以证明，在给定约束下总是存在可行解。如果存在多个可行解，你可以输出任意一个。

### Analysis

Checker Hack 题。

对于这种构造题，$\epsilon = 0.01$ 和 $n = 100$ 看起来实在太奇怪了。一般来说  $\epsilon$ 会是 $10^{-6},10^{-7}$ 这种级别才会是真的在容忍精度误差，而这里的 $0.01$ 和 $100$ 之间的关系很容易引起怀疑。

赛时这题半小时就有人过了，并且经过半小时的思考发现常规的构造方案似乎非常不可做，于是坚信这题就是个乱搞题了。

---

那么这题的思路就清晰了：构造方案大概就是在 $(0,0,0)$ 和 $(0,0,1)$ 这两个点附近放一堆点，每个点之间的距离 $>\epsilon$，且这两堆点之间的距离 $1-\epsilon<dis(P,Q) < 1+\epsilon$。

$\sqrt{1+x^2} \sim 1 + x^2/2 \Rightarrow \sqrt{1+2\epsilon} \approx 1 + \epsilon$

对于平面 $z=0$ 内的点，在平面 $z=1$ 内有一个半径约为 $\sqrt{2\epsilon}$ 的圆内的点与它的欧式距离在 $[1, 1+\epsilon)$ 范围内。

考虑在两个平面上相对的半径约为 $\sqrt{2\epsilon}/2$ 圆内分别放入 $n$ 个两两距离 $>\epsilon$ 的点，按照正方形网格密铺即可在 $\epsilon=0.01$ 的限制下放入 $\le 100$ 个点。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 1e5 + 5;
void draw(int n, int z)
{
    int m = sqrt(n) + 1;
    int d = m / 2;
    int cnt = n;
    for (int i = -d, t = 1; t <= m; i++, t++)
    {
        if (!cnt)
            break;
        for (int j = -d, p = 1; p <= m; j++, p++)
        {
            if(!cnt)
                break;
            printf("%.6lf %.6lf %.6lf\n", (double)i * 0.01001, (double)j * 0.01001, (double)z);
            cnt--;
        }
    }
}
void solve()
{
    int n;
    cin >> n;
    cout << 2 * n << '\n';
    draw(n, 0);
    draw(n, 1);
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

## Problem C

给定一个 $n \times m$ 的网格，其中一些格子包含鱼，其余格子是障碍物。网格上有一条鱼由你控制，你可以反复将其移动到任何相邻的格子（上、下、左或右），但不能移出网格：如果目标格子包含一条尺寸**不大于**其自身的鱼，它可以吃掉那条鱼并且其尺寸增加 1；如果目标格子是障碍物或包含一条严格更大的鱼，它就不能移动到那里。

最初，每个格子都是障碍物。依次有 $q$ 个操作，分为以下两种类型：

1. 移除 $(x, y)$ 处的障碍物，并在那里放置一条鱼尺寸为 $v$ 的鱼。求出在你的控制下这条鱼最多能吃掉多少条鱼。保证 $(x, y)$ 当前是障碍物，并且 $v$ **不小于**所有先前放置的鱼的大小。

2. 假设在虚拟过程开始之前，你可以将 $(x, y)$ 处鱼的初始尺寸增加一个**非负整数**。设 $k$ 为在所有增大方案中它最多可能吃掉的鱼的数量。求出为了吃掉 $k$ 条鱼所需的最小增大值。保证 $(x, y)$ 处已经放置了一条鱼。

对于这两种类型的操作，吃鱼的过程都是虚拟的，并不会改变任何已放置鱼的实际状态。

$1 \le n, m, n \times m \le 2.5 \times 10^5$，$1 \le q \le 5 \times 10^5$，代表网格的尺寸和操作的数量。

时间限制为 4s。

### Analysis

这题的官方题解说的很明白了，再写一遍题解也大同小异。

按鱼的大小非降序加入点，考虑动态维护 Kruskal 重构树。

每个点 $u$ 维护一个 $sz_u$，表示子树内鱼的数量，以及一个 $w_u$，表示往上走需要的大小，那么对于一条大小为 $v$ 的鱼，假设它走到了点 $x$，那么大小会变成 $v + sz_x - 1$，如果还要往上走，则需要 $v + sz_x - 1 \ge w_x$。

所以也说明了一点，如果一条鱼要走到重构树的根，那么它的大小 $v$ 应该大于等于它到根上所有点的 $w_x - sz_x + 1$。所以每个点除了维护并查集祖先 $Fa_u$，再维护一个 $Max_u$，表示 $u$ 到 $Fa_u$ 的 $w - sz + 1$ 的最大值。

考虑两个询问，第一问就是加入一条鱼之后输出并查集根节点的 $sz - 1$，然后对于一条大小为 $v_u$ 的鱼 $u$，第二问的答案就是 $\max(Max_u - v_u, 0)$。

实际上我们不需要把重构树显式地构建出来，只需要维护 $Fa, sz, w, Max$ 等信息就可以了。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
using namespace std;
const int N = 5e5 + 5;
int n, m, q;
int dx[4] = {1, 0, -1, 0};
int dy[4] = {0, 1, 0, -1};
int p[N];
int val[N];
int si[N];
inline int find(int x)
{
    if (p[x] == x)
        return x;
    int fa = find(p[x]);
    val[x] = max(val[x], val[p[x]]);
    p[x] = fa;
    return fa;
}
signed main()
{
    // ios::sync_with_stdio(false);
    // cin.tie(nullptr);
    cin >> n >> m >> q;
    vector<vector<int>> a(n + 1, vector<int>(m + 1, 0));
    vector<vector<int>> id(n + 1, vector<int>(m + 1, 0));
    int lstans = 0;
    int tot = 0;
    while (q--)
    {
        int opt, x, y, w;
        cin >> opt >> x >> y;
        x ^= lstans, y ^= lstans;
        if (opt == 1)
        {
            cin >> w;
            a[x][y] = w;
            int u = ++tot;
            p[u] = id[x][y] = u;
            si[u] = 1, val[u] = 0;
            for (int i = 0; i < 4; i++)
            {
                int nx = x + dx[i], ny = y + dy[i];
                if (nx && nx <= n && ny && ny <= m && id[nx][ny])
                {
                    int v = id[nx][ny];
                    int pu = find(u), pv = find(v);
                    if (pu != pv)
                    {
                        int rt = ++tot;
                        p[rt] = rt, val[rt] = 0;
                        si[rt] = si[pu] + si[pv];
                        p[pu] = p[pv] = rt;
                        val[pu] = max(val[pu], w - si[pu] + 1);
                        val[pv] = max(val[pv], w - si[pv] + 1);
                        si[pu] = si[pv] = 0;
                    }
                }
            }
            lstans = si[find(u)] - 1;
            cout << lstans << '\n';
        }
        else
        {
            find(id[x][y]);
            lstans = max(0, val[id[x][y]] - a[x][y]);
            cout << lstans << '\n';
        }
    }
    return 0;
}
```

