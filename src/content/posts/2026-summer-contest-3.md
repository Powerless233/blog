---
title: 2026暑期训练赛3
published: 2026-07-16
tags: [Algorithm]
category: ACM
draft: false
---

# 2026 暑期训练赛 3

这场整体难度偏简单，较难的题会展开，比较简单的就思路带过了。

## Problem A

给定两个字符串 s 和 t。在一次删除中，你可以选择任意一个字符串并删除第一个（即最左边的）字符。删除后，字符串的长度减少 11。如果字符串为空，则不能选择该字符串。

例如：

- 对字符串 "where" 应用一次删除后，得到字符串 "here"，
- 对字符串 "a" 应用一次删除后，得到空字符串 ""。

你需要通过最少的删除次数使两个给定的字符串相等。最终，两个字符串可能都变为空字符串，因此相互相等。在这种情况下，答案显然是初始字符串长度的总和。

编写一个程序，找到使两个给定字符串 s 和 t 相等所需的最小删除次数。s 和 t 的长度不超出 2e5。

### Analysis

等价求最长公共后缀，循环即可。

## Problem B

在宇宙深处的一个神秘星球上，一天由 24 个时刻组成，编号从 0 到 23（本题只考虑整数时刻）。一天中，时刻 6 到 17 是 **白天**，而时刻 0 到 5 以及 18 到 23 是 **夜晚**。时刻 23 之后紧接着是下一天的时刻 0。

小A和小E住在不同的城市，两地可能存在时差。小E的时区比小A晚 d个时刻。也就是说，如果小A的时刻是 t，那么小E的时刻是 (t+d) mod 24。

现在，小A当前时刻是 t0。小A想给小E发一条“晚安早安”的问候，但发消息时，小A必须处于 **白天**，而小E必须处于 **夜晚**。请问小A至少要等多少个时刻才能发送这条消息？注意，可能存在永远无法发送的情况。

### Analysis

基础题，循环 + if 判断即可。

## Problem C

给定一张 n 点 m 边的带权有向图，现有 q 次询问。每次询问给出一个起点 p 和体力值 x，每通过一条边体力值变为 $\lfloor x/w\rfloor$, 其中 w 为边权，每次往任意方向走，求最坏情况下可以通过几条边。

### Analysis

设在某种路径下通过的边权为 $w_1,w_2,...,w_k$，则 $\lfloor x/w_1 \rfloor / w_2 / ... / w_k =  \lfloor x / (w1 \times w2 \times ... \times w_k \rfloor)$。

那么题目等价问从某一点出发，至少经过几条边，其边权积大于 x。

每多一条边，积至少乘以 2，x 是 1e9 则至多走 30 条边。

