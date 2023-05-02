---
title: luogu5008 锦鲤抄 题解
urls: lg5008-solution
tags:
  - 图论
  - 强连通分量
  - DAG
categories: 题解
math: true
abbrlink: 4d331a68
date: 2021-09-20 17:26:48
---

## update 2022.6.28 修改了证明部分

在 DAG 中，必然有入度为 0 的点，这些点是不能选择的。而其他的点则可以选择。

用 Tarjan 算法缩点，得到一个 DAG。

<!--more-->

讨论每个 SCC 内部的选择。

若存在入度为不为 0 的 SCC，设其为 $x$。

那么 $x$ 内的点可以随便选。

证明：

>如果 $x$ 的入度不为 0，分以下情况讨论。
>
>对于 $x$ 中只有 1 个点的情况，显然成立。
>
>否则，$x$ 至少由一个简单环构成，且一定存在入度 $\ge 2$ 的点，设其为 $u$。
>
>对于 $x$ 中的每个简单环，设其入度最大的点为 $ u$，则至少可以删去 1 条 $u$ 的入边，断开这个简单环。而只要 $u$ 的入度不为 0，就不会影响到点的选择。最终可以得到一个多了若干条入边的 DAG。
>
>因为 $x$ 入度不为 0，所以这个 DAG 不存在入度为 0 的点，那么都可以删去，命题得证。

&nbsp;

考虑入度为 0 的 SCC，设其为 $y$。

>
>仿照上述证明思路，我们仍以 $y$ 内每个简单环删去若干边为代价，得到一个 DAG。
>
>
>它是一个普通 DAG，而我们不能选择删去一个入度为 0 的点，设其为 $v$。
>
>通过不同的修改简单环能够让不同的节点入度为 0，所以最优解让入度为 0 的节点权值最小，除了它其他的都能删掉。
>
>更进一步地说，以上关于 SCC 的删边策略，最终得到的都是 SCC 的搜索树，搜索树一定是 DAG。
>
>不同的是前者没有入度为 0 的点，后者有且仅有一个。

所以我们按照上述操作之后，把能够选择的点集（决策集合）排个序，取前 $k$ 大就可以。

## CODE

```cpp 
#include<cstdio>
#include<cstring>
#include<iostream>
#include<algorithm>
#include<vector>
using namespace std;
const int N=5e5+6, M=2e6+6;
int n, m, k, num, tp, dfn[N], low[N], st[N];
int scc, o, c[N], w[N], deg[N], f[N], ans[N];
int cnt, h[N], ver[M], nxt[M];
vector<int> p[N];
void add(int x,int y) { ver[++cnt]=y, nxt[cnt]=h[x], h[x]=cnt; }
void tarjan(int x) {
    dfn[x]=low[x]=++num, st[++tp]=x;
    for(int i=h[x];i;i=nxt[i]) {
        int y=ver[i];
        if(!dfn[y]) {
            tarjan(y);
            low[x]=min(low[x],low[y]);
        } else if(!c[y]) low[x]=min(low[x],dfn[y]);
    }
    if(dfn[x]==low[x]) {
        ++scc;
        int y;
        do {
			y=st[tp--], c[y]=scc, f[scc]=min(f[scc],w[y]);
            // f[scc]表示scc里面最小的点权
            p[scc].push_back(y);
        } while(x!=y);
    }
}
void sol() {
    for(int x=1;x<=n;++x) for(int i=h[x];i;i=nxt[i]) {
        int y=ver[i];
        if(c[x]!=c[y]) ++deg[c[y]];
    }
    for(int x=1;x<=scc;++x) {
        bool fg=(deg[x]==0);
        for(int i=0;i<p[x].size();++i) {
            int y=p[x][i];
            if(w[y]==f[x]&&fg) fg=0; else ans[++o]=w[y];
            // 当w[y]==f[x]且fg==1，保留入度为0的点y，因为它的点权最小
        }
    }
}
int main() {
    int Ans=0;
    memset(f,0x3f,sizeof(f));
    scanf("%d%d%d",&n,&m,&k);
    for(int i=1;i<=n;++i) scanf("%d",&w[i]);
    for(int i=1,x,y;i<=m;++i) scanf("%d%d",&x,&y), add(x,y);
    for(int i=1;i<=n;++i) if(!dfn[i]) tarjan(i);
    sol();
    if(o<=k) for(int i=1;i<=o;++i) Ans+=ans[i];
    else { sort(ans+1,ans+o+1); for(int i=o-k+1;i<=o;++i) Ans+=ans[i]; }
    printf("%d\n",Ans);
}
```