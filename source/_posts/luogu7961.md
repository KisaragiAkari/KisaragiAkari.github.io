---
title: luogu7961 数列 题解
urls: lg7961-solution
tags: DP
categories: 题解
math: true
abbrlink: abc6f0b7
date: 2022-06-21 18:08:46
---

## 前言

好强的状态设计啊，从来没有见过。

可是暴力分竟然给到了一半，连一半都拿不到吗？

那个时候连记忆化搜索都写不熟练，总是止步于似懂非懂的家伙。

我承认，那时候的遗憾，和我即将面临的抉择，没有任何关系。但是，连续地与期望失之交臂，真的会感觉自己是彻头彻尾地废物。

25 号，什么时候到来啊！马上要见分晓了！

卷王同学们在一次迷惑的提前批招生后安定下来，只有一面之缘的朋友的情况也渐入佳境。

我只能不去关心“他们”的事情，这般沉溺于夏日，沉溺在自己的世界中。

<!--more-->

## 暴力

### 分析

首先明确 $a_i \in [0,m]$。

设 $f(x,S)$ 为当前 $\{a_i\}$ 有 $x$ 项，其中 $S = \sum_{i=1}^x 2^{a_i}$，还能够产生的权值和。

状态总数为 $O(2^{m}nm)$，只要保证转移在 $O(m)$ 之内实现就可行。

这个状态明显是记忆化搜索。

如果多选 $a_{x+1}=k$，那么必然 $k \in [0,m]$，同时 $S+2^k$，状态变成了 $f(x+1,S+2^k)$，对 $f(x,S)$ 产生的贡献为 $f(x+1,S+2^k) \cdot v_k$。这个状态的每一个前继状态都必定要在算权值的时候乘上 $v_k$，由于乘法结合律与分配律，只要边界设置对了，这样做就是对的。枚举 $k$，转移 $O(m)$。
$$
f(x,S) = \sum_{i=0}^m f(x+1,S+2^{m}) \cdot v_i
$$
然后就是边界，如果上面我们这么干，那么必须 $x=n$ 且 1 的个数小于 $k$ 的状态的贡献为 1，否则为 0。

最终答案 $f(0,0)$

复杂度 $O(2^m m^2 n)$

期望得分 $50pts$

### CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N=35, M=105;
const ll mod=998244353;
int n, m, k, v[N];
ll f[N][123000];
int read() {
	int a=0; char c=getchar();
	for(;!isdigit(c);c=getchar());
	for(;isdigit(c);a=a*10+(c^48),c=getchar());
	return a;
}
int count(ll x) {
	int cnt=0;
	while(x) cnt+=x&1, x>>=1;
	return cnt;	
}
// 统计x中1的个数
ll dp(int x,ll s) {
	if(x>=n) return count(s)<=k;
	if(f[x][s]!=-1) return f[x][s];
	ll dlt=0;
	for(int i=0;i<=m;++i) (dlt+=dp(x+1,s+(1<<i))*v[i])%=mod;
	return f[x][s]=dlt;
}
int main() {
	n=read(), m=read(), k=read();
	for(int i=0;i<=m;++i) v[i]=read();
	memset(f,-1,sizeof(f));
	printf("%lld\n",dp(0,0));
}
```

## 正解

### 分析

暴力状态无论如何优化都无法承受了，考虑重新设计状态。

暴力状态瓶颈的原因在与状态数量太多，十进制下 $S$ 实在是太大了。那么可以试着按照二进制位设计状态。

建议好好品一下**样例解释 #1**……

当一个 $a_i$ 确定时，只会发生 2 种情况。

1. $a_i$ 这个数首次出现，那么 $S$ 的二进制中 1 的个数增加 1。
2. $a_i$ 这个数之前出现过了，那么 $S$ 的二进制中那一位就要向前一位进 1，同时变成 0。由于不知道下一位是啥所以无法预测 1 的数量变化。

但是对于任何一位，他只能向更高的一位进，至于最后到了那里不重要，可以到更高位的时候再考虑。从高位状态转移到低位状态无后效性，还是记忆化搜索。

那么一个神级状态就要产生了！

$f(k,i,x,y)$，表示已经确定了 $S$ 二进制的 $k$ 位，其中 $\{a\}$ 数列确定了 $i$ 项，有 $x$ 个 1，同时第 $k-1$ 位进了 $y$ 个 1 到 $k$ 位，还能够产生的贡献。

为什么这么设计呢？首先它能够唯一确定地表示 $S$ 与 $\{a\}$，并且状态数量级别是 $O(mn^3)$。要知道测试点中 $m \in [7,100]$，$n \in [5,30]$，这是相当大的优化了。而且样例的做法启发我们用组合数求出方案，用 $v_k$ 的不同次方来统计贡献。

$k$ 其实隐式代表着 $v_k$。转移的时候枚举 $j \in [0,n-i]$，表示剩下的 $n-i$ 项中，$k$ 用几项。那么从 $n-i$ 个位置里任选 $j$ 个为 $k$，方案数为 $C_{n-i}^j$，都会产生 $v_k^j$ 的贡献。

最关键的是剩下两个状态如何表示。

上一位进了 $y$ 个 1，这一位多了 $j$ 个 1，二进制下满二进一，那么就是进 $\lfloor \frac{y+j}{2} \rfloor$ 个 1 到 $k+1$ 位。

由于规定了进位顺序，当 $y+j$ 是奇数时，$k$ 位上必定留下 1，$x+1$，否则 $x$ 不变。

好的！转移 $O(n)$！
$$
f(k,i,x,y) = \sum_{j=0}^{n-i} f(k+1,i+j,x+d(y+j),\lfloor \frac{y+j}{2} \rfloor) \cdot C^j_{n-i} \cdot v_k^j
$$
当 $t$ 为奇数时，$d(t)$ 为 1，反之为 0。

边界依然很重要。当 $k>m$ 时，不存在这样的 $S$，这个状态直接返回 0。

当 $i=n$ 的时候，此时 $[1,k]$ 位上有 $x$ 个 1，前面进了 $y$ 个 1。由于 $(k,m]$ 此时全为 0，那么 $y$ 个 1 最后产生的 1 数量是它的二进制中 1 的个数。所以就像暴力方法一样，如果 1 的数量合法，那么返回 1，否则返回 0。

答案 $f(0,0,0,0)$

复杂度 $O(mn^4)$

期望得分 $100pts$

### CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N=35, M=105;
const ll mod=998244353;
int n, m, kk;
ll c[N][N], v[M][N], f[M][N][N][N];
// c[i][j]是组合数，v[i][j]是v[i]的j次幂
ll read() {
	ll a=0; char c=getchar();
	for(;!isdigit(c);c=getchar());
	for(;isdigit(c);a=a*10+(c^48),c=getchar());
	return a;
}
int count(ll x) {
	int cnt=0;
	while(x) cnt+=x&1, x>>=1;
	return cnt;	
}
ll dp(int k,int i,int x,int y) {
	if(i>=n) return x+count(y)<=kk;
	if(k>m) return 0;
	if(f[k][i][x][y]!=-1) return f[k][i][x][y];
	ll dlt=0;
	for(int j=0;j<=n-i;++j) (dlt+=dp(k+1,i+j,x+(y+j)%2,(y+j)/2)*c[n-i][j]%mod*v[k][j]%mod)%=mod;
	return f[k][i][x][y]=dlt;
}
int main() {
	n=read(), m=read(), kk=read();
	for(int i=0;i<=m;++i) v[i][1]=read();
	c[0][0]=1;
	for(int i=1;i<=n;++i) {
		c[i][0]=c[i][i]=1;
		for(int j=1;j<i;++j) c[i][j]=(c[i-1][j]+c[i-1][j-1])%mod;
	}
	for(int i=0;i<=m;++i) {
		v[i][0]=1;
		for(int j=2;j<=n;++j) v[i][j]=(v[i][j-1]*v[i][1])%mod;
	}
    // 预处理组合数与次幂
	memset(f,-1,sizeof(f));
	printf("%lld\n",dp(0,0,0,00));
}
```
