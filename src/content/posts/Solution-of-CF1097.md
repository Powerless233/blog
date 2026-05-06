---
title: Solution of CF1097
published: 2026-05-06
tags: [Solution, CF]
category: ACM
draft: false

---

# Codeforces Round 1097 Div.2

Link here: https://codeforces.com/contest/2224

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

