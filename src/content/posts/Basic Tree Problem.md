---
title: 基础树上问题
published: 2026-01-26
tags: [Algorithm]
category: ACM
draft: false
---

# 基础树上问题

树是一种无环连通无向图，具有 $n$ 个节点和 $n-1$ 条边，任意两点之间有且仅有一条简单路径。

本文将讨论的问题较为基础，分别为：**树的遍历**，**树的直径与重心**，**最近公共祖先（LCA）**。

这些问题虽然理解起来都难度不大，但是是后续算法与问题的基石，还请务必自行实现一次。

> 后面预计会补上**树形dp**，**树链剖分**，**点分治与边分治**。如果写好了会把链接贴在这里。

## 1.树的遍历

比较基础，基本就是 dfs（深度优先搜索）的基本操作。基础的问题有：**统计子树大小、求深度**等。

### T1: [P5908 猫猫和企鹅 - 洛谷](https://www.luogu.com.cn/problem/P5908)

题意记为输入一棵树，求深度不大于 $d$ 的节点数量（除去根）。

dfs 的过程中多记一个当前节点深度就行，当前节点深度超了，下面的肯定更没戏，直接 return 就行了。

最后答案减一是因为自己不算。

#### Code

```cpp
int n, d;
int ans = 0;
vector<int> e[N];
void dfs(int x, int fa, int dep)
{
    if (dep > d)
        return;
    ans++;
    for (int i = 0; i < e[x].size(); i++)
    {
        int y = e[x][i];
        if (y != fa)
        {
            dfs(y, x, dep + 1);
        }
    }
}
int main()
{
    n = read(), d = read();
    for (int i = 1; i < n; i++)
    {
        int x = read(), y = read();
        e[x].push_back(y);
        e[y].push_back(x);
    }
    dfs(1, 0, 0);
    cout << ans - 1 << '\n';
    return 0;
}
```

在这里补充一个语法：

在 dfs 遍历树的过程中，以下两种写法的效果是相同的：

```cpp
void dfs(int x, int fa)
{
    for (int i = 0; i < e[x].size(); i++)
    {
        int y = e[x][i];
        if (y != fa)
        {
            ......
        }
    }
}

void dfs(int x, int fa)
{
    for (int y: e[x])
    {
        if (y != fa)
        {
            ......
        }
    }
}
```

其中的第二种写法是 C++11 引入的范围 for 循环，后文不再过多解释。

## 2.树的直径 & 重心

树的直径是指**树中任意两点之间路径长度的最大值**。

- 直径可能不唯一，但长度唯一。

### T2: [B4016 树的直径 - 洛谷](https://www.luogu.com.cn/problem/B4016)

#### Sol.1 两次 dfs/bfs

1. 任选一个点 $u$；
2. 从 $u$ 出发 dfs/bfs 找到离它最远的点 $v$；
3. 从 $v$ 出发 dfs/bfs 找到离它最远的点 $w$；
4. 则 $v$ 到 $w$ 的路径就是一条直径，其长度即为所求。

> 原理：第一次找到的 $v$ 一定是直径的一个端点。

#### Code

```cpp
int n;
int maxdep = 0, rt, ans = 0;
vector<int> e[N];
void dfs1(int x, int fa, int dep)
{
    if (dep > maxdep)
        rt = x, maxdep = dep;
    for (int i = 0; i < e[x].size(); i++)
    {
        int y = e[x][i];
        if (y != fa)
        {
            dfs1(y, x, dep + 1);
        }
    }
}
void dfs2(int x, int fa, int dep)
{
    if (dep > ans)
        ans = dep;
    for (int i = 0; i < e[x].size(); i++)
    {
        int y = e[x][i];
        if (y != fa)
        {
            dfs2(y, x, dep + 1);
        }
    }
}
int main()
{
    n = read();
    for (int i = 1; i < n; i++)
    {
        int x = read(), y = read();
        e[x].push_back(y);
        e[y].push_back(x);
    }
    dfs1(1, 0, 0);
    dfs2(rt, 0, 0);
    cout << ans << '\n';
    return 0;
}
```

#### Sol.2 树形 dp

