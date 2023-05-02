---
title: luogu1967 货车运输 题解
urls: lg1967-solution
tags:
  - 生成树
  - 最近公共祖先
  - 倍增
categories: 题解
math: true
abbrlink: ec7efb1
date: 2021-10-10 21:46:18
---

题目要求不超过限重，不难想到因该最大化每条路的限重。所以在原图上求出最大生成树。

对于点 $(x,y)$，如果在并查集中 $x$ 与 $y$ 不在同一个集合，则 $x$ 不能到达 $y$

<!--more-->

接下来就是每辆车最多运送的货物，不难想到最多运送的货物就是 $ (x \rightarrow y)$ 路径上权值最小的边。

如果用朴素的算法去求最小的边权，那么复杂度会上天，$O(n)$。

联系我们对求 LCA 的倍增优化，可以对求路径上最小的边权进行倍增优化。

设 $d(x,k)$ 为节点 x$ 到它的 $2^k$ 辈祖先这条路径上最小的边权。

接着不难想到转移
$$
d(x,k)=\min_{k \le \log_2n}{ \{ d(x,k-1),d(f(x,k-1),k-1) \} }
$$
可以在求 $ f$ 数组的同时求出。

所以，在求 LCA 的过程中不断维护路径上最小的 $ d(x,k)$。

这题毒瘤数据，给出的图不一定联通。

```cpp
#include<bits/stdc++.h>
using namespace std;
#define R register
const int N=1e5+10, M=5e5+5;
int n, m, q, t, o;
int fr[N], f[N][20], d[N][20], dep[N];
int cnt, h[N], ver[N<<1], nxt[N<<1], g[N<<1];
struct pt { int u, v, w; } a[M];
void add(int x,int y,int z) { ver[++cnt]=y, g[cnt]=z, nxt[cnt]=h[x], h[x]=cnt; }
int get(int x) { return x==fr[x]? x:fr[x]=get(fr[x]); }
bool cmp(pt a,pt b) { return a.w>b.w; }
void kruskal() {
    sort(a+1,a+m+1,cmp);
    R int i, x, y;
    for(i=1;i<=m;++i) {
        x=get(a[i].u), y=get(a[i].v);
        if(x!=y) {
            fr[x]=y;
            add(a[i].u,a[i].v,a[i].w), add(a[i].v,a[i].u,a[i].w);
        }
    }
}
void dfs(int x,int pre) {
    R int i, y;
    dep[x]=dep[pre]+1;
    for(i=1;i<=17;++i) {
        f[x][i]=f[f[x][i-1]][i-1];
        d[x][i]=min(d[x][i-1],d[f[x][i-1]][i-1]);
    }
    for(i=h[x];i;i=nxt[i]) if(ver[i]!=pre) {
        y=ver[i];
        f[y][0]=x, d[y][0]=g[i], dfs(y,x);
    }
}
int lca(int x,int y) {
    R int i, res=1<<30;
    if(get(x)!=get(y)) return -1;
    if(dep[x]<dep[y]) swap(x,y);
    for(i=17;i>=0;--i) {
        if(dep[f[x][i]]>=dep[y]) res=min(res,d[x][i]), x=f[x][i];
        if(x==y) return res;
    }
    for(i=17;i>=0;--i) if(f[x][i]!=f[y][i]) {
        res=min(min(res,d[x][i]),d[y][i]);
        x=f[x][i], y=f[y][i];
    }
    return min(min(res,d[x][0]),d[y][0]);
}
int main() {
    R int i, x, y;
    memset(d,0x3f,sizeof(d));
    scanf("%d%d",&n,&m);
    for(i=1;i<=n;++i) fr[i]=i;
    for(i=1;i<=m;++i) scanf("%d%d%d",&a[i].u,&a[i].v,&a[i].w);
    kruskal();
    for(i=1;i<=n;++i) if(!dep[i]) dfs(i,0);
    scanf("%d",&q);
    while(q--) {
        scanf("%d%d",&x,&y);
        printf("%d\n",lca(x,y));
    }
}
```

