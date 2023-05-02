---
title: luogu6748 Fallen Lord 题解
urls: lg6748-solution
tags:
  - DP
  - 树形DP
  - 贪心
categories: 题解
math: true
abbrlink: '38144765'
date: 2022-10-23 11:55:45
---

值得一提的是，我一开始读错题，认为对于节点 $x$，是根到它的路径上的中位数不能超过 $a_x$，导致连暴力都不会打。

<!--more-->

## 分析

发现每条边 $(x,y)$ 的权值只可能是 $a_x$，$a_y$，$m$。

对于一个节点 $x$，与它相连的所有边权的中位数不超过 $a_x$，那么 $deg_x$ 条边中，最多有 $\lfloor \frac{deg_x}{2} \rfloor + 1$ 条边小于等于 $a_x$，也就是至多有 $\lceil \frac{deg_x}{2} \rceil - 1$ 条边权大于 $a_x$。设其为 $t$。

设 $f(x,0/1)$ 表示 $(x,fa_x)$ 的权值是小于等于还是大于 $a_x$。

对于 $f(x,0)$，在 $x$ 连向子节点的边中，最多可以有 $t$ 条边大于 $a_x$。

对于 $f(x,1)$，这样的边数为 $t-1$。

考虑 $x$ 的子节点 $y$，设 $g(y,0)$ 为 $(x,y)$ 的边小于等于 $a_x$ 时，以 $y$ 为根的子树加上这条边的最大权值，$g(y,1)$ 则反之。
$$
g(y,0) = \max\{ f(y,0) +\min(a_x,a_y), f(y,1)+a_x\}
$$

$$
g(y,1) = \max\{ f(y,0) + a_y, f(y,1) + m \}
$$

对于每个 $y$，只会为 $f(x)$ 贡献其中一个。

显然 $g(y,1) \ge g(y,0)$，那么设 $g(y,0)$ 的和为 $s$。

将所有 $g(y,1)-g(y,0)$ 递减排序，记为 $g$。
$$
f(x,0) = s +\sum_{i=1}^t g(i)
$$

$$
f(x,1) = s + \sum_{i=1}^{t-1} g(i)
$$

特别的，当 $t=0$ 时，$f(x,1) = -\infty$

答案是 $\max\{ f(1,0),f(1,1)\}$。

## CODE

```cpp
const int N=5e5+5;
int n, m, a[N], c[N], g[N], f[N][2], deg[N];
int tot, h[N], to[2*N], nxt[2*N];
void add(int x,int y) {
	to[++tot]=y, nxt[tot]=h[x], h[x]=tot;
}
void addedge(int x,int y) {
	add(x,y), add(y,x);
	++deg[x], ++deg[y];
}
void dp(int x,int fa) {
	int cnt=0, s=0, t=(deg[x]+1)/2-1;
	for(int i=h[x];i;i=nxt[i]) {
		int y=to[i];
		if(y==fa) continue;
		dp(y,x);
	}
	for(int i=0;i<=t;++i) g[i]=c[i]=0;
	for(int i=h[x];i;i=nxt[i]) {
		int y=to[i];
		if(y==fa) continue;
		int g1=max(f[y][0]+a[y],f[y][1]+m);
		int g0=max(f[y][0]+min(a[x],a[y]),f[y][1]+a[x]);
		s+=g0;
		c[++cnt]=g1-g0;
	}
	sort(c+1,c+cnt+1,greater<int>());
	for(int i=1;i<=t;++i) g[i]=g[i-1]+c[i];
	f[x][0]=s+g[t];
	if(t) f[x][1]=s+g[t-1]; else f[x][1]=-1e15;
}
signed main() {
	n=read(), m=read();
	for(int i=1;i<=n;++i) a[i]=read();
	for(int i=1;i<n;++i) {
		int x=read(), y=read();
		addedge(x,y);
	}
	dp(1,1);
	printf("%lld\n",max(f[1][0],f[1][1]));
}
```
