---
title: luogu6185 序列 题解
urls: lg6185-solution
tags:
  - 二分图
  - 并查集
categories: 题解
math: true
abbrlink: 64870d9d
date: 2022-05-01 11:20:46
---

## 分析

~~好像这种加一减一无限次操作的都有差不多的套路。~~

把每个位置 $i$ 看作点，把 $\{a_i\}$ 与 $\{b_i\}$ 做差作为点权。

<!--more-->

对于 2 操作，一加一减，那么它们的和不变。所以对于 2 操作的 $(u,v)$，从 $u$ 向 $v$ 连一条边。这样，在每一个连通块中，无论如何操作，所有点权的总和不变。按照连通块缩点 ，用并查集维护即可。

对于 1 操作，同时加减 $1$，那么他们的奇偶性同时变化。在缩点后的图中，对于操作 1 的 $(u,v)$，从 $f_u$ 向 $f_v$ 连一条边。（也就是缩点后它们所在的连通块）

将这张图黑白染色，如果这张图是二分图，那么由于每对点都只能同加减，若左部点权和不等于右部点权和，那么肯定不能把点权全部变为 $0$。

如果不是二分图，由于不同连通块之间只能同时改变奇偶性，所以总权值和的奇偶性不变。那如果总点权是奇数，也不能全部变为 $0$。

## CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N=1e5+5;
int t, n, m, cnt, f[N], u[N], v[N], a[N], b[N], vis[N];
ll s[N], c[3];
vector<int> p[N];
int get(int x) { return x==f[x]? x:f[x]=get(f[x]); }
void init() {
    cnt=0;
    for(int i=1;i<=n;++i) f[i]=i, s[i]=vis[i]=0, p[i].clear();
}
bool dfs(int x,int k) {
    vis[x]=k, c[k]+=s[x];
    bool fg=1;
    for(auto y:p[x]) {
        if(vis[y]==k) fg=0;
        if(!vis[y]&&!dfs(y,3-k)) fg=0;
    }
    return fg;
}
bool solve() {
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;++i) scanf("%d",&a[i]);
    for(int i=1;i<=n;++i) scanf("%d",&b[i]);
    init();
    for(int i=1;i<=m;++i) {
        int op, x, y; scanf("%d%d%d",&op,&x,&y);
        if(op==2) f[get(x)]=get(y); else u[++cnt]=x, v[cnt]=y;
    }
    for(int i=1;i<=n;++i) s[get(i)]+=b[i]-a[i];
    for(int i=1;i<=cnt;++i) {
        int x=get(u[i]), y=get(v[i]);
        p[x].push_back(y), p[y].push_back(x);
    }
    for(int i=1;i<=n;++i) if(get(i)==i&&!vis[i]) {
        c[1]=c[2]=0;
        bool fg=dfs(i,1);
        if(fg&&c[1]!=c[2]) return 0;
        if(!fg&&(c[1]+c[2])&1) return 0;
    }
    return 1;
}
int main() {
    scanf("%d",&t);
    while(t--) if(solve()) puts("YES"); else puts("NO");
}
```
