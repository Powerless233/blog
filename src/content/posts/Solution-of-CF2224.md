---
title: Solution of CF2224
published: 2026-05-06
tags: [Solution, CF]
category: ACM
draft: false
---

# Codeforces Round 1097 Div.2

这场比较偏向 trick，见过就会比较好做。

## A. Zhily and Array Operating

贪心。从后往前枚举，只要 $a_{i+1}>0$ 就加到 $a_i$，容易证明这样一定会使结果更优。

#### Code

```cpp
    int T = read();
    while (T--)
    {
        int n = read();
        vector<int> a(n + 5);
        for (int i = 1; i <= n; i++)
        {
            a[i] = read();
        }
        int ans = 0;
        if (a[n] > 0)
            ans++;
        for (int i = n - 1; i >= 1; i--)
        {
            if (a[i + 1] > 0)
                a[i] += a[i + 1];
            if (a[i] > 0)
                ans++;
        }
        cout << ans << '\n';
    }
```

## B. Zhily and Mex and Max

贪心，注意处理细节。

第一个放最大的，记为 $a_{max}$，然后开始从小往大填并更新 $\texttt{mex}$。

两个细节：

- 若 $a_{max} = 0$，则要一开始就将 $\texttt{mex} = 1$。
- 若处理到当前有 $\texttt{mex} = a_{max}$ ，则 $\texttt{mex} = a_{max} + 1$，因为已经放在第一个位置了。

#### Code

```cpp
    int T = read();
    while (T--)
    {
        int n = read();
        int mx = 0;
        for (int i = 1; i <= n; i++)
        {
            a[i] = read();
            mx = max(mx, a[i]);
        }
        sort(a + 1, a + 1 + n);
        int tot = unique(a + 1, a + 1 + n) - (a + 1); // 同一值有多个其实没有用，只会先填一个进去
        int cur = 0; // mex
        int ans = mx;
        if (mx == 0)
        {
            cur++;
            ans++;
        }
        for (int i = 1; i < tot; i++)
        {
            if (a[i] == cur)
            {
                cur++;
            }
            if (cur == mx)
            {
                cur++;
            }
            ans += cur + mx;
        }
        ans += (cur + mx) * (n - tot);
        cout << ans << '\n';
    }
```

## C. Zhily and Bracket Swapping

常见的 Ad-hoc。

成立的条件即时刻保持 左括弧数 >= 右括弧数，且结束时 左括弧数 = 右括弧数。

显然只有 $a_i \neq b_i$ 时才有操作的必要。

可以将操作抽象为：在 $a_i \neq b_i$ 的位置把 左括弧 分给 a 还是分给 b。

那么显然要想让两个序列的 左括弧数 都大于等于 0，最好的做法是轮流分，一个 a，一个 b，如此循环。

按照这种分发检验 左括弧数 是否大于等于 0，在结束时检验是否有 左括弧数 = 右括弧数。

#### Code

```cpp
    int T = read();
    int tot = 1;
    while (T--)
    {
        int n = read();
        string s1, s2;
        cin >> s1 >> s2;
        int mid = 0; // 左括弧数 - 右括弧数
        bool ans = 1;
        bool fl = 0;
        for (int i = 0; i < n; i++)
        {
            if (!ans)
                break;
            if (s1[i] == s2[i])
            {
                mid += (s2[i] == '(' ? 1 : -1);
            }
            else
            {
                fl ^= 1;
            }
            int x = mid - fl;
            if (x < 0)
            {
                ans = 0;
            }
        }
        if (mid == 0 && !fl && ans)
        {
            cout << "YES\n";
        }
        else
            cout << "NO\n";
    }
```

## D. Zhily and Barknights

从点对的方式分析。

对于一组 $i < j$ ，若存在一组 $x \neq y$ 使得 $a_i\times b_x < a_j\times b_y$，那么就会产生一个逆序对。

一共存在 $n(n-1)$ 对  $(x,y)$。这个逆序对对于期望的贡献为 $\frac{1}{n(n-1)}$。

问题相当于求 $(i,j,x,y)$ 满足 $a_i\times b_x < a_j\times b_y$ 的个数。

$a_i\times b_x < a_j\times b_y \Rightarrow \frac{a_i}{a_j} < \frac{b_x}{b_y}$

将 $\frac{a_i}{a_j}$ 和 $\frac{b_x}{b_y}$ 分别排序，然后跑一遍双指针即可。

```cpp
    int T = read();
    while (T--)
    {
        int n = read();
        vector<int> a(n + 5);
        vector<int> b(n + 5);
        for (int i = 1; i <= n; i++)
        {
            a[i] = read();
        }
        for (int i = 1; i <= n; i++)
        {
            b[i] = read();
        }
        vector<pair<int, int>> f;
        vector<pair<int, int>> g;
        for (int i = 1; i <= n; i++)
        {
            for (int j = 1; j <= n; j++)
            {
                if (i == j)
                    continue;
                f.push_back(make_pair(b[i], b[j]));
            }
        }
        for (int i = 1; i < n; i++)
        {
            for (int j = i + 1; j <= n; j++)
            {
                g.push_back(make_pair(a[i], a[j]));
            }
        }
        sort(f.begin(), f.end(), cmp);
        sort(g.begin(), g.end(), cmp);
        int j = 0;
        int cnt = 0;
        for (int i = 0; i < g.size(); i++)
        {
            while (j < f.size() && cmp(f[j], g[i]))
                j++;
            cnt += j;
            cnt %= mod;
        }
        int ans = cnt * inv(n * (n - 1) % mod) % mod;
        cout << ans << '\n';
    }
    return 0;
```

