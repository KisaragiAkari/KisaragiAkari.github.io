---
title: luogu3349 小星星 题解
urls: lg3349-solution
tags:
  - DP
  - 树形DP
  - 状态压缩
  - 容斥原理
categories: 题解
math: true
abbrlink: a9ee8bb4
date: 2021-10-16 19:38:19
---

设 $f_{x,i}$ 为以 $x$ 为根的子树中，节点 $x$ 映射到图中节点 $i$ 的方案数。

转移
$$
f_{x,i} = \prod_{y \in son(x)} \sum_{j \in G}{f_{y,j}}
$$
答案为 $\sum_{i \in [1,n]} f_{1,i}$

复杂度是 $O(3^n n)$ 的，考虑优化。

题目给出了两条限制

1. 每条树边都要在图中出现
2. 每个编号仅出现 1 次

不难发现，DP 枚举点的过程已经满足了条件 $1$。

考虑第二个条件。如果有编号出现不止一次，那么一定有编号没有出现，等价于有点没有被选到。我们无法直接求出答案，但可以构造容斥。$0$ 个点没有被选的方案数-至少 $1$ 个点没有被选的方案数+至少 $2$ 个节点没有被选的方案数……

枚举每个点被选的情况，容斥一下，最后就是正确答案了。

复杂度 $O(2^n n^3)$

```cpp
#include<cstdio>
#include<cstring>
#include<iostream>
using namespace std;
#define ll long long
const int N=20;
int n, m, s, tot;
int cnt, h[N], ver[N<<1], nxt[N<<1];
int tc, hc[N], vc[N*N], nc[N*N];
ll ans, dlt, f[N][N];
void add(int x,int y) { ver[++cnt]=y, nxt[cnt]=h[x], h[x]=cnt; }
void addc(int x,int y) { vc[++tc]=y, nc[tc]=hc[x], hc[x]=tc; }
void dp(int x,int fr) {
    int i, j, k, y;
    bool fg=0;
    for(i=h[x];i;i=nxt[i]) if(ver[i]!=fr) {
        y=ver[i];
        dp(y,x), fg=1;
    }
    if(!fg) {
        for(i=1;i<=n;++i) if(!((s>>(i-1))&1)) f[x][i]=1;
        // 特判
        return;
    }
    for(i=1;i<=n;++i) if(!((s>>(i-1))&1)) {
        f[x][i]=1;
        for(j=h[x];j;j=nxt[j]) if(ver[j]!=fr) {
            y=ver[j];
            ll o=0;
            for(k=hc[i];k;k=nc[k]) o+=f[y][vc[k]];
            f[x][i]*=o;
        }
    }
}
int main() {
    int i, x, y;
    scanf("%d%d",&n,&m);
    for(i=1;i<=m;++i) scanf("%d%d",&x,&y), addc(x,y), addc(y,x);
    for(i=1;i<n;++i) scanf("%d%d",&x,&y), add(x,y), add(y,x);
    for(s=0;s<(1<<n);++s) {
        tot=dlt=0;
        for(i=0;i<n;++i) if((s>>i)&1) ++tot;
        memset(f,0,sizeof(f));
        dp(1,0);
        for(i=1;i<=n;++i) dlt+=f[1][i];
        if(tot&1) ans-=dlt; else ans+=dlt;
    }
    printf("%lld\n",ans);
}
```
