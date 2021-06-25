---
title: Dijkstra+路径还原
date: 2020-01-16 23:41:20
tags: [最短路,图论]
categories: [数据结构与算法]
---

## Dijkstra算法

适用于解决没有负边的单源最短路问题，相比于Bellman-Ford虽然有处理负边的局限，但是可以通过优先队列实现O(E logV)的复杂度。

本质其实是一种贪心思想。

我们只需要找到最短距离已经确定的顶点，从它出发更新相邻顶点的最短距离。

那么如何找到最短距离已经确定的顶点呢？

首先，初始点d[s]=0为已知最短距离已经确定的顶点。

然后，以它出发更新，那么在与起点S相邻且距离最短的点A就是可以确定的第2个最短距离点。

为什么可以确定呢？因为假设S->A的距离不是最短的，那么一定存在一个中转点B位于S->A之间。

但是S直接到A的距离是S到与其相邻点中最短的，光S->B的距离就已经大于S->A了。所以假设不成立。

那么接下来要做的就是以此类推了，

从A开始更新相邻顶点的最短距离，再从这些顶点中挑出与点A相距最短的那个顶点继续更新，知道所有顶点都被更新过为止。

那么按照图用邻接矩阵存或邻接表存该算法有两种实现方法：

**按邻接矩阵存图：**

<font color=red>初始化</font>

建图时  r[i] [j] 初始化为INF，r[i] [i]初始化为0

求最短路前 d[v]先全设为INF，d[源点]=0；

还原路径时 pre[v]可以全设为-1，方便路径还原

```c++
struct edge
{
	int to,cost;
};
int d[MAX_V],vis[MAX_V]，r[MAX_V][MAX_V];
int pre[MAX_V];
void dijkstra(int s)
{	memset(pre,-1,sizeof(pre));
	memset(d,0x3f,sizeof(d));
	memset(vis,0,sizeof(vis));
	d[s]=0;
	for(int i=0;i<n;i++)
	{	int MIN=inf;
		int p;
		for(int j=0;j<n;j++)
		{	
			if(!vis[j]&&d[j]<MIN)
			{
				p=j;
				MIN=d[j];
			}
		}//从尚未使用过的顶点中选择一个距离最小的点
		vis[p]=1;
		if(MIN==inf) break;	//所有顶点都已经用过
		for(int j=0;j<n;j++)
        {
            d[j]=min(d[j],d[p]+r[p][j]);//更新相邻顶点的最短距离
            pre[j]=p;
        }	
	}
}

```

**按邻接表存图：**

<font color=red>初始化</font>

建图的时候  记得G[MAX_V].clear( );

计算最短路前 d[v]先全设为INF，d[源点]=0；

还原路径时 pre[v]可以全设为-1，方便路径还原

```c++
#include<bits/stdc++.h>
#define PII pair<int,int> //first最短距离，second顶点编号 
#define MAX_V 100
using namespace std;
const int inf=0x3f3f3f3f;
struct edge
{
	int to,cost;
};
vector<edge>G[MAX_V];
int d[MAX_V];
int pre[MAX_V];
void dijkstra(int s)
{	priority_queue<PII,vector<PII>,greater<PII> >que;
	memset(pre,-1,sizeof(pre));
 	memset(d,0x3f,sizeof(d));
	d[s]=0;
	que.push(PII(0,s));
	while(!que.empty())
	{
		PII p=que.top();
		que.pop();
		int v=p.second;
		if(d[v]<p.first) continue;//已经不是最短距离了就舍弃
		for(int i=0;i<G[v].size();i++)
		{
			edge e=G[v][i];
			if(d[e.to]>d[v]+e.cost)
			{
			   d[e.to]=d[v]+e.cost;
			   que.push(PII(d[e.to],e.to));//更新相邻顶点距离入队
               pre[e.to]=v;
			}
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

