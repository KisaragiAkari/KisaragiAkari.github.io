---
title: luogu4139 上帝与集合的正确用法 题解
urls: lg4139-solution
tags:
  - 数论
  - 欧拉函数
categories: 题解
math: true
abbrlink: '6931367'
date: 2021-08-28 21:35:14
---


[link](https://www.luogu.com.cn/problem/P4139)

设 $S=2^{2^{2 \cdots}}$

由扩展欧拉定理得欧拉降幂公式

$$
\forall k > \varphi(p) \quad a^k \equiv a^{k \, \bmod \, \varphi (p) + \varphi (p)} \, (\bmod p)
$$

对于本题

$$
2^S \equiv 2^{S \, \bmod \, \varphi (p) + \varphi (p)} \, (\bmod p)
$$

<!--more-->

仔细观察不难发现

$S \bmod \varphi(p)$

实际上又是一个这样的式子，我们采用递归的方法求解。

在不断递归的过程中，要膜的欧拉函数是越来越小的，并且最多递归 $\log_2p$ 次。

不要傻乎乎地去预处理欧拉函数（确信

```cpp 
#include<cstdio>
using namespace std;
#define ll long long
ll t, p;
ll fp(ll x,ll y,ll p) {
    ll z=1ll;
    for(;y;x=x*x%p,y>>=1ll) if(y&1ll) z=z*x%p;
    return z;
}
ll phi(ll x) {
    ll y=x;
    for(int i=2;i*i<=x;++i) if(x%i==0) {
        y/=i, y*=i-1;
        while(x%i==0) x/=i;
    }
    if(x^1) y/=x, y*=x-1;
    return y;
}
ll f(ll x) {
    if(x==1) return 0ll;
    ll y=phi(x);
    return x!=1ll? fp(2ll,f(y)+y,x):0ll;
}
int main() {
    scanf("%lld",&t);
    while(t --> 0) scanf("%lld",&p), printf("%lld\n",f(p));
}
```
