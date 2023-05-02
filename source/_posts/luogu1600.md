---
title: luogu1600 天天爱跑步 题解
urls: lg1600-solution
tags: 树上差分
categories: 题解
math: true
abbrlink: 39511a39
date: 2021-10-02 14:57:10
---

经典树上差分。

<!--more-->

考虑能观察到玩家的条件，不难发现，对于每个观察员 $x$，能观察到玩家 $i$，当且仅当满足

1. $ d(s_i)-d(x)=w_x$
2. $ d(s_i)+d(x)-2 \times d(lca(s_i,t_i))=w_x$

上述两式可化为
$$
d(s_i)=w_x+d(x) 
$$
$$
d(s_i)-2 \times d(lca(s_i,t_i))=w_x-d(x)
$$



观察员可以看作点。

对于一条 $(s_i \rightarrow t_i)$ 的路径，由于玩家会经过路径上的每一个点，等号左边是个定值，所以问题可以转化为

> 对于每条路径，将路径上每一个点都加上两个权值为等号左边的物品。询问每个点权值等于等号右边的物品的个数，两种物品分别计数。

&nbsp;

显然是树上差分。

对于每个玩家 $ (x,y)$，求出 $ z=lca(x,y)$，令
$$
x+d(x), \, father(z)-d(x) 
$$
$$
 y+d(x)+2 \times d(z), \, z-(d(x)+2 \times  d(z))
$$



然后考虑计数。

用 $a1,a2$ 记录两种加法操作，用 $ b1,b2$ 记录两种减法操作。

用值域数组 $c1,c2$ ，分别记录两种操作的物品个数。

则要对应着差分来计数，对于每个点的操作，令对应位置加 1 或减 1。

设计数前  $p=c1(d(x)+w_x), \, q=c2(w_x-d(x))$。

计数后集合得到答案
$$
 ans(x)=c1(d(x)+w_x)-p+c2(w_x-d(x))-q
$$
注意 $ w_x-d(x)$ 可能为负，平移一下下标就好了。

```cpp 
#include<cstdio>
#include<iostream>
#include<vector>
using namespace std;
#define pb push_back
#define sz size
const int N=3e5+10;
int n, m, d[N], f[N][18], w[N], c1[N<<1], c2[N<<1], ans[N];
bool v[N];
int cnt, h[N], ver[N<<1], nxt[N<<1];
vector<int> a1[N], a2[N], b1[N], b2[N];
void add(int x,int y) { ver[++cnt]=y, nxt[cnt]=h[x], h[x]=cnt; }
void dfs(int x,int fr) {
    int i, y;
    d[x]=d[fr]+1, f[x][0]=fr;
    for(i=1;(1<<i)<d[x];++i) f[x][i]=f[f[x][i-1]][i-1];
    for(i=h[x];i;i=nxt[i])
        if(ver[i]!=fr) y=ver[i], dfs(y,x);
}
int lca(int x,int y) {
    int i, j;
    if(d[x]<d[y]) swap(x,y);
    for(i=0,j=d[x]-d[y];j;++i,j>>=1) if(j&1) x=f[x][i];
    if(x==y) return x;
    for(i=17;i>=0;--i) if(f[x][i]!=f[y][i]) x=f[x][i], y=f[y][i];
    return f[x][0];
}
void kawaii(int x) {
    int i, y, p=c1[w[x]+d[x]], q=c2[w[x]-d[x]+n];
    v[x]=1;
    for(i=h[x];i;i=nxt[i]) if(!v[ver[i]]) y=ver[i], kawaii(y);
    for(i=0;i<a1[x].sz();++i) ++c1[a1[x][i]];
    for(i=0;i<b1[x].sz();++i) --c1[b1[x][i]];
    for(i=0;i<a2[x].sz();++i) ++c2[a2[x][i]+n];
    for(i=0;i<b2[x].sz();++i) --c2[b2[x][i]+n];
    ans[x]+=c1[d[x]+w[x]]-p+c2[w[x]-d[x]+n]-q;
}
signed main() {
    int i, x, y, z;
    scanf("%d%d",&n,&m);
    for(i=1;i<n;++i) {
        scanf("%d%d",&x,&y);
        add(x,y), add(y,x);
    }
    for(i=1;i<=n;++i) scanf("%d",&w[i]);
    dfs(1,0);
    while(m--) {
        scanf("%d%d",&x,&y), z=lca(x,y);
        a1[x].pb(d[x]), b1[f[z][0]].pb(d[x]);
        a2[y].pb(d[x]-2*d[z]), b2[z].pb(d[x]-2*d[z]);
    }
    kawaii(1);
    for(i=1;i<=n;++i) printf("%d ",ans[i]);
}
```
