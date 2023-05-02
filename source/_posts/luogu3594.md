---
title: luogu3594 WIL 题解
urls: lg3594-solution
tags:
  - 双指针
  - 单调队列
categories: 题解
math: true
abbrlink: 63e9075f
date: 2022-10-01 11:37:01
---

## 分析

显然，置零的区间长度一定为 $d$，否则一定不优。

那么答案大于等于 $d$。

<!--more-->

暴力做法，枚举左右端点 $[i,j]$，贪心地减去区间里长度为 $d$ 的最大的一块，用前缀和搞一下，$O(n^3)$。

考虑一个双指针做法，固定左端点 $i$，贪心地让右端点在 $[i+d,n]$ 中增长，维护区间内最大的长度为 $d$ 的块。如果区间和大于 $p$ 了，那么看减去最大块之后是否合法，$O(n^2)$。

考虑优化这个做法。

不难发现对于每个固定 $i$ 找 $j$ 的过程，都要维护区间最大块，但是很多是重复的。反过来，固定右端点，维护最靠前的合法左端点，这个过程是个滑动窗口。而要维护的区间最大块，在这个过程中也是一个滑动窗口。

用单调队列维护当前 $[i,j]$ 区间最大块即可，复杂度 $O(n)$。

## CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
int read() {
	int a=0, f=1; char c=getchar();
	while(!isdigit(c)) {
		if(c=='-') f=-1;
		c=getchar();
	}
	while(isdigit(c)) a=a*10+c-'0', c=getchar();
	return a*f;
}
const int N=2e6+5;
int n, p, d, ans, a[N], s[N], q[N], c[N];
signed main() {
	n=read(), p=read(), d=read();
	for(int i=1;i<=n;++i) a[i]=read(), s[i]=s[i-1]+a[i];
	for(int i=d;i<=n;++i) c[i]=s[i]-s[i-d]; 
    // c[i]表示以i结尾的，长度为d的块的和
	int w=s[d];
	int l=1, r=1;
	q[l]=d;
	ans=d;
	for(int i=d+1,j=1;i<=n;++i) {
        // 这里写的是[j,i]
		w+=a[i];
		while(l<=r&&c[q[r]]<=c[i]) --r;
		q[++r]=i;
		while(l<=r&&(w-c[q[l]])>p) {
            // 当前区间和减去最大块也不能满足
			if(q[l]<j+d) ++l;
            // q[l]<j+d时，q[l]才非法
			w-=a[j++];
            // 这个j就不可能作为左端点了
        }
		ans=max(ans,i-j+1);
	}
	printf("%lld\n",ans);
}

```

