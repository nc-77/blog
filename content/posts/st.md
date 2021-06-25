---
title: ST&LCA
date: 2020-05-05 16:55:50
tags: [ST,LCA]
categories: [数据结构与算法]
---

## ST表

### 引言

给出一个长度为$n$的数组，再给出$m$个询问,每次询问任意区间$[l,r]$的最小值。

这是一个常见的RMQ问题。显然当n,m为$10^5$量级时，暴力$n^2$算法行不通。

当然这种可以区间合并的问题我们都可以用线段树来解决（$nlogn$),而且线段树应用范围还会比ST倍增算法更广。

但是由于当m很大的时候线段树可能会被卡常（不会真的有出题人这么丧心病狂吧），同时引入LCA的倍增求法，还是来写下ST倍增算法吧。

### 正文

首先，要求一个任意区间$[l,r]$，如果我们知道两个区间$[l,k]$和$[k,r]$,同时保证这两个区间能完全覆盖$[l,r]$，那么这两个区间的min再min一下就可以了。

我们先预处理一下，用$dp[i][j]$表示区间起始点为$i$，长度为$2^j$的区间的最小值（倍增算法的核心）

显然$dp[i][0]$应初始化为$a[i]$

同时可以得到dp的递推公式 $dp[i][j]=min(dp[i][j-1],dp[i+(1<<(j-1))][j-1])$

$dp[i][j-1]$表示前半部分，$dp[i+(1<<(j-1))][j-1])$表示后半部分。

那么怎么查询任意区间呢？我们不妨令$k=log2(r-l+1)$，显然$\frac{r-l+1}{2}<=2^k<r-l+1$

这样就回到一开始的问题可以实现$O1$查询了。

同理RMQ还可以用来查询区间gcd等区间合并会有重复但不影响结果的东西。

贴下模板

```c++
void stinit(int n){
    for(int i=1;i<=n;i++) dp[i][0]=a[i];
    for(int j=1;(1<<j)<=n;j++)
        for(int i=1;i+(1<<j)-1<=n;i++)
        dp[i][j]=max(dp[i][j-1],dp[i+(1<<(j-1))][j-1]);
}
int query(int l,int r){
    int k=log2(r-l+1);
    int res=max(dp[l][k],dp[r-(1<<k)+1][k]);
    return res;
}
```

## LCA

 给定一棵有根多叉树，请求出指定两个点直接最近的公共祖先。 

<img src="https://i.loli.net/2020/05/05/r2mh4i7Jc8QIWVo.png" alt="lca.png" style="zoom: 67%;" />

如果询问只有一次，很简单，我们只需要顺着两个点往上走，求出两条链，然后最早在两条链中均有出现的点就是最近公共祖先。比如说3，5的最近公共祖先。两条链为3->1->4, 5->1->4，所以3，5的lca为1。

但当询问次数一大，这种暴力方法显然不可取了。

所以我们可以考虑用上面的ST表来加速这种暴力方法。

比如说要求节点u，v的最近公共祖先

我们先dfs一遍求出u，v到根的距离$dis$，接下来把节点较深的那个移至和另一个节点dis相同的距离。

一步一步移太慢了，由于任意一个数都可以由二进制来表示，我们每次走$2^i$的步，$i$同时减小就可以。（$i$的初值可以设为$log2(dis[u])$或一个比较大的常数，比如20。

 注意移到相同层次时会有一个特殊情况，即v是u的祖先，这种情况直接返回v即可。

一般情况下，移动同一层次后，若u往上走$2^i$步与v往上走$2^i$步不为同一个点，则将它们同时往上走$2^i$步,这样移动后就能保证father[u]=father[v]（这里相当于用二进制来表示dis[u]到dis[lca(u,v)]的距离）

那么怎么用代码来实现移动呢？

可以发现我们移动的步数都是$2^i$步，因此可以用st表来预处理出$p[i][j]$表示编号为$i$的节点向上$2^j$单位的祖先节点。

贴个模板：

```c++
#include<bits/stdc++.h>
#define ll long long
#define debug(x) cout<<#x<<'='<<x<<endl
#define set0(x) memset(x,0,sizeof(x))
using namespace std;
const int inf=0x3f3f3f3f;
const int maxn=5e5+10;
vector<int>g[maxn];
int dis[maxn],p[maxn][25],par[maxn];
void dfs(int u,int fa){
    par[u]=fa;
    for(int i=0;i<g[u].size();i++)
    {
        if(g[u][i]==fa) continue;
        dis[g[u][i]]=dis[u]+1;
        dfs(g[u][i],u);
    }
}
void lca(int n,int s){
    memset(dis,0,sizeof(dis));
    dis[s]=1;dfs(s,0);
    memset(p,-1,sizeof(p));
    for(int i=1;i<=n;i++) p[i][0]=par[i];
    for(int j=1;(1<<j)<=n;j++)
        for(int i=1;i<=n;i++){
            if(p[i][j-1]!=-1)
            p[i][j]=p[p[i][j-1]][j-1];
        }
}
int query(int u,int v){
    if(dis[u]<dis[v])  swap(u,v);//深度深的设为u
    //int log=log2(dis[u]);
    for(int i=20;i>=0;i--)
        if(dis[p[u][i]]>=dis[v]) u=p[u][i];//移至同一层
    if(u==v) return u;
    for(int i=20;i>=0;i--){
        if(p[u][i]!=-1&&p[u][i]!=p[v][i]){
            u=p[u][i];v=p[v][i];
        }
    }
    return par[u];
}
int main()
{
   int n,m,s;
   cin>>n>>m>>s;
   for(int i=1;i<n;i++){
       int u,v;
       cin>>u>>v;
       g[u].push_back(v);
       g[v].push_back(u);
   }
   lca(n,s);
   while(m--)
   {
       int u,v;
       cin>>u>>v;
      cout<<query(u,v)<<endl;
   }
   return 0;
}

```

