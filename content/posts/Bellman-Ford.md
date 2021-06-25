---
title: Bellman-Ford+路径还原
date: 2020-01-16 22:29:59
tags: [最短路,图论]
categories: [数据结构与算法]
---

## Bellman-Ford 算法

适用于解决单源最短路问题，即固定一个起点，求它到其他所有点的最短路的问题。

本质思想为dp。

记从起点s出发到顶点i的最短距离为d[i]。

则有**d[i]=min{d[j]+从j直接到i边的距离}**

那么我们只要初始化d[s]=0，d[i]=INF，再不断使用这条递推关系式更新d[i]。只要图中没有负圈，这样的更新操作一定是有限的。结束之后的d[i]就是最短距离了。

<font color=red>初始化</font>

建图的时候 e.cost 先全设为INF（表示两点之间没有路）

dp前 d[v]先全设为INF，d[源点]=0；

还原路径时 pre[v]可以全设为-1，方便路径还原


```c++
struct edge
{
    int from,to,cost;
};
edge es[MAX_E];
int d[MAX_V],pre[MAX_V],path[MAX_V];
int V,E;//v顶点数 E边数
int Bellman-Ford（int s）
{	memset(pre,-1,sizeof(pre));
    memset(d,0x3f,sizeof(d));
    d[s]=0;
    for(int i=0;i<V;i++)
    {	int flag=0;
        for(int j=0;j<E;j++)
        {
            edge e=es[j];
            if(d[e.to]>d[e.from]+e.cost)
            {
                d[e.to]=d[e.from]+e.cost;
                pre[e.to]=e.from;
                flag=1;
            }
            if(i==v-1&&flag==1)  return -1;//如果第v次更新了，则存在负圈
            if(flag==0) return 1;//flag=0 此次没更新，表示更新已经结束，直接退出
        }
    }
}
void getpath(int t) //还原路径
{
	int i;
	for( i=0;i<=n;i++)
	{	path[i]=t;
		t=pre[t];
		if(t==-1) break;
	}
	reverse(path,path+i);   
}
```

这里解释一下为什么外循环至多只要执行v-1次。

如果在图中没有负圈，那么任何一条最短路径都不会经过同一个顶点两次（也就是说，最多通过v-1条边）

对于外循环而言，最差的情况（即线路是从D->C->B->A倒着给的）每次也能更新一条边，所以最多只要循环v-1次。

因此Bellman-Ford算法的复杂度为**O（V*E）**，而且可以适用于存在负边的情况。