设 $f[u]$ 表示以 $u$ 为根的子树中，从 $u$ 向下走的最长路径长度。  
在 dfs 过程中，对每个节点 $u$：

- 维护其子节点中最大的两个 $f[v_1] + 1, f[v_2] + 1$；
- 更新全局答案：$\text{ans} = \max(\text{ans}, (f[v_1] + 1) + (f[v_2] + 1))$

若某一个节点只有一个儿子，仍然可以使用上面的公式更新，因为第二长的 $f[v_2] + 1$ 为 0。

#### Code

```cpp
int n, f[N], ans = 0;
vector<int> e[N];
void dfs(int x, int fa)
{
    int mx = 0, se = 0;
    for (int i = 0; i < e[x].size(); i++)
    {
        int y = e[x][i];
        if (y != fa)
        {
            dfs(y, x);
            f[x] = max(f[x], f[y] + 1);
            if (f[y] + 1 > mx)
                se = mx, mx = f[y] + 1;
            else if (f[y] > se)
                se = f[y] + 1;
        }
    }
    ans = max(ans, mx + se);
}
int main()
{
    n = read();
    for (int i = 1; i < n; i++)
    {
        int x = read(), y = read();
        e[x].push_back(y);
        e[y].push_back(x);
    }
    dfs(1, 0);
    cout << ans << '\n';
    return 0;
}
```

当然，我们这里只讨论了边长为 1 的情况，对于边长不同的情况，原理是一样的。

---

### 树的重心

树的重心是树中的一个（或两个）节点，**删除该节点后，使得剩余各个连通块中最大子树的节点数最小**。

更形式化地说：
对于节点 $u$，令 $max\_subtree(u) = \max\{ size[v] \} \cup \{ n - size[u] \}$，
其中 $size[u]$ 是以 $u$ 为根的子树大小， $v$ 是 $u$ 的子节点。
重心即为使 $max\_subtree(u)$ 最小的节点。

#### 性质
1. 一棵树最多有两个重心，且这两个重心一定相邻（仅当 $n$ 为偶数且存在一条边分割成两个大小为 $n/2$ 的部分时）。
2. 重心的所有子树大小都不超过 $\lfloor n/2 \rfloor$。

### T3: [P1395 会议 - 洛谷](https://www.luogu.com.cn/problem/P1395)

找会议节点即为**找一个使所有节点深度之和最小的根节点**。

假设我们已经确定了一个点 $u$ 为根，并且计算出了此时的总代价为 $c$。

我们记与 $u$ 连边的其中一个点为 $v$，即 $u$ 与 $v$ 之间有连边。

记 $v$ 的子树大小为 $size[v]$，粗略地画一个图：

```
        u
       / \
      v   (n-size[v]-1)
     /
  (size[v]-1)
```

现在我们把会议地点从 $u$ 转移到 $v$，我们来看看这会使总代价如何变化。

$size[v]-1$ 个点**少**走了 1 步，$v$ **少**走了 1 步；

$u$ **多**走了 1 步，$n-size[v]-1$ 个点多走了一步。

总代价增加了 $n-2\times size[v]$。

这也就是意味着，当 $size[v]>\lfloor n/2 \rfloor$ 时，总代价将会减小。

反之，当 $size[v]\leq\lfloor n/2 \rfloor$ 时，无论怎样转移根节点都不会使代价增加，那么这个点就是最优解。

根据定义，这个点是重心。

> 竞赛中常采用这样的策略：**先“钦定”某个解（或状态），再考虑如何转移或调整它，并判断是否能得到更优解**。

#### Code

```cpp
int n, si[N];
int minsize = inf, id, tot = 0;
vector<int> e[N];
void dfs1(int x, int fa)
{
    si[x] = 1;
    for (int y : e[x])
    {
        if (y != fa)
        {
            dfs1(y, x);
            si[x] += si[y];
        }
    }
}
void dfs2(int x, int fa)
{
    int maxcur = n - si[x];
    for (int y : e[x])
    {
        if (y != fa)
        {
            dfs2(y, x);
            maxcur = max(maxcur, si[y]);
        }
    }
    if (maxcur == minsize)
        id = min(id, x);
    if (maxcur < minsize)
        minsize = maxcur, id = x;
}
void dfs3(int x, int fa, int dep)
{
    tot += dep;
    for (int y : e[x])
    {
        if (y != fa)
        {
            dfs3(y, x, dep + 1);
        }
    }
}
int main()
{
    n = read();
    for (int i = 1; i < n; i++)
    {
        int x = read(), y = read();
        e[x].push_back(y);
        e[y].push_back(x);
    }
    dfs1(1, 0);
    dfs2(1, 0);
    dfs3(id, 0, 0);
    cout << id << ' ' << tot << '\n';
    return 0;
}
```

