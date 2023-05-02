---
title: luogu4381 Island 题解
urls: lg4381-solution
tags:
  - 基环树
  - 单调队列
categories: 题解
math: true
abbrlink: '12412137'
date: 2021-10-01 21:12:35
---

不难发现，给出的是一个基环树森林。

渡船的使用条件，实际上是：离开一颗基环树后，就不能再回来。

题目要求走过的路最长，不难想到是基环树的直径。

<!--more-->

显然基环树的直径有两种可能

1. 在去掉环后的某棵子树中（若有负边权）
2. 两端在在去掉环后的两颗子树中 / 把环断开后，树的直径

（但在本题中只有第二种就是了）

最终答案为每颗基环树直径的和。

对于第一种情况，找到环后，以环上的每个节点为根，在它的子树中跑 DP 求最长链就行了。

设以环上节点 $ x$ 为根，不经过换上节点，能够到达的最远距离为 $d(x)$，两点间距离为 $dis(x,y)$。

第二种情况仅仅是：选择两个环上的点 $ (i,j)$，最大化 $d(i)+d(j)+dis(i,j)$。

考虑环形 DP 的处理方案，我们将环断开并复制一倍，用单调队列优化点的枚举。

用前缀和处理两点间的距离，设其为 $S$。

设环为 $ u$。

在队头维护：满足 $d(u_i)+d(u_j)+S(u_j)-S(u_i)$ 的最大的 $ u_j$。

在队尾维护：$ d(u_r)-S(u_r)$ 单调减。

环上两点有顺时针和逆时针两个距离，本题无负权，所以其中一个距离一定大于另一个。

若环上有 $ p$ 个节点，则在队头排除距离小于 $ i-p$ 的决策。

实现时注意细节。

```cpp 
#include<bits/stdc++.h>
using namespace std;
#define R register
#define ll long long
const int N=1e6+5;
int n, num, p, cnt;
int dfn[N], fr[N], s[N<<1], q[N<<1];
int h[N], nxt[N<<1], ver[N<<1], w[N<<1];
bool v[N];
ll ans, ANS;
ll d[N], st[N<<1];
void add(int x,int y,int z) { ver[++cnt]=y, nxt[cnt]=h[x], w[cnt]=z, h[x]=cnt; }
void init() {
    R int x, y, z;
    scanf("%d",&n);
    for(x=1;x<=n;++x) scanf("%d%d",&y,&z), add(x,y,z), add(y,x,z);
}
void fc(int x,int y,int z) {
    R int i;
    st[1]=z;
    while(x!=y) s[++p]=y, st[p+1]=w[fr[y]], y=ver[fr[y]^1];
    s[++p]=x;
    for(i=1;i<=p;++i) v[s[i]]=1, s[p+i]=s[i], st[p+i]=st[i];
    for(i=1;i<=p<<2;++i) st[i]+=st[i-1];
}
void dfs(int x) {
    R int i, y;
    dfn[x]=++num;
    for(i=h[x];i;i=nxt[i]) {
        y=ver[i];
        if(!dfn[y]) fr[y]=i, dfs(y);
        else if((i^1)!=fr[x]&&dfn[y]>dfn[x]) fc(x,y,w[i]);
    }
}
void DP(int x) {
    R int i, y, z;
    v[x]=1;
    for(i=h[x];i;i=nxt[i]) if(ver[i]!=fr) {
        y=ver[i];
        DP(y);
        ans=max(ans,d[x]+d[y]+z), d[x]=max(d[x],d[y]+z);
    }
}
void korekara(int x) {
    R int i;
    p=ans=0;
    dfs(x);
    for(i=1;i<=p;++i) DP(s[i]);
    R int l=1, r=0;
    for(i=1;i<=p<<1;++i) {
        while(l<=r&&q[l]<=i-p) ++l;
        ans=max(ans,d[s[i]]+d[s[q[l]]]+st[i]-st[q[l]]);
        while(l<=r&&d[s[q[r]]]-st[q[r]]<=d[s[i]]-st[i]) --r;
        q[++r]=i;
    }
    ANS+=ans;
}
int main() {
    cnt=1;
    init();
    for(R int i=1;i<=n;++i) if(!dfn[i]) korekara(i);
    printf("%lld\n",ANS);
}
```
