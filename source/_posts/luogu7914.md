---
title: luogu7914 括号序列 题解
urls: lg7914-solution
tags:
  - DP
  - 区间DP
categories: 题解
math: true
abbrlink: 5c98d7fc
date: 2022-04-13 18:01:44
---

## 分析

明显是一个区间 DP，但是合法串情况比较多，所以要仔细分类。

设 $f(l,r)$ 为区间 $[l,r]$ 内的合法串的数量，$q(l,r)$ 为 $[l,r]$ 能否变成不超过 $k$ 个`*`组成的串。

<!--more-->

以下转移都是在满足 $l$ 能够成为左括号，$r$ 能够成为右括号的前提下。

&nbsp;

1. `()`

显然，特判 $l+1=r$ 就好了。
$$
1 \rightarrow f(l,r)
$$

2. `(S)`

这种情况只会发生在 $l+1 \neq r$ 的时候。左右括号都已经确定了，只要里面的能构成一整个长度合法的`*`串，就能贡献 1 个方案数。而 $[l+1,r-1]$ 不管怎么样都能产生 $f(l+1,r-1)$ 的贡献。
$$
f(l+1,r-1) + q(l+1,r-1) \rightarrow f(l,r)
$$

3. `(SA)`

枚举`S`最后一个字符的位置 $k$，如果 $q(l,k)=1$，那么就转移。
$$
q(l,k) \cdot f(k+1,r) \rightarrow f(l,r)
$$

4. `(AS)`

与上一步相同，枚举 $A$ 的最后一个字符的位置 $k$，可以与上一步一起转移。
$$
f(l,k) \cdot q(l+1,k) \rightarrow f(l,r)
$$

5. `AB`

枚举断点 $k$，暴力转移。
$$
f(l,k) \cdot f(k+1,r) \rightarrow f(l,r)
$$
且慢！

`()()()`

这个串是不是被重复计算了？$[1,2]$ 与 $[3,6]$，$[1,4]$ 与 $[5,6]$。

>至理名言：当你想到一个 fAKe 的思路时，想想它为什么是 fAKe 的。

由于从不同的断点断开组成了相同的串，所以重复计算了。

如何避免？如果强制让 $A$ 是由上面 4 种转移方式得到的串，那么 $[1,4]$ 这个区间就不合法了。因为以上 4 种方法要求左右端点组成`()`，所以`()()`里面的`)(`是不合法的，因此 $[1,4]$ 这个种断开方法不会产生贡献。如果不这样，那么`()()`可以看作`A=()`，`B=()`组成的`AB`形式的合法串，就重复了。

所以设 $g(l,r)$ 为前 4 种转移对 $f(l,r)$ 产生的贡献，能够在处理完后直接求出。

转移
$$
g(l,k) \cdot f(k+1,r) \rightarrow f(l,r)
$$

6. `ASB`

同上，枚举 $A$ 的最后一位 $i$ 和 $B$ 的第一位 $j$。

那么 $A = [l,i]$，$S=[i+1,j-1]$，$B[j,r]$。

转移
$$
\sum_{i=l+1}^{i<r} \sum_{j=i+2}^{j<r} g(l,i) \cdot q(i+1,j-1) \cdot f(j,r)
$$
单是这个转移就是 $O(n^2)$ 的，考虑优化。

首先，`S`这个串必须是连续的，而只有能够出现一个`S`才能产生贡献。所以，能够产生贡献的一定是一个连续的区间。（想象一下“滑动窗口”）。

所以一定能找到一个 $p$，满足 $\forall k \in[i,p]$，都有 $q(i,k)=1$。

所以提一下公因式 $g(l,i)$，得到
$$
\sum_{i=l+1}^{i<r} \Big(  g(l,i) \cdot \sum_{j=i+2}^{j \le p} f(j,r) \Big)
$$
关于 $j$ 的那一项是能够 $O(1)$ 计算的。右端点固定，左端点不同，可以使用后缀和优化。这个可能不太熟悉，但就是把前缀和对 $[1,k]$ 求和改为对 $[k,n]$ 求和。

设 $S_k$ 为 $[k,r]$ 这一段的和，于是转移就有了
$$
\sum_{i=l+1}^{i<r} \Big(  g(l,i) \cdot (S_{i+2}-S_{p+1} ) \Big) \rightarrow f(l,r)
$$
&nbsp;

最终答案 $f(1,n)$

## CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N=505, mod=1e9+7; 
int n, k;
char s[N];
ll f[N][N], g[N][N], q[N][N], sum[N];
bool check(int l,int r) {
	if(s[l]=='('&&(s[r]==')'||s[r]=='?')) return 1;
	if(s[l]=='?'&&(s[r]==')'||s[r]=='?')) return 1;
    // 判断是否合法
	return 0;
}
int main() {
	scanf("%d%d%s",&n,&k,s+1);
	for(int l=1;l<=n;++l) for(int r=l;r<=n;++r) {
		if(s[r]!='*'&&s[r]!='?') break;
		if(r-l+1>k) break; // 长度不超过k
		q[l][r]=1;
	}
	for(int len=2;len<=n;++len) for(int l=1;l+len-1<=n;++l) {
		int r=l+len-1;
		if(check(l,r)) {
			if(l+1==r) (f[l][r]+=1)%=mod; // 1
			else (f[l][r]+=f[l+1][r-1]+q[l+1][r-1])%=mod; // 2
			for(int k=l+1;k<=r-2;++k) {
				(f[l][r]+=f[l+1][k]*q[k+1][r-1]+f[k+1][r-1]*q[l+1][k])%=mod;
                // 3和4放到一块转移了
			}
			g[l][r]=f[l][r];
			for(int k=l+1;k<=r-2;++k) (f[l][r]+=g[l][k]*f[k+1][r]%mod)%=mod;
            // 5
			memset(sum,0,sizeof(sum));
			for(int k=r;k>=l;--k) (sum[k]+=sum[k+1]+f[k][r])%=mod;
            // 后缀和数组
			int p=0;
			for(int k=l+1;k<=r-2;k++) {
				p=max(p,k+1);
				while(q[k+1][p]) ++p; 
				(f[l][r]+=g[l][k]*(sum[k+2]-sum[p+1])%mod)%=mod;
                //就和前缀和数组求区间和要a[r]-a[l-1]一样，这里p要+1
			}
		}
	}
	printf("%lld\n",(f[1][n]+mod)%mod);
}
```

