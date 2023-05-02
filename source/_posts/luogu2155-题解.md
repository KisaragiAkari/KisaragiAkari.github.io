---
title: luogu2155 沙拉公主的困惑 题解
urls: lg2155-solution
tags: 数论
categories: 题解
math: true
abbrlink: df397a9d
date: 2021-08-28 22:34:23
---

[link](https://www.luogu.com.cn/problem/P2155)

由于 $\gcd(x,y) = \gcd(x + y,y)$，$m! \mid n!$

所以答案即为 $\frac{n!}{m!} \cdot \varphi(m!)$

<!--more-->

设 $\gcd(a,m!)=1$，那么 $\gcd(a+m!,m!)=1$，$\gcd(a+m!+m!,m!)=1$，最多重复这个过程 $\frac{n!}{m!}$ 次。

简单地说，每找到一个与 $m!$ 互质的数，就有 $\frac{n!}{m!}$ 个与 $m!$ 互质的数。

所以计算上式就可以了。

$O(n)$ 求阶乘和线性筛求欧拉函数。

$O( \log_2 y!)$ 用快速幂和费马小定理求逆元并回答询问。

实测这种筛法可过加强后的数据。

```cpp 
#include<cstdio>
#include<iostream>
using namespace std;
#define ll long long
const int N=1e7+10;
int t, p, n, m, cnt, pr[N], v[N];
int f[N], phi[N];
int fp(int x,int y) {
    int z=1;
    for(;y;x=(ll)x*x%p,y>>=1) if(y&1) z=(ll)z*x%p;
    return z;
}
void init() {
    int i, j;
    f[0]=f[1]=phi[1]=1;
    for(i=2;i<=1e7;++i) {
        // 这里的phi[i]实际上是 phi[1]*phi[2]*...phi[i]
        // 也就是phi[i!]。
        if(i!=p) f[i]=(ll)f[i-1]*i%p; else f[i]=f[i-1];
        if(!v[i]) v[i]=i, pr[++cnt]=i, phi[i]=(ll)phi[i-1]*(i-1)%p;
        else phi[i]=(ll)phi[i-1]*i%p;
        for(j=1;j<=cnt;++j) {
            if(pr[j]>v[i]||pr[j]*i>1e7) break ;
            v[pr[j]*i]=pr[j];
        }
    }
}
int main() {
    int x, y;
    scanf("%d%d",&t,&p);
    init();
    while(t--) {
		scanf("%d%d",&x,&y);
		if(x>=p&&y<p) puts("0");
		else printf("%lld\n",(ll)f[x]*phi[y]%p*fp(f[y],p-2)%p);	
	}
}
```
