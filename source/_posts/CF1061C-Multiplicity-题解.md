---
title: CF1061C Multiplicity 题解
urls: CF1061C-solution
tags: DP
categories: 题解
math: true
abbrlink: 4bcbb89b
date: 2021-08-19 19:50:57
---

[link](https://codeforces.com/problemset/problem/1061/C)

相信大家都能秒推这道题的方程。

<!--more-->

状态：设 $ f(i,j)$ 为原序列中前 $i$ 个数选出了 $j$ 个的方案数。

转移是显然的。
$$
f(i,j)=
\begin{cases}
 f(i-1,j)+f(i-1,j-1) \quad j \mid a_i\\
f(i-1,j) \quad j \nmid a_i
\end{cases}
$$
然而这样的复杂度过高，数组也开不下。

考虑只有 $j \mid a_i$ 时状态才会被更新，否则只会将上一个 $i$ 的状态继承过来。

所以对于每个 $a_i$，能够影响方案数的仅仅只有它的约数。

将每个 $a_i$ 分解，决策集合为 $a_i$ 的约数集合。

然后用滚动数组优化掉第一维。

设 $ f(i)$ 为长度为 $i$ 的序列的方案数，由于少了一维，所以转移时要累加所有状态，同时也不需要从第  $i-1$ 个数继承状态了。

但是我这样的菜比无法写出优化后的转移方程。

设 $j$ 为 $a_i$ 的约数，将  $f(j-1)$ 累加到 $ f(j)$ 中。

大概就是这样。

&nbsp;

从「 $a_i$ 的约数 $j$ 对长度为 $j$ 的序列的方案数产生的贡献」考虑的话，我很难明白转移。

所以从 $a_i$ 产生的贡献考虑。

在原来朴素的转移方程中，对于 $a_i$，只会在每个 $j$ 处更新状态，它对总方案数的贡献是更新的这一部分。

这样新状态的转移就很好理解了。

&nbsp;

这种直接省略一维的滚动数组要注意循环顺序，确保决策从上一轮转移而来。

最后累加答案。

这道题启示我滚动数组优化仅仅是节省了空间，求解问题仍要从原状态、决策、方程考虑。DP 问题的求解要抓住并分析问题的本质。

```cpp 
#include<cstdio>
#include<iostream>
#include<algorithm>
using namespace std;
#define R register
const int N=1e6+10, mod=1e9+7;
int n, p, dv[N], f[N]={1};
int DIV(int x) {
    R int i, y=0;
    for(i=1;i*i<=x;++i) if(!(x%i)) {
        dv[++y]=i;
        if(i*i!=x) dv[++y]=x/i;
    }
    return y;
}
int main() {
    R int i, j, x, y, z;
    scanf("%d",&n);
    for(i=1;i<=n;++i) {
        scanf("%d",&x);
        y=DIV(x);
        for(j=y;j;--j) (f[dv[j]]+=f[dv[j]-1])%=mod;
    }
    for(i=1,z=0;i<=n;++i) (z+=f[i])%=mod;
    printf("%d\n",z);
}
```