## 3.最近公共祖先（LCA）

在一棵**有根树**中，给定两个节点 $ u $ 和 $ v $，它们的 **最近公共祖先（LCA）** 是指在从根到 $ u $ 和从根到 $ v $ 的两条路径上，**深度最大**（即离 $ u, v $ 最近）的**公共节点**。

#### 举例：

```
        1
       / \
      2   3
     / \
    4   5
```

- LCA(4, 5) = 2  
- LCA(4, 3) = 1  
- LCA(4, 4) = 4（自己和自己的 LCA 是自己）

解决 LCA 问题的方法有非常的多，在此处介绍 **倍增法** 与 **Tarjan**，剩下的就咕咕咕了。

#### Sol.0 暴力

作为引入，我们先介绍一个非常暴力的方法。

```
        1
       / \
      2   3
     / \   \
    4   5   6
   /
  7
```

在上图中，我们想找到 6 和 7 的 LCA。

注意到 7 的深度比 6 要大，显然它们的 LCA 不可能在两个点的深度之间，所以我们要 **“往上跳”**，跳到它的父节点，然后一直循环直到两个点的深度一致。

现在我们要求的 LCA(6, 7) 就转化为了 LCA(6, 4)。

这个时候，我们就让这两个点 **同时** “向上跳”。

LCA(6, 4) $\rightarrow$ LCA(3, 2)

LCA(3, 2) $\rightarrow$ LCA(1, 1)

最后我们发现 LCA(6, 7) = LCA(1, 1) = 1。

---

这个方法非常的直观，但同时也非常的暴力，复杂度来到了单次查询 $O(n)$。

#### Sol.1 倍增法

在同时向上跳的过程中，如果我们用 0 表示“未找到LCA”，用 1 表示“已找到LCA”，那么这个过程的序列将会是

```
000...0111...
```

注意到这个**答案具有单调性**。换句话说，随着答案的增大，问题的可行性呈现**单调变化**，向上跳的越多，越会可能是公共祖先。

可能会想到二分，但我们没有办法在很快的时间获取到某一个节点的第 $k$ 个父节点。因此引入了倍增。

我们记录每个节点的 $1,2,4, ...,2^k$ 个父节点，这样一来，就可以在同时向上跳的过程中，从高位向地位逐次进行，只要 $x$ 和 $y$ 的 $2^k$ 个父节点不相同，那么就同时向上跳 $2^k$ 次，最后， $x$ 和 $y$ 会跳到恰好为它们 LCA 的子节点的位置。

在第一步使 $x$ 与 $y$ 深度一致时，也可以通过将 $dep_x - dep_y$ 的值拆分为二的幂次来优化。

时间复杂度预处理 $O(n\log n)$，单次查询 $O(\log n)$。

#### Code

```cpp
int n, m, rt, fa[N][25], dep[N];
vector<int> e[N];
void dfs(int x, int f, int d)
{
    dep[x] = d;
    fa[x][0] = f;
    for (int y : e[x])
    {
        if (y != f)
        {
            dfs(y, x, d + 1);
        }
    }
}
int LCA(int x, int y)
{
    if (dep[x] < dep[y])
        swap(x, y);
    int t = dep[x] - dep[y];
    for (int i = 0; i <= 20; i++)
    {
        if (t & (1 << i))
            x = fa[x][i];
    }
    if (x == y)
        return x;
    for (int i = 20; i >= 0; i--)
    {
        if (fa[x][i] != fa[y][i])
            x = fa[x][i], y = fa[y][i];
    }
    return fa[x][0];
}
int main()
{
    n = read(), m = read(), rt = read();
    for (int i = 1; i < n; i++)
    {
        int x = read(), y = read();
        e[x].push_back(y);
        e[y].push_back(x);
    }
    dfs(rt, 0, 0);
    for (int j = 1; j <= 20; j++)
        for (int i = 1; i <= n; i++)
            fa[i][j] = fa[fa[i][j - 1]][j - 1];

    for (int i = 1; i <= m; i++)
    {
        int x = read(), y = read();
        cout << LCA(x, y) << '\n';
    }
    return 0;
}
```

