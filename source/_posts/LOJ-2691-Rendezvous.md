---
title: LOJ2691 Rendezvous 题解
urls: loj2691-solution
tags:
  - 基环树
  - 最近公共祖先
categories: 题解
math: true
abbrlink: 2106eded
date: 2021-08-21 14:32:39
---

[link](https://loj.ac/p/2691)

给出的是一个内向基环树森林。

很容易想到，如果两个点不在同一颗基环树中，则不存在合法的 $(x,y)$。

<!--more-->

但是怎么处理在同一颗树中的点呢？

我有一个大胆的想法。

无法直接处理 lca，是因为基环树有一个环。

但是如果 $ (a,b)$ 两点在同一颗子树中，且该子树没有环上的边，则不影响求出 $ LCA(a,b)$。
所以，该子树应含不超过1个的环上的点。若含有，则一定是根。

更进一步地说，把每个环上的点看作「根」，它的子树中节点可以直接求 lca，这要我们求出它子树中每个节点的「深度」。

所以，如果 $ (a,b)$ 两点的「根」是同一个节点，那么只需要求出 $c=LCA(a,b)$，答案即为 $dep(a)-dep(c)$ 与 $dep(b)-dep(c)$。

若 $(a,b)$ 的「根」不同，那么该怎么处理呢？

仔细分析不难发现，在合法的解中，一定有它们与「根」的距离，即 $dep(a)-1$ 与 $ dep(b)-1$ ，记为 $ (p,q)$

而剩下的部分，就是它们的「根」在环中互相到达的距离。

我们设 $ ord(x)$ 为环上节点 $x$ 在环上的顺序编号， $ s(x)$ 为基环树 $x$ 环上的节点数量。

设 $(x,y)$ 为  $(a,b)$ 两点分别的「根」，$z$ 为它们所属的基环树编号。

不难发现，有两种情况。

$(x \rightarrow y)$ 距离为 $( ord(y)-ord(x)+s(z)) \bmod s(z)$。

$(y \rightarrow x)$ 距离为 $(ord(x)-ord(y)+s(z)) \bmod s(z)$。

最后检查 $ (p+(x \rightarrow y),q)$ 与 $( p,q+(y \rightarrow x))$，选取合法解。

&nbsp;

那么如何实现呢？

先找环，每个环对应着每颗基环树。

标记环上的点并将其深度置为1，记录它所在的基环树编号、此树环上节点个数、环上节点的编号、将其根标记为自己。

$ c(x)$ 用来判断环，$ d(x)$ 记录深度，$ bel(x)$ 记录所属基环树编号，$root(x)$ 记录环上的「根」。

```cpp
void find_circle(int x) {
    int y=x;
    while(1) {
        if(c[x]==y) break;
        // 找到了环
        if(c[x]) return ;
        // 找到了已经访问过却不是环上节点的点
        // 直接结束，因为这个循环一定找不到环了，不加这句TLE
        c[x]=y, x=to[x];
    }
    ++num;
    while(!root[x]) d[x]=1, ord[x]=++s[num], bel[x]=num, root[x]=x, x=to[x];
}
```

在倍增求 lca 的时候，不断更新 $root(x)$，其他的就是模板。

```cpp
void dfs(int x) {
    if(d[x]) return ;
    dfs(to[x]);
    // 内向树只有一条出边
    f[x][0]=to[x], d[x]=d[to[x]]+1, root[x]=root[to[x]];
    for(int i=1;(1<<i)<d[x];++i) f[x][i]=f[f[x][i-1]][i-1];
    // 转移树上倍增
}
int lca(int x,int y) {
    if(d[x]<d[y]) swap(x,y);
    for(int i=0,j=d[x]-d[y];j;j>>=1,++i) if(j&1) x=f[x][i];
    if(x==y) return x;
    for(int i=lg[d[x]];i>=0;--i) if(f[x][i]!=f[y][i]) x=f[x][i], y=f[y][i];
    return f[x][0]; 
}
```

我这里与处理了一下 $ \log_2x$。

剩下的。

```cpp 
#include<cstdio>
#include<iostream>
using namespace std;
const int N=5e5+10;
int n, k, num, d[N], f[N][20], to[N], lg[N], bel[N];
int root[N], s[N], ord[N], c[N];
bool ck(int x1,int y1,int x2,int y2) {
    int P=max(x1,y1), Q=max(x2,y2), p=min(x1,y1), q=min(x2,y2);
    if(P<Q) return 1;
    if(Q<P) return 0;
    if(p<q) return 1;
    if(q<p) return 0;
    return x1>=y1;
}
int main() {
    // freopen("data_\\ran10c.in","r",stdin);
    // freopen("data_\\qwq.out","w",stdout);
    scanf("%d%d",&n,&k);
    lg[1]=0, lg[2]=1;
    for(int i=3;i<=n;++i) lg[i]=lg[i>>1]+1;
    for(int i=1;i<=n;++i) scanf("%d",&to[i]);
    for(int i=1;i<=n;++i) fc(i);
    for(int i=1;i<=n;++i) dfs(i);
    while(k --> 0) {
        scanf("%d%d",&x,&y);
        if(bel[root[x]]!=bel[root[y]]) { puts("-1 -1"); continue ; }
        if(root[x]==root[y]) z=lca(x,y), printf("%d %d\n",d[x]-d[z],d[y]-d[z]);
        else {
            int p=d[x]-1, q=d[y]-1, d1, d2;
            int z=bel[root[x]], x=root[x], y=root[y];
            int d1=(ord[y]-ord[x]+s[z])%s[z], d2=(ord[x]-ord[y]+s[z])%s[z];
            if(ck(p+d1,q,p,q+d2)) printf("%d %d\n",p+d1,q); else printf("%d %d\n",p,q+d2);
        }
    }
}
```
