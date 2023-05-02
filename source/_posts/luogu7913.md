---
title: luogu7913 廊桥分配 题解
urls: lg7913-solution
tags: 贪心
categories: 题解
math: true
abbrlink: 22ae602b
date: 2022-06-20 16:25:56
---

## 前言

我几乎搞砸了 2021 年 8 月 31 日之后的所有事。

无论是 OI 与 文化课，还是人际关系与整个自己。

我根本不配参加 CSP 与 NOIP，最终的结果也在预料之中。更何况在那之后的

那些煎熬着，逃避着的日子啊……

<!--more-->

## 分析

不难发现国际区与国内区完全没有关系，分开处理就行了。

注意到当某个区只有一个廊桥的时候，就等价于保证在选择第一个区间的情况下，选择最多数量的不相交区间。

那么如果廊桥数量更多呢？

显然如果在廊桥数量为 1 时选择了最多数量的不相交区间，那么当廊桥数量为 2 时仍然能够选取这么多，方法是钦定选出来的飞机只能用第 1 个廊桥。多出来的这个廊桥又能够再对剩下的的飞机做一次选择最多数量的不相交区间。这样一定是最多的。

归纳一下就得到

- 廊桥越多，所容纳的飞机数量单调不减，且具有最优子结构性质，可以直接贪心。
- 如果某个区有 $n_0$ 个廊桥，那么最多容纳的飞机数量就是做 $n_0$ 次选择最多不相交区间。

因为只有 $n$ 个廊桥，那么国内去分 $n_0$ 个，国外区一定有 $n - n_0$ 个。这样只要预处理出每个区分配 $k \in[0,n]$ 个廊桥最多能容纳的飞机数量，那么就能 $O(n)$ 取最大值了。

维护每个区的飞机用`std::set`，它本身是有序的，自带`lower_bound`而且还支持删除操作。

廊桥分配情况是 $O(n)$ 的，每个飞机最多进出`set`一次。

复杂度 $O(n \log_2 n)$。

## CODE

```cpp
#include<cstdio>
#include<iostream>
#include<set>
using namespace std;
const int N=1e5+6;
int n, c[2], ans[2][N];
set<pair<int,int> > s[2];
int main() {
	scanf("%d%d%d",&n,&c[0],&c[1]);
	for(int i=0;i<=1;++i) for(int j=1;j<=c[i];++j) {
		int l, r; scanf("%d%d",&l,&r);
		s[i].insert(make_pair(l,r));
	}
	for(int k=0;k<=1;++k) for(int i=1;i<=n;++i) {
		ans[k][i]=ans[k][i-1];
        // 先继承i-1个廊桥的数量
		int lst=0;
		if(!s[k].size()) continue;
		while(1) {
			auto p=s[k].upper_bound(make_pair(lst,0));
			if(p==s[k].end()) break;
			lst=p->second, ++ans[k][i];
			s[k].erase(p);
		}
	}
	int res=0;
	for(int i=0;i<=n;++i) res=max(res,ans[0][i]+ans[1][n-i]);
	printf("%d\n",res);
}
```
