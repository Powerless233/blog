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
- 更新全局答案：$\text{ans} = \max(\text{ans}, f[v_1] + f[v_2])$

若某一个节点只有一个儿子，仍然可以使用上面的公式更新，因为第二长的子链长度为 0。

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
