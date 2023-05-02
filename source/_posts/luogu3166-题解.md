---
title: luogu3166 数三角形 题解
urls: lg3166-solution
tags: 组合数学
categories: 题解
math: true
abbrlink: 810c794f
date: 2021-08-28 21:34:35
---

[link](https://www.luogu.com.cn/problem/P3166)

显然的，在  $N \times M$ 的网格中任选三点的方案数为
$$
C^3_{n+m} = \frac{(n+m)!}{3! \cdot (n+m-3)!} = \frac{(n+m) \cdot (n+m-1) \cdot (n+m-2)}{6}
$$
还要减去三点共线的情况。

<!--more-->

三点共线且沿矩形边的方案数
$$
C_n^3 = \frac{n!}{3! \cdot (n-3)!} = \frac{n \cdot (n-1) \cdot (n-2)}{6}
$$

$$
 C^3_m= \frac{m!}{3! \cdot (m-3)!} = \frac{m \cdot (m-1) \cdot (m-2)}{6}
$$



三点共线且不沿矩形边的方案数

设斜线 $ (0,0) \rightarrow (x,y)$，则其经过点数为 $ \gcd(x,y)+1$。

又因为是三点共线，所以有 $ \gcd(x,y)-1$ 种可能。

该斜线可以左右平移 $ (n-x)$ 次，上下平移 $ (m-y)$ 次，还有正反两种可能。

乘法原理。

实现的时候将 $ n$ 与 $ m $ 都加 1，因为这是坐标，而组合数计算的是点数，并且计算斜边时也要注意边界的变化。

```cpp 
#include<cstdio>
using namespace std;
#define ll long long
ll n, m, x, y, z;
ll gcd(ll x,ll y) { return y? gcd(y,x%y):x; }
int main() {
    scanf("%lld%lld",&n,&m), ++n, ++m;
    z=n*m*(n*m-1)*(n*m-2)/6;
    x=n*(n-1)*(n-2)/6*m;
    y=m*(m-1)*(m-2)/6*n;
    z=z-x-y;
    for(int i=1;i<n;++i) for(int j=1;j<m;++j) z-=(n-i)*(m-j)*(gcd(i,j)-1<<1);
    printf("%lld\n",z);
}
```
