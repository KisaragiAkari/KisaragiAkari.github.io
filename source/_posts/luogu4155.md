---
title: luogu4155 国旗计划 题解
urls: lg4155-solution
tags: 倍增
categories: 题解
math: true
abbrlink: '61392649'
date: 2022-05-07 16:39:51
---

## 分析

既然区间没有包含关系，那么把所有区间按照左端点递增排序，这样右端点也是递增的。

<!--more-->

断环为链，当区间 $[l_i,r_i]$  被强制选择时，只要贪心地选择靠右的区间，记录达到 $l_i+m$ 位置时经过的区间数量。（也就是把环覆盖一遍）

这个算法是没问题的，但是要求出每个区间的情况，复杂度 $O(n^2)$，不能承受。

要优化的是暴力求固定边界的区间数量，可以用倍增。

设 $f(x,i)$ 为区间 $x$ 经过 $2^i$ 个区间所能达到的区间编号。

注意最后要加上 2，自己和最后一个区间统计不到。

## CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=5e5+5;
#define ll long long
int n, m, len, ans[N], f[N][22];
struct Interval { int l, r, id; } a[N];
bool operator<(Interval a,Interval b) { return a.l!=b.l? a.l<b.l:a.r<b.r; };
int solve(int x) {
	int ans=0, ed=a[x].l+m;
	for(int i=20;i>=0;--i) if(f[x][i]&&a[f[x][i]].r<ed) ans+=1<<i, x=f[x][i];
	return ans+2;
} 
int main() {
	scanf("%d%d",&n,&m), len=n;
	for(int i=1;i<=n;++i) {
		scanf("%d%d",&a[i].l,&a[i].r), a[i].id=i;
		if(a[i].l>a[i].r) a[i].r+=m;
     	// 特殊处理，此时是l>m，r<m
		else a[++len]={a[i].l+m,a[i].r+m,i};
        // 断环为链
	}
	sort(a+1,a+len+1);
	a[len+1].r=0x7fffffff; // 这里一定要足够大
	int r=1;
	for(int i=1;i<=len;++i) {
		while(r<=len&&a[r+1].l<=a[i].r) ++r;
		f[i][0]=r;
	}
	for(int j=1;j<=20;++j) for(int i=1;i<=len;++i) f[i][j]=f[f[i][j-1]][j-1];
	for(int i=1;i<=len;++i) if(a[i].l<=m) ans[a[i].id]=solve(i);
	for(int i=1;i<=n;++i) printf("%d%c",ans[i]," \n"[i==n]);
} 
```

