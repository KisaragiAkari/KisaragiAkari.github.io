---
title: luogu8112 符文破译 题解
urls: lg8112-solution
tags:
  - 字符串
  - 扩展kmp
  - DP
  - 单调队列
categories: 题解
math: true
abbrlink: f8981e35
date: 2022-10-22 21:51:29
---

## 分析

显然 $Z$ 函数。

由于在 $Z$ 函数中考虑的是匹配串的后缀，所以从后往前考虑。

<!--more-->

设 $f_i$ 为考虑 $[i,n]$，能够划分的最小段数，设 $g_i$ 为匹配串的 $[i,n]$ 与模式串的 LCP 长度。

转是显然的。

$$
f_i = \min_{j \in [i+1,i+g_i]} \{ f_j  \} + 1
$$

虽然 $i+g_i$ 不存在单调性，但是考虑如果存在 $i<j$，满足 $g_i > g_j$，那么从 $j$ 划分一定不优，于是直接把 $j$ 删掉就行，这样就满足单调性，可以用单调队列优化。

不难证明，如果这样做后无解，那么不这样做一定无解。


## CODE

```cpp
const int N=1e7+5, inf=0x3f3f3f3f;
int n, m, z[N], g[N], f[N], q[N];
char s[N], t[N];
void Z() {
	int l=0, r=0;
	z[0]=n;
	for(int i=1;i<n;++i) {
		if(i<=r&&z[i-l]<r-i+1) z[i]=z[i-l];
		else {
			z[i]=max(r-i+1,0);
			while(i+z[i]<n&&t[z[i]]==t[i+z[i]]) ++z[i];
			if(r<i+z[i]-1) l=i, r=i+z[i]-1;
		}
	}
}
void exkmp() {
	Z();
	int l=-1, r=-1;
	for(int i=0;i<m;++i) {
		if(i<=r&&z[i-l]<r-i+1) g[i]=z[i-l];
		else {
			g[i]=max(r-i+1,0);
			while(i+g[i]<m&&t[g[i]]==s[i+g[i]]) ++g[i];	
			if(r<i+g[i]-1) l=i, r=i+g[i]-1;
		}
		
	}
}
signed main() {
	n=read(), m=read();
	scanf("%s%s",t,s);
	exkmp();
	for(int i=0,j=0;i<m;++i) {
		int k=i+g[i];
		if(j>k) g[i]=inf; else j=k;
	}
	memset(f,0x3f,sizeof(f));
	f[m]=0;
	int l=1, r=1;
	q[l]=m;
	// 初始决策，使得f[q[l]]=0
	for(int i=m-1;~i;--i) if(g[i]!=inf) {
		while(l<=r&&q[l]>i+g[i]) ++l;
		f[i]=f[q[l]]+1;
		while(l<=r&&f[q[r]]>=f[i]) --r;
		q[++r]=i;
	}
	if(f[0]<inf) printf("%d\n",f[0]);
	else puts("Fake");
}
```
