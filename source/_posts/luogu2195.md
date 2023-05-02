---
title: luogu2195 HXY造公园 题解
urls: lg2195-solution
tags: 树的直径
categories: 题解
math: true
abbrlink: 48707b39
date: 2021-09-20 16:32:28
---

我最喜欢的紫色水题（

<!--more-->


给出一个森林，有两种操作。

1. 询问某个点所在的树的直径
2. 在两个点所在的两棵树间连一条边，最小化其直径

显然的，对于第一种操作，DP / DFS / BFS 预处理直径，并查集维护每棵树的点就行了。

问题在于高效维护第二种操作。

不难想到，两棵树之间连一条边，相当于合并两个集合。

而最小化新树的直径，显然要在两树直径的中点处连边。

证明：

>反证法。若最优点不是直径中点，由于 直径有两个端点 且 树上两点有且仅有一条简单路径，若在非直径中点的 $x$ 点连边，则当其接近直径一端时，直径另一端到达它的距离就大于到达直径中点的距离，反之则显然。这与假设不符，故原命题正确。

设从 $x$ 与 $y$ 之间连边，$l(t)$ 为点 $t$ 所在的树的直径。

则新树的直径只有三种可能。

1. $ l(x)$
2. $l(y)$
3. $ \lceil \frac{l(x)}{2} \rceil + \lceil \frac{l(y)}{2} \rceil + 1$

合并后求最大值就行了。

```cpp 
#include<cstdio>
#include<iostream>
using namespace std;
const int N=3e5+6;
int n, m, q, ans, f[N], d[N], c[N];
int cnt, h[N], ver[N<<1], nxt[N<<1];
bool v[N];
int r_() {
    int a=0; char c=getchar();
    for(;!isdigit(c);c=getchar());
    for(;isdigit(c);a=(a<<1)+(a<<3)+(c^48),c=getchar());
    return a;
}
void add(int x,int y) { ver[++cnt]=y, nxt[cnt]=h[x], h[x]=cnt; }
int get(int x) { return x==f[x]? x:f[x]=get(f[x]); }
void dp(int x,int fr) {
    int i, y;
    for(i=h[x];i;i=nxt[i]) if(ver[i]!=fr) {
        y=ver[i], dp(y,x);
        ans=max(ans,d[x]+d[y]+1), d[x]=max(d[x],d[y]+1);
    }
}
void miku(int x) { ans=0, dp(x,0), c[x]=ans; }
int main() {
    n=r_(), m=r_(), q=r_();
    for(int i=1;i<=n;++i) f[i]=i;
    for(int i=1,x,y,z;i<=m;++i) x=r_(), y=r_(), add(x,y), add(y,x), f[get(x)]=get(y);
    for(int i=1;i<=n;++i) if(f[i]==i) miku(i);
    while(q --> 0) {
        int op=r_(), x=r_();
        if(op&1) { printf("%d\n",c[get(x)]); continue; }
        int y=r_();
        x=get(x), y=get(y);
        if(x==y) continue;
        c[x]=max((c[x]+1)/2+(c[y]+1)/2+1,max(c[x],c[y]));
        f[y]=x;
    }
}
```