做图上 dp，$f[i][j]$ 表示从节点 i 出发走 j 条边的最大边权积。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const LL N = 5e5 + 5;
int n, m, q;
vector<pair<int, int>> e[N];
int f[N][32];
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> n >> m >> q;
    for (int i = 1; i <= m; i++)
    {
        int x, y, w;
        cin >> x >> y >> w;
        e[x].push_back({y, w});
    }
    for (int i = 1; i <= n; i++)
    {
        f[i][0] = 1;
    }
    for (int t = 1; t <= 30; t++)
    {
        for (int i = 1; i <= n; i++)
        {
            for (auto [j, w] : e[i])
            {
                f[i][t] = max(f[i][t], f[j][t - 1] * w);
            }
        }
    }
    while (q--)
    {
        int p, x;
        cin >> p >> x;
        for (int i = 1; i <= 30; i++)
        {
            if (f[p][i] > x)
            {
                cout << i << '\n';
                break;
            }
        }
    }
    return 0;
}
```

## Problem E

给定 $n$（$n$ 是偶数）个整数，用 $a_1, a_2, \dots, a_n$ 表示。给定整数 $k$，初始得分为0。操作持续 $\frac{n}{2}$ 回合，在每个回合中依次发生以下事件：

-   小A从序列中选择一个整数并将其擦除。设小A选择的整数为 $a$
-   小B从序列中选择一个整数并将其擦除。设小B选择的整数为 $b$。
-   如果 $a + b = k$，则得分加1。

小A的目标是最小化得分，而小B的目标是最大化得分。假设两个人都采用最优策略，游戏结束后得分是多少？

### Analysis

看似是很难的博弈，实际不管通过直觉还是分类讨论，都可以推出简单结论：答案就等于原题意中符合条件的整数对 (a, b) 的数量。

下面分析为什么：

记符合条件的整数对 (a, b) 的数量 = P。

显然答案不会比 P 更大，下面只需说明可以构造出答案 P 即可。

简单将所有数分为 3 类：

1. 完全孤立的数：没有数可以与其配对。

2. 可配对但多余的数：存在数可以与其配对，但是它的数量大于可配对的数。

   比方说：a + b = k，a 有 3 个，b 有 4 个，那么有 1 个 b 就属于可配对但多余的数。

3. 可配对的数

由于 n 是偶数，可配对的数（第 3 类）有偶数个，那么第 1 类和第 2 类的数也有偶数个。

那么 B 就可以实现以下策略：

1. 当 A 取第 1 / 2 类数时，也取一个第 1 / 2 类数。这些数本来就无法配对。
2. 当 A 取第 3 类数时，取和它配对的数。

所有第 3 类数均可配对，最终答案为 P。

## Problem F

给定一个包含 $n$ 个节点的有向图 $G=(V, E)$，节点编号为 $1$ 到 $n$。每个节点 $i$ 拥有一个初始激活阈值 $a_i$。同时，图中每条有向边 $(u, v)$ 具有一个非负整数权值 $w_{uv}$，表示信号传输的延迟时间。

系统的时间从 $t=0$ 开始流逝，节点的激活（解锁）遵循以下规则：

1.  **初始状态**：在 $t=0$ 时刻，所有节点均处于未激活状态。
2.  **激活条件**：对于任意节点 $i$，设当前时刻为 $t$，该节点累计接收到的“能量”总值为 $E_i(t)$。当且仅当 $E_i(t) \ge a_i$ 时，节点 $i$ 被立即激活。
3.  **能量传播**：一旦节点 $u$ 在时刻 $T_u$ 被激活，它会立即向其所有出边邻居 $v$ 发送 1 点能量。该能量将在 $T_u + w_{uv}$ 时刻到达节点 $v$ 并累加到 $v$ 的总能量中。
4.  **外部干预（刷新器）**：系统提供了 $k$ 次外部干预机会。第 $j$ 次干预由参数 $(t_j, S_j)$ 描述，其中 $S_j$ 是一个节点集合。在时刻 $t_j$，集合 $S_j$ 中所有节点的激活阈值 $a_i$ 将被强制修改为 $0$。这意味着只要这些节点在该时刻接收到任意非零能量（或者如果 $t_j=0$ 且定义 $0 \ge 0$），它们就会立即激活。

3≤*n*≤1e5,0≤*m*≤1e6,0≤*k*≤1e5

### Analysis

如果没有刷新器，那这个东西就非常类似于拓扑排序，只不过队列里存的是边。

开一个优先队列，按照到达节点的时间从早到晚排序。

当某一个节点解锁时，将它所有的边加到队列里。循环至队列中没有元素。

加入刷新器也是比较经典的模型：

我们开一个超级源点，然后以一条时间为 t，权值为 inf 的边连向刷新节点即可。

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const int N = 1e5 + 5, M = 1e6 + 5, inf = 0x3f3f3f3f3f3f3f3f;
int n, m, k, a[N];
int ans[N];
struct Edge
{
    int v, dt, c;
};
vector<Edge> e[N];

struct Node
{
    int v, t, c;
    friend bool operator<(const Node &a, const Node &b)
    {
        return a.t > b.t;
    }
};
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m >> k;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
        ans[i] = -1;
    }
    for (int i = 1; i <= k; i++)
    {
        int t, sc;
        cin >> t >> sc;
        for (int j = 1; j <= sc; j++)
        {
            int x;
            cin >> x;
            e[0].push_back((Edge){x, t, inf});
        }
    }
    for (int i = 1; i <= m; i++)
    {
        int x, y, w;
        cin >> x >> y >> w;
        e[x].push_back((Edge){y, w, 1});
    }
    priority_queue<Node> q;
    for (int i = 0; i <= n; i++)
    {
        if (!a[i])
        {
            ans[i] = 0;
            for (auto [v, dt, dc] : e[i])
            {
                q.push({v, dt, dc});
            }
        }
    }
    while (!q.empty())
    {
        auto [x, t, c] = q.top();
        q.pop();
        if (ans[x] != -1)
            continue;
        a[x] -= c;
        if (a[x] <= 0)
        {
            ans[x] = t;
            for (auto [v, dt, dc] : e[x])
            {
                if (ans[v] == -1)
                {
                    q.push({v, t + dt, dc});
                }
            }
        }
    }
    for (int i = 1; i <= n; i++)
    {
        cout << ans[i] << ' ';
    }
    return 0;
}

```

