---
title: luogu7514 卡牌游戏 题解
urls: lg7514-solution
tags: 双指针
categories: 题解
math: true
abbrlink: b6941fec
date: 2022-04-15 19:15:20
---

## 分析

要求极差尽量小，也就是让最小值尽量大，最大值尽量小。

套路就是固定一个值，然后贪心地去扩展另一个值。

<!--more-->

把 $a_i$ 与 $b_i$ 放在一起排序，记为 $c_i$。不难发现每一个区间 $[l,r]$，都能用 $c_r - c_l$ 表示它的极差。而如果要满足最小值尽量大，最大值尽量小，可以固定最大值 $r'$，尽量让 $l'$ 提前。

这也就意味着我们已经确定了 $[l',r']$ 内元素的值（也就是每一张牌的正反）。反过来，也确定了 $[1,l']$ 与 $[r',n]$ 的值。所以开一个标记数组 $v$，如果 $v_i = 1$ 说明第 $i$ 张牌已经被确认了，一张牌不能确认两次。

如果确定了一张牌，那么如果确定的值是正面，那么使用的翻转次数为 0，反之为 1。总的翻转次数不超过 $m$。

这样就可以用双指针求解了。维护两个指针 $l$ 和 $r$。

开始先尽可能把 $r$ 往后放，接着再尽可能把 $l$ 提前。最后不断让 $r$ 向前，贪心地提前 $l$，记录答案就好了。

## CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+5;
int n, m, ans=0x3f3f3f3f;
bool v[N];
struct card { int x, id, fg; } c[N*2];
// 数字，属于的牌的编号，翻到反面操作几次
bool operator<(card a,card b) { return a.x<b.x; }
int main() {
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n;++i) scanf("%d",&c[i].x), c[i].id=i, c[i].fg=1;
	for(int i=1;i<=n;++i) scanf("%d",&c[i+n].x), c[i+n].id=i, c[i+n].fg=0;
	sort(c+1,c+2*n+1);
	int l=1, r=2*n, cnt=0;
	while(cnt+c[r].fg<=m&&!v[c[r].id]) {
		cnt+=c[r].fg, v[c[r].id]=1;
		--r;
	}
	while(cnt+c[l].fg<=m&&!v[c[l].id]) {
		cnt+=c[l].fg, v[c[l].id]=1;
		++l;
	}
    // 有标记数组v的存在，不用担心l>r的情况
	while(r<n*2) {
		ans=min(ans,c[r].x-c[l].x);	
		cnt-=c[r+1].fg, v[c[r+1].id]=0;
		++r;
		while(cnt+c[l].fg<=m&&!v[c[l].id]) {
			cnt+=c[l].fg, v[c[l].id]=1, ++l;
		}
	}
	printf("%d\n",min(ans,c[r].x-c[l].x));
    // 这里少比较了一次，所以最后要取min

}
```

