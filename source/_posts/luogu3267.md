---
title: luogu3267 侦察守卫 题解
urls: lg3267-solution
tags:
  - DP
  - 树形DP
categories: 题解
math: true
abbrlink: 44aa4b71
date: 2021-10-16 17:44:28
---

本题难点，在于设计「互补」的状态。

每个守卫覆盖的 $d$ 的方向是任意的，不能再像传统树形 DP 那样状态单单表示子树信息。

<!--more-->

定义「上方」为向根的方向，「下方」为子树内。

设 $f_{x,i}$ 为覆盖 $x$ 下方的关键点，并且**能够**向上方覆盖 $i$ 范围内的关键点，所需要的最小代价。

设 $g_{x,i}$ 为覆盖 $x$ 下方 $[i,d]$ 范围内的关键点，所需要的最小代价。

这样， $f$ 与 $g$ 就能拼凑出正确的答案。

&nbsp;

边界
$$
g_{x,i}=w_x, g_{x,d+1}=\inf \quad i \in [1,d]
$$
考虑转移。
$$
g_{x,i} = \min_{y \in son(x)}{ \{ f_{y,i}+g_{x,i},f_{x,i+1}+g_{y,i+1} \}  }
$$

因为 $[i,d]$ 的代价一定能够覆盖 $[i-1,d]$，所以

$$
g_{x,i} = \min_{i \in [0,d]} { \{  g_{x,i+1}\} }
$$

接着不难想到
$$
 f_{x,0}=g_{x,0}
$$

$$
f_{x,i}=\sum_{y \in son(x)} {f_{y,i-1}} \quad i \in [1,d+1]
$$
向上覆盖 $i+1$ 的代价，一定能够覆盖 $i$，所以
$$
f_{x,i}=\min_{i \in [1,d+1]}{ \{ f_{x,i-1} \} }
$$
答案 $f_{1,0}$

复杂度 $O(nd)$

```cpp
#include<cstdio>
#include<iostream>
using namespace std;
const int N=5e5+6;
int n, m, d, w[N], f[N][20], g[N][20];
int c, h[N], ver[N<<1], nxt[N<<1];
bool v[N];
void add(int x,int y) { ver[++c]=y, nxt[c]=h[x], h[x]=c; }
void dp(int x,int fr) {
    int i, j, y;
    if(v[x]) f[x][0]=g[x][0]=w[x];
    for(i=1;i<=d;++i) g[x][i]=w[x]; g[x][d+1]=(1<<30);
    for(i=h[x];i;i=nxt[i]) if(ver[i]!=fr) {
        y=ver[i], dp(y,x);
        for(j=d;j>=0;--j) g[x][j]=min(g[x][j]+f[y][j],g[y][j+1]+f[x][j+1]);
        for(j=d;j>=0;--j) g[x][j]=min(g[x][j],g[x][j+1]);
        f[x][0]=g[x][0];
        for(j=1;j<=d+1;++j) f[x][j]+=f[y][j-1];
        for(j=1;j<=d+1;++j) f[x][j]=min(f[x][j],f[x][j-1]);
        // 注意转移的循环顺序
    }
}
int main() {
    int i, x, y;
    scanf("%d%d",&n,&d);
    for(i=1;i<=n;++i) scanf("%d",&w[i]);
    scanf("%d",&m);
    for(i=1;i<=m;++i) scanf("%d",&x), v[x]=1;
    for(i=1;i<n;++i) scanf("%d%d",&x,&y), add(x,y), add(y,x);
    dp(1,0);
    printf("%d\n",f[1][0]);
}
```