## Problem G

给定一个有且仅有 5 位，且各个数位互不相同的十进制正整数 $n$。你可以重新排列 $n$ 的各个数位，但需要保证重新排列得到的整数 $n'$ 没有前导零。请问重新排列数位得到的 $n'$ 能否为合数？若能为合数，请求出一个满足条件的 $n'$。

### Analysis

暴力排列组合就能过，复杂度为 $O(Td\times d!)$，其中 d = 5，约为 6e7。

## Problem H

今天，樱子正在研究数组。一个长度为 $n$ 的数组 $a$ 被称为“好数组”，当且仅当满足以下条件：

-   数组 $a$ 是严格递增的，即对于所有 $2 \le i \le n$ 都满足 $a_{i-1} < a_i$；
-   相邻元素的差值也是递增的，即对于所有 $2 \le i < n$ 都满足 $a_i - a_{i-1} < a_{i+1} - a_i$。

樱子已经确定了边界值 $l$ 和 $r$，现在想要构造一个最长的**好数组**，其中每个元素 $a_i$ 都满足 $l \le a_i \le r$。

请帮助樱子找出在给定 $l$ 和 $r$ 的情况下，好数组的最大可能长度。

### Analysis

构造方案显然是相邻元素的差递增，1，2，...。

解方程即可。

## Problem I

一个长度为 $m$ 的序列 $[v_1, \dots, v_m]$ 的 **权值** 定义如下：

1. 如果 $m = 1$，那么权值就是 $v_1$。
2. 如果 $m > 1$，那么权值是序列最后一个数字加上之前元素中的最大值，也就是 $\max_{i=1}^{m-1} v_i + v_m$。

给你一个序列 $a_1, a_2, \dots, a_n$，你想把它分成最多 $k$ 个非空子序列，使得这些子序列权值的和最大化。

这里的“子序列”指的是从原序列中删掉 0 个或多个元素后，剩下元素的相对顺序不变的序列。

分割后，原序列的每个元素必须且只能属于一个子序列。

### Analysis

每个子区间有两种情况：只有 1 个数；至少有 2 个数。

首先最后一个必须取，而且不管是否有分组，1 $\to$ n - 1 中最大的那个数也一定会取。

那么由于第 1 组里面已经包括了 a[n] 和 1 $\to$ n - 1 中最大的那个数，那么无论再添加什么元素都不会改变结果。