#### Sol.2 Tarjan

Tarjan 的 LCA 算法是**离线算法**，必须预先知道所有查询，不能动态处理。主要利用了 dfs + 并查集，感觉非常巧妙。

这个感觉用自然语言分析过程太抽象了，还是弄一个例子。

```
        1
       / \
      2   3
     / \   \
    4   5   6
   /
  7
```

依旧是这个图，我们要找 LCA(6, 7)。

首先开一个并查集，约定 $\texttt{find(x)}$ 为并查集搜索函数，$\texttt{bind(x, y)}$ 为并查集合并函数，并查集数组为 $\texttt{p[x]}$。

开一个 $\texttt{vis[x]}$ 数组，表示是否访问过这个节点。

首先访问节点 $1$ ，标记 $\texttt{vis[1]=1}$。

依次访问节点 $2, 4$，标记 $\texttt{vis[2]=1, vis[4]=1}$。

访问节点 $7$，标记 $\texttt{vis[7]=1}$，注意到有一个询问 LCA(6, 7)，但发现 $\texttt{vis[6]=0}$，不作处理。

返回节点 $4$，$\texttt{bind(7, 4)}$，注意这里应当为 $\texttt{p[7]=4}$，也就是并查集总是向深度较低的节点合并。

依次返回节点 $2, 1$，$\texttt{bind(4, 2), bind(2, 1)}$。

访问节点 $3$ ，标记 $\texttt{vis[3]=1}$。

访问节点 $6$ ，标记 $\texttt{vis[6]=1}$。注意到有一个询问 LCA(6, 7)，发现 $\texttt{vis[7]=1}$，处理询问。

此时的 $LCA(6, 7) = \texttt{find(7) = 1}$。

---

这个算法巧妙的点就在：**将并查集的逻辑树 与 原树的层次结构 建立了联系**，将 LCA 的查询转化为对并查集根节点的查询。

可以尝试想象在并查集合并过程中，一颗子树中的所有节点都指向这颗子树的根节点，查询的时候相当于在这颗并查集逻辑树上找根。

时间复杂度**近似线性**： $ O(n + q \cdot \alpha(n)) $，其中 $\alpha(x)$ 为反阿克曼函数。

#### Code

```cpp
int n, m, rt;
int id[N], tot = 0;
int ans[N];
bool vis[N];
vector<int> e[N];
struct Question
{
    int y, id;
};
vector<Question> q[N];
int p[N];
int find(int x)
{
    if (p[x] == x)
        return x;
    return p[x] = find(p[x]);
}
void Bind(int x, int y)
{
    p[find(x)] = find(y);
}
void Tarjan(int x)
{
    vis[x] = 1;
    for (int i = 0; i < e[x].size(); i++)
    {
        int y = e[x][i];
        if (!vis[y])
        {
            Tarjan(y);
            Bind(y, x); // 朝根节点的方向合并
        }
    }
    for (int i = 0; i < q[x].size(); i++)
    {
        int y, id;
        y = q[x][i].y, id = q[x][i].id;
        if (vis[y])
            ans[id] = find(y);
    }
}
int main()
{
    n = read(), m = read(), rt = read();
    for (int i = 1; i <= n; i++)
        p[i] = i;
    for (int i = 1; i < n; i++)
    {
        int x, y;
        x = read(), y = read();
        e[x].push_back(y);
        e[y].push_back(x);
    }
    for (int i = 1; i <= m; i++)
    {
        int x, y;
        x = read(), y = read();
        if (x == y)
            ans[i] = x;
        else
        {
            q[x].push_back({y, i});
            q[y].push_back({x, i});
        }
    }
    Tarjan(rt);
    for (int i = 1; i <= m; i++)
    {
        cout << ans[i] << '\n';
    }
    return 0;
}
```
