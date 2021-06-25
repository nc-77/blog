---
title: Floyd-Warshall+路径还原
date: 2020-01-16 23:14:33
tags: [最短路,图论]
categories: [数据结构与算法]
---

## Floyd-Warshall算法

适用于求解任意两点间的最短路问题。

本质思想还是dp。

在只使用了0-k个顶点和i,j的情况下，我们记i到j的最短路长度为d[k] [i] [j]。

那么只使用0-k时，i到j的最短路情况可分为正好经过k点一次和完全不经过顶点k。

不经过顶点k的情况下，**d[k] [i] [j]=d[k-1] [i] [j]**；

经过顶点k的情况下，**d[k] [i] [j]=d[k-1] [i] [k]+d[k-1] [k] [j]**.

合起来，就得到了**d[k] [i] [j]=min{d[k-1] [i] [j]，d[k-1] [i] [k]+d[k-1] [k] [j]}**，那么其实只要使用同一个数组，不断进行**d[k] [i] [j]=min{d [i] [j]，d [i] [k]+d[k] [j]}**的更新就好了。

<font color=red>初始化</font>

建图的时候 d[i] [j] 先全设为INF（表示两点之间没有路）,d[i] [i]设为0；

### **还原路径**

**记录前趋**：如同Bellman和dijkstra算法一样，Floyd也可以通过记录前趋来还原路径。

用pre[i] [j]来表示以i为起点，j的前趋。先初始化pre[i] [j]=i,同时pre[i] [i]=-1,方便还原路径。之后每次更新最短路更新pre[i] [j]=pre[k] [j]即可。

**记录后趋**：

不过其实floyd可以完全使用记录后趋的方法，还能省去记录前趋后还原的步骤，因此<font color=red>推荐</font>使用这种方法。

用pre[i] [j]来表示以j为终点，i的后趋。先初始化pre[i] [j]=j,同时pre[j] [j]=-1,方便还原路径。之后每次更新最短路更新pre[i] [j]=pre[i] [k]即可。

```c++
int d[MAX_V][MAX_V];
int V;
int pre[MAX_V][MAX_V];
void floyd_warshall()
{	
    for(int k=0;k<V;k++)
        for(int i=0;i<V;i++)
            for(int j=0;j<V;j++)
            {
                d[i][j]=min(d[i][j],d[i][k]+d[k][j]);
                pre[i] [j]=pre[i] [k]；//记录后趋
             }
}
void init()
{
    for(int i=0;i<V;i++)
        for(int j=0;j<V;j++)
        {
            if(i==j) pre[i][j]=-1;
            else pre[i][j]=j;
        }
}
void getpath(int st,int ed)
{	int t;
 `	printf("%d",st);
    while(t != -1)
            {
                printf("-->%d",t);
                t = p[t][ed];
            }
}
```

代码如此简洁，时间复杂度就大了，有O(V^3)呢！

可适用于有负边的情况，如果要判断图中是否有负圈，只需检查是否存在d[i] [i]是负数的顶点即可。