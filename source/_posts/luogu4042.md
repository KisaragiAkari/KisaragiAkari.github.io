---
title: luogu4042 骑士游戏 题解
urls: lg4042-solution
tags: DP
categories: 题解
math: true
abbrlink: 6d81b2c2
date: 2022-07-06 17:00:22
---

## 前言

好久之前就见过的题，那时候不求甚解地把代码写上，认为自己已经明白了。

之后经历的那些事情，让我越发怀疑自己对所有东西的认识。经常想起这道题，因为「偷来的 AC」，我难以原谅自己。

不过，这一切都已经过去了。那么差劲的自己，那么单纯的想法。

<!--more-->

一年来，相对于大部分人，我的进步实在是太少了。在这个人均初三 D 类 Ag 的年代，进队是不要想了。

我只想，把 OI 当作爱好。正因如此，我永远不会自称为 OIer；同时，我也永远不会再为它黯然神伤。

## 分析

DP。设 $f(i)$ 为杀死怪物 $i$ 以及它直接或间接生成的怪物的最小代价。

只有魔法攻击才能杀死怪物，$f(i)$ 的初始值为用魔法攻击杀死它的代价。

由于物理攻击杀死某个怪物后，又会生成很多怪物，而再杀死它们之后，又会再次生成其他怪物。无论怎样设计状态都必定存在环，只能从转移下手。

把怪物抽象成节点，如果杀死 $x$ 生成 $y$，那么连边 $(x \rightarrow y)$。设 $ph(x)$ 为用物理攻击杀死 $x$ 的代价。

对于 $x$，求出用魔法攻击杀死 $x$ 与生成的 $y$ 的总代价 $z$。

$$
z=ph(x) + \sum_{(x \rightarrow y)} f(y)

$$

转移



$$
f(x) = \min{\{ f(x),z \} }
$$

放在 SPFA 的过程中转移即可。

看着似乎没问题，但是有环图没有拓扑序，没有固定的转移顺序。考虑到 SPFA 中的队列，本质上是已经进行完一轮迭代之后，能够用到这些信息且不在队列中的点。那么再本题中就要将依赖于 $f(x)$ 的点加入队列，也就是将所有 $(u \rightarrow x)$ 的 $u$ 加入队列。

方法是建立一张反图，对于杀死 $x$ 生成 $y$，加边 $(y \rightarrow x)$，表示 $f(x)$ 依赖于 $f(y)$。如果更新了 $f(x)$，将反图中所有从 $x$ 出发直接到达的点加入队列即可。一开始则要把所有节点加入队列，因为每个点都能够更新其它点。

## CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=2e5+10; 
#define ll long long
int n;
int c, h[N], to[N<<3], nxt[N<<3];
int cc, hh[N], ot[N<<3], txn[N<<3];
ll f[N], ph[N];
bool v[N];
void add(int x,int y) { to[++c]=y, nxt[c]=h[x], h[x]=c; }
void dda(int x,int y) { ot[++cc]=y, txn[cc]=hh[x], hh[x]=cc; }

void spfa() {
    queue<int> q;
    for(int i=1;i<=n;++i) q.push(i), v[i]=1;
    while(q.size()) {
        int x=q.front(); q.pop();
        v[x]=0;
        ll z=ph[x];
        for(int i=h[x];i;i=nxt[i]) z+=f[to[i]];
        if(f[x]>z) {
            f[x]=z;
            for(int i=hh[x];i;i=txn[i]) if(!v[ot[i]]) v[ot[i]]=1, q.push(ot[i]);
        }
    }
}
int main() {
    scanf("%d",&n);
    for(int i=1;i<=n;++i) {
        int x, y;
        scanf("%lld%lld%d",&ph[i],&f[i],&x);
        while(x--) scanf("%d",&y), add(i,y), dda(y,i);
    }
    spfa();
    printf("%lld\n",f[1]);
}
```