那么接下来就在 1 $\to$ n - 1 中取正数就行。每一组只取 1 / 2 个数。

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const int N = 2e5 + 5;
int n, k, ans[N];
struct Node
{
    int x, id;
    friend bool operator<(const Node &_x, const Node &_y)
    {
        return _x.x > _y.x;
    }
} a[N];
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> k;
    for (int i = 1; i <= n; i++)
    {
        int x;
        cin >> x;
        a[i] = {x, i};
        ans[i] = 1;
    }
    sort(a + 1, a + n);
    int tot = 0;
    tot += a[n].x + a[1].x;
    for (int i = 2, j = 2; i <= k && j < n; i++, j += 2)
    {
        if (j < n && a[j].x > 0)
        {
            ans[a[j].id] = i;
            tot += a[j].x;
        }
        if (j + 1 < n && a[j + 1].x > 0)
        {
            ans[a[j + 1].id] = i;
            tot += a[j + 1].x;
        }
    }
    cout << tot << '\n';
    for (int i = 1; i <= n; i++)
    {
        cout << ans[i] << ' ';
    }
    return 0;
}
```

## Problem J

给定一个长度为 $n$ 的字符串 $S$，该字符串仅由小写字母组成，现定义如下概念：

-   **子序列**：从 $S$ 中提取若干元素（不必连续）并保持其相对顺序构成的序列称为子序列。
-   **$k$-宽松子序列**：若字符串 $S$ 的某个子序列中，任意两个相邻字符在原字符串 $S$ 中的位置间隔至少为 $k$，则称该子序列为 $S$ 的 $k$-宽松子序列。具体而言，对于长度为 $m$ 的子序列 $T = \overline{S_{pos_1} S_{pos_2} \cdots S_{pos_m}}$，当且仅当满足 $\forall i \in [1, m-1]$ 和 $pos_{i+1} - pos_i > k$ 时，$T$ 才是 $S$ 的 $k$-宽松子序列。

现给定非负整数 $k$，请计算 $S$ 所有不同的非空 $k$-宽松子序列的数量，并将结果对 $998244353$ 取模后输出。

当且仅当 $|A| \neq |B|$ 或存在某个下标 $i$ 使得 $A_i \neq B_i$ 时，$S$ 的两个子序列 $A$ 和 $B$ 被视为不同。

### Analysis

在经典的统计本质不同子序列问题上，新增了相邻距离大于 k 的规则。

思路和原模型基本一模一样，状态转移方程进行相应修改即可。

### Code

非常搞笑的是我把模数写错了 wa 了 4 发。

```cpp
#include <bits/stdc++.h>
#define int long long
#define LL long long
using namespace std;
const int N = 1e6 + 5, mod = 998244353;
int n, k;
int lst[N];
int tmp[30];
int f[N];
int c[N];
void solve()
{
    memset(tmp, 0, sizeof tmp);
    cin >> n >> k;
    string s;
    cin >> s;
    s = " " + s;
    for (int i = 1; i <= n; i++)
    {
        lst[i] = tmp[s[i] - 'a'];
        tmp[s[i] - 'a'] = i;
        f[i] = c[i] = 0;
    }
    for (int i = 1; i <= n; i++)
    {

        if (lst[i])
        {
            if (i - k - 1 >= 0)
                f[i] = (f[i] + c[i - k - 1]) % mod;

            if (lst[i] - k - 1 >= 0)
                f[i] = (f[i] - c[lst[i] - k - 1] + mod) % mod;
        }
        else
        {
            if (i - k - 1 >= 0)
                f[i] = (f[i] + c[i - k - 1]) % mod;
            f[i] = (f[i] + 1) % mod;
        }
        f[i] = (f[i] % mod + mod) % mod;
        c[i] = (c[i - 1] + f[i]) % mod;
        // cout << f[i] << ' ';
    }
    // cout << '\n';
    cout << c[n] << '\n';
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

## Problem K

输入 a,b,c,d，输出 (a+b+c)*d。

### Analysis

请输入文本。

## Problem M

给出长度为 $n$ 的正整数序列 $\{a_n\}$ 和 $\{b_n\}$。对于每个 $a_i (1 \le i \le n)$，进行恰好一次以下操作：

-   将 $a_i$ 变成满足 $|a_i - x| \le k \times b_i$ 的任意整数 $x$。

请你求出最小的非负整数 $k$，使得存在至少一种方法使得操作后序列 $\{a_n\}$ 所有数都相等。

### Analysis

二分答案板子。

### Code

```cpp
#include <bits/stdc++.h>
#define LL long long
#define int long long
using namespace std;
const int N = 3e5 + 5;
int n;
int a[N], b[N];
bool check(int k)
{
    int L = 0, R = INT_MAX;
    for (int i = 1; i <= n; i++)
    {
        int l = a[i] - k * b[i];
        int r = a[i] + k * b[i];
        L = max(L, l);
        R = min(R, r);
        if (L > R)
            return 0;
    }
    return 1;
}
void solve()
{
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
    }
    for (int i = 1; i <= n; i++)
    {
        cin >> b[i];
    }
    int l = 0, r = INT_MAX, mid;
    while (l < r)
    {
        mid = (l + r) >> 1;
        if (check(mid))
            r = mid;
        else
            l = mid + 1;
    }
    cout << r << '\n';
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

