---
title: luogu7715 Shape 题解
urls: lg7715-solution
tags: DP
categories: 题解
math: true
abbrlink: 7b6b8f65
date: 2022-11-12 19:58:42
---

直接枚举 $(x_1,y_1)$，$(x_2,y_2)$，对每行每列都做前缀和，可以得到 20pts。

<!--more-->

枚举点不如枚举横杠，设 $f(x,y)$ 为在节点 $(x,y)$，同时往上下方向扩展的格数。对于一个 $x$，枚举 $y_1,y_2$，如果 $(x,y_1)$ 和 $(x,y_2)$ 构成一个横杠，那么贡献为 $\min\{ f(x,y_1),f(x,y_2) \}$。

可以拿到 50pts。

枚举横杠又不如枚举极长横杠。

考虑对于每一行 $x$，直接处理两个黑色格子中间的部分，设为 $[l,r]$，那么贡献为
$$
\sum_{y_1=l}^r \sum_{y2=y_1+1}^r \min\{f(x,y_1),f(x,y_2)\}
$$
考虑如果将 $f(x,y)$ 递增排序，那么对于排名为 $k$ 的 $f(x,y_0)$ 会贡献出 $r-k$ 次。

于是乎这部分的复杂度为 $O(nm \log_2 m)$，可以通过。

关于 $f$ 的预处理，要摒弃求前缀和暴力判断的做法，下面给出官方题解中的 $O(nm)$ 做法。

## CODE

```cpp
const int N=2e3+5;
int n, m, ans, t[N], a[N][N], f[N][N];
int calc(int x,int l,int r) {
	if(l>r) return 0;
	int cnt=0, ans=0;
	for (int i=l;i<=r;++i) t[++cnt]=f[x][i];
	sort(t+1,t+cnt+1);
	for(int i=1;i<=cnt;++i) ans+=t[i]*(cnt-i);
	return ans;
}
signed main() {
	n=read(), m=read();
	for(int i=1;i<=n;++i) for(int j=1;j<=m;++j) {
		a[i][j]=read()? 0:1;
	}
	for(int j=1;j<=m;++j) {
		for(int i=1,p=-1;i<=n;++i) {
			if(!a[i][j]) p=-1; else ++p;
			f[i][j]=p;
		}
		for (int i=n,p=-1;i;--i){
			if (!a[i][j]) p=-1; else ++p;
			f[i][j]=min(f[i][j],p);
		}
	}
	for(int x=1;x<=n;++x) {
		int pre=1;
		for(int y=1;y<=m;++y) if(!a[x][y]) ans+=calc(x,pre,y-1), pre=y+1;
		ans+=calc(x,pre,m);
	}
	printf("%lld\n",ans);
}
```
