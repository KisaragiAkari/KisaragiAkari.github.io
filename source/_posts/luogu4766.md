---
title: luogu4766 Outer space invaders 题解
urls: lg4766-solution
tags:
  - DP
  - 区间DP
categories: 题解
math: true
abbrlink: ae16c1b8
date: 2022-02-04 15:04:48
---

## 分析

满足 $n \in [1,300]$ 和 $a_i,b_i \in [1,10000]$，这都不离散化，那还是人么（）。

没有明显的线性更新顺序，「时间」则是一个天然的序，且区间信息可以合并，所以设 $f(l,r)$ 为在时间 $[l,r]$ 内，消灭所有 aliens，所需要的最小代价。

<!--more-->

将时间离散化，数组就能开的下了。此时设所有不重复的 $a_i$ 与 $b_i$ 将总时间划分成 $m$ 段（也即是离散化后 $a_i$ 与 $b_i$ 的总数）。

边界 $f(i,j) = \inf$。

答案 $f(1,m)$。

由于一次会消灭所有距离 $R$  以内的 aliens，所以优先攻击最远的。对于区间 $[i,j]$，找到区间内距离最大的 $x$。如果没有，那么 $f(i,j) = 0$。如果有的话，那么我们可在 $[a_x,b_x]$  任意一个时刻消灭它，可以取最小值。

转移
$$
f(i,j) = \min { \{  f(i,k-1)+f(k+1,j)+d_x\} }
$$
要注意枚举元区间长时应从 1 开始，因为不能直接处理这部分信息。

## CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=605, inf=0x3f3f3f3f;
int T, n, m, t[N], a[N], b[N], d[N], f[N][N];
inline void sol() {
    m=0;
    memset(f,0x3f,sizeof(f));
    scanf("%d",&n);
    for(int i=1;i<=n;++i) {
        scanf("%d%d%d",&a[i],&b[i],&d[i]);
        t[++m]=a[i], t[++m]=b[i];
        f[i][i-1]=f[i+1][i]=0;
    }
    sort(t+1,t+m+1);
    m=unique(t+1,t+m+1)-t-1;
    for(int i=1;i<=n;++i) {
        a[i]=lower_bound(t+1,t+m+1,a[i])-t;
        b[i]=lower_bound(t+1,t+m+1,b[i])-t;
        // 直接预处理下标
    }
    for(int l=1;l<=m;++l) for(int i=1;i+l-1<=m;++i) {
    // 从l=1开始
        int j=i+l-1, x=0;
        for(int k=1;k<=n;++k)
            if(a[k]>=i&&b[k]<=j&&d[k]>d[x]) x=k;
        if(!x||a[x]==b[x]) { f[i][j]=0; continue; }
        // a[x]==b[x]的情况不会出现，但是强迫症还是特判一下
        for(int k=a[x];k<=b[x];++k)
            f[i][j]=min(f[i][j],f[i][k-1]+f[k+1][j]+d[x]);
    }
    printf("%d\n",f[1][m]);
}
int main() { for(scanf("%d",&T);T--;sol()); }
```
