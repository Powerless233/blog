---
title: Solution of ABC442
published: 2026-01-27
tags: [Solution, ABC]
category: ACM
draft: false
---

# Solution of ABC442

Contest Link: https://atcoder.jp/contests/abc442

## **D - Swap and Range Sum**

维护数列前缀和，注意到每次交换 $a_i,a_{i+1}$ 只会改变前缀和 $c_i$ 的值，所以可以做到 $O(1)$ 修改和查询。

```cpp
	n = read();
    int q = read();
    for (int i = 1; i <= n; i++)
    {
        a[i] = read();
        c[i] = c[i - 1] + a[i];
    }
    for (int i = 1; i <= q; i++)
    {
        int opt, x, l, r;
        opt = read();
        if (opt == 1)
        {
            x = read();
            c[x] = c[x] + a[x + 1] - a[x];
            swap(a[x], a[x + 1]);
        }
        else
        {
            l = read(), r = read();
            cout << c[r] - c[l - 1] << '\n';
        }
    }
```

## **E - Laser Takahashi**

赛时使用了 STL 中将坐标转换为辅角的函数 $\texttt{atan2(float y, float x)}$ ，但很不幸地被卡精度 WA 了。

既然不能偷鸡，就只能老老实实写一个比较函数按照顺时针顺序排序。

一个比较好想（但写起来比较垃圾）的写法：将四个想象和四条坐标轴按顺时针编号 1~8，排序时先按照这个编号排序，若编号相同（即在同一象限 / 坐标轴）则按照斜率比较。

同一根线上会有多个点，所以再手写一个去重。

```	cpp
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
    n = read(), q = read();
    for (int i = 1; i <= n; i++)
    {
        int x = read(), y = read();
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
        int a = read(), b = read();
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

## **F - Diagonal Separation 2**

如果按照行来考虑问题，会发现每一行的状态只和黑 / 白的分界点位置有关。

令 $f_{i,j}$ 表示考虑了前 $i$ 行，第 $1 \rightarrow j$ 为白，第 $j+1 \rightarrow n$ 为黑时的最小代价。

则有 $ f_{i,j} = cost(i,j) + \min\limits_{j \leq k \leq n}{f_{i-1},k} $ ，其中 $cost(i,j)$ 表示将第 $i$ 行转化为上述状态时的代价。

暴力求解是 $O(n^3)$ 的，但我们可以通过记录 $f_{i-1}$ 的前缀最小值来避免枚举 $k$，这样就达到了 $O(n^2)$。

代码是从 $n \rightarrow 1$ 枚举的，不过本质没有什么区别。

```cpp
	for (int j = 0; j <= n; j++) 
    {
        f[n][j] = cost(n, j);
        if (j)
            g[n][j] = min(g[n][j - 1], f[n][j]);
        else
            g[n][j] = f[n][j];
    }
    for (int i = n - 1; i >= 1; i--)
    {
        for (int cur = 0; cur <= n; cur++)
        {
            f[i][cur] = min(f[i][cur], g[i + 1][cur] + cost(i, cur));
            if (cur)
                g[i][cur] = min(g[i][cur - 1], f[i][cur]);
            else
                g[i][cur] = f[i][cur];
        }
    }
    int ans = inf;
    for (int j = 0; j <= n; j++)
    {
        ans = min(ans, f[1][j]);
    }
    cout << ans << '\n';
```