## E. Zhily and Signpost

首先观察到这个关键性质：

设每个点的子节点个数为 si[x]。从根节点往下走，若当前节点 x 的子节点个数 si[x] 可以被之前路径上所有 si[i] 的 lcm 整除，那么必定只会走向同一个子节点。

比方说：之前经过了一个节点，当时的时间满足 t mod 4 = 1。

然后来到了一个节点，这个节点有 2 个子节点。因为前面的时间 t 模 4 已经唯一了，那么 (t + l[x]) mod 4 同样是唯一的， mod 2 自然也是唯一的。

那么所有走到这个节点的询问，都只会走向同一个子节点。

------

因为我们有询问 m < 1e18，因此当 mod > 1e18 时，一定只会存在一个询问。

我们从根节点往下 dfs，通过 move 函数将询问往下分配到子节点上。

当且仅当 gcd(mod, si[x]) != 1 时，询问会分散到不同子节点上，而每一次进行这样的操作，都会使 mod 至少 x2。

因此时间复杂度为 $O(n\log_2M)$，其中 n = 5e5, M = 1e18。

#### Code

```cpp
#include <bits/stdc++.h>
#define int long long
#define LL long long
#define pii pair<int, int>
using namespace std;
inline LL read()
{
    LL res = 0, fl = 1;
    char ch = getchar();
    while (!(ch >= '0' && ch <= '9'))
    {
        if (ch == '-')
            fl = -1;
        ch = getchar();
    }
    while (ch >= '0' && ch <= '9')
        res = (res << 3) + (res << 1) + ch - '0', ch = getchar();
    return res * fl;
}
inline LL max(LL a, LL b) { return a > b ? a : b; }
inline LL min(LL a, LL b) { return a < b ? a : b; }
const LL N = 1e5 + 5, inf = 0x3f3f3f3f;
void solve()
{
    int n = read(), q = read();
    vector<int> f(n + 5);
    vector<int> l(n + 5);
    vector<vector<int>> e(n + 5);
    for (int i = 2; i <= n; i++)
    {
        f[i] = read();
        e[f[i]].push_back(i);
    }
    for (int i = 2; i <= n; i++)
    {
        l[i] = read();
    }
    vector<pii> qr(q + 5);
    for (int i = 1; i <= q; i++)
    {
        qr[i].first = read();
        qr[i].second = i;
    }
    sort(qr.begin() + 1, qr.begin() + 1 + q);
    vector<int> qv;          // 存的是询问的具体值
    vector<vector<int>> qid; // 存的是询问的 id

    for (int i = 1; i <= q; i++)
    {
        qv.push_back(qr[i].first);
        qid.push_back({});
        qid.back().push_back(qr[i].second);
        while (i + 1 <= q && qr[i + 1].first == qr[i].first)
        {
            i++;
            qid.back().push_back(qr[i].second);
        }
    }
    int nq = qv.size();
    vector<int> qa(nq); // 存的是离散后的询问组的 id
    vector<int> ans(q + 5);
    for (int i = 0; i < nq; i++)
        qa[i] = i;

    auto lcm = [&](LL a, LL b) -> LL
    {
        return a * b / __gcd(a, b);
    };

    auto dfs = [&](auto &&self, int x, int f, int t, int mod, vector<int> a) -> void
    {
        if (!e[x].size())
        {
            for (auto i : a)
            {
                for (auto j : qid[i])
                {
                    ans[j] = x;
                }
            }
            return;
        }
        if (!a.size())
            return;

        int xn = e[x].size();
        if (a.size() == 1 || mod == -1 || mod % xn == 0)
        {
            int re = (qv[a[0]] + t) % xn;
            int y = e[x][re];
            self(self, y, x, t + l[y], mod, move(a));
        }
        else
        {
            vector<vector<int>> b(xn);
            for (auto i : a)
            {
                int re = (t + qv[i]) % xn;
                b[re].push_back(i);
            }
            int nmod = lcm(mod, xn);
            if (nmod > 1e18)
                nmod = -1;
            for (int i = 0; i < xn; i++)
            {
                if (b[i].size())
                {
                    int y = e[x][i];
                    self(self, y, x, t + l[y], nmod, move(b[i]));
                }
            }
        }
    };

    dfs(dfs, 1, 0, 0, 1, qa);
    for (int i = 1; i <= q; i++)
    {
        cout << ans[i] << ' ';
    }
    cout << '\n';
}
signed main()
{
#ifndef ONLINE_JUDGE
    freopen("test.in", "r", stdin);
    freopen("test.out", "w", stdout);
#endif
    int T = read();
    while (T--)
    {
        solve();
    }
    return 0;
}
```

第一次写这种 lambda 体的递归函数，感觉挺好用的，解决了数组全局定义和多测初始化的问题。
