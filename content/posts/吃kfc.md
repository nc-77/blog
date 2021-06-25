---
title: 吃kfc
date: 2019-12-11 23:34:16
tags: [BFS]
categories: [数据结构与算法]
---

## 题目描述

众所周知，duxing哥喜欢吃kfc。

今天终于到了周二，duxing哥又可以去吃kfc了，于是他打开手机app自助点餐，发现附近有许多家kfc店，但是他想选择一家店并尽快取餐回到寝室，忽略到店取餐时间。

地图可以被看做是一个n * m的网格，其中有一些不可通过的障碍，duxing哥、kfc和寝室所在的地方也可以看做是道路，可以通过。duxing哥可以在任意一条道路中选择上下左右四个方向移动，一次移动算作一步。现从duxing初始位置出发，求取到kfc并且回到家的最少步数。

## 输入

输入包含多个测试用例。

   每个测试用例都包括前两个整数n，m。(1<=n,m<=1000)

   接下来的n行，每行包含m个字符，表示地图。

​      ‘S’表示duxing哥的初始位置；

​      ‘E’表示寝室的位置；

​      ‘＃’表示障碍；

​      ‘.’表示道路；

​      ‘K’表示kfc。

## 输出

对于每个测试用例输出一行一个整数，duxing哥带上kfc并且回到寝室需要的最少步数。如果无解，请输出“-1”。

## 样例输入

```
5 5
S..##
##..K
K..#.
#.#.K
#.K#E

5 5
S..##
##..K
K..#.
#.#.#
#.K#E

5 5
S....
.....
.....
.....
....E
```

## 样例输出

```
8
-1
-1
```

c++版本1：

由于题目要求必须经过至少一家kfc，因此最短路径就是其中某一个k到起点的最短路径和到终点的最短路径。

可以先bfs起点和终点2次来求得起点和终点到各个k的最短距离，最后加起来求得每个k的总路径，最小的即为答案。

（第一次写bfs，可能代码有些繁琐了

```c++
#include<bits/stdc++.h>
using namespace std;
const int MAX=1000+10;
char f[MAX][MAX];
struct pos
{
	int x;
	int y;
};
int dx[4]={1,0,-1,0},dy[4]={0,1,0,-1};
int dis[MAX][MAX],n,m,vis[MAX][MAX],k1[MAX][MAX],k2[MAX][MAX];//每个k到起（终）点的距离存储在k1（2）中
int bfs(pos start,int k[][MAX],int sumk)
{	 memset(dis,0,sizeof(dis));
	 memset(vis,0,sizeof(vis));
	 pos cur,nex;
	 queue<pos> que;
	 que.push(start);
	 while(!que.empty()&&sumk)
	 {	cur=que.front();que.pop();
	 	if(f[cur.x][cur.y]=='K')
	 	{
	 		k[cur.x][cur.y]=dis[cur.x][cur.y];
	 		//cout<<k[cur.x][cur.y]<<endl;
	 		sumk--;
		 }
	 	for(int i=0;i<4;i++)
	 	{
	 		nex.x=cur.x+dx[i];nex.y=cur.y+dy[i];
	 		if(f[nex.x][nex.y]!='#'&&nex.x>=0&&nex.x<n&&nex.y>=0&&nex.y<m&&!vis[nex.x][nex.y])
	 		{
	 			que.push(nex);
	 			vis[nex.x][nex.y]=1;
	 		    dis[nex.x][nex.y]=dis[cur.x][cur.y]+1;
			}
		 }
		 
	 }
	 return dis[cur.x][cur.y];
}
int main()
{
	
	while(cin>>n>>m)
	{	int min=1000010,ans=0,flag=0,sumk=0;
		pos begin1,begin2;
		for(int i=0;i<n;i++)
			for(int j=0;j<m;j++)
			{
				scanf(" %c",&f[i][j]);
				if(f[i][j]=='S')
				{
					begin1.x=i;
					begin1.y=j;
				}
				else if(f[i][j]=='E')
				{
					begin2.x=i;
					begin2.y=j;
				}
				else if(f[i][j]=='K')
				sumk++;
			}
			memset(k1,0,sizeof(k1));
			memset(k2,0,sizeof(k2));
			bfs(begin1,k1,sumk);
			bfs(begin2,k2,sumk);
			for(int i=0;i<n;i++)
			{
				for(int j=0;j<m;j++)
				{
					if(k1[i][j]&&k2[i][j])
					{ 
						flag=1;
						ans=k1[i][j]+k2[i][j];
						if(ans<=min) min=ans;
					}
				}
			}
		if(flag)cout<<min<<endl;
		else cout<<"-1"<<endl;
	}
}
```

c++版本2（感谢shn大佬提供的思路）：

可以在之前二维数组dis的基础上再添加一维来表示状态（有没有遇到kfc）

bfs状态转移前后都判断一下是否是k即可，而且这样扩充状态能保证bfs在经过kfc的条件下依然还是最短路径

附上大佬的代码

```c++
#include<cstdio>
#include<iostream>
#include<cstring>
#include<algorithm>
#include<queue>
#include<map>
using namespace std;
const int inf=0x3f3f3f3f;
const int maxn=1050;
struct node{
    int x,y,flag;
}s,e;
 
int n,m,dis[maxn][maxn][2];
char a[maxn][maxn];
int dx[]={1,-1,0,0};
int dy[]={0,0,1,-1};
bool calc(int x,int y,int z)
{
    if(x<0||x>=n||y<0||y>=m) return false;
    if(a[x][y]=='#') return false;
    if(dis[x][y][z]!=inf) return false;
    return true;
}//判断下一个坐标是否越界，障碍或已经走过
void bfs()
{
    memset(dis,0x3f,sizeof(dis));//记笔记，memset初始最大值的方法0x3f
    queue<node> q;
    q.push(s);
    dis[s.x][s.y][0]=0;
    while(!q.empty())
    {
        node t=q.front();q.pop();
        if(t.x==e.x&&t.y==e.y&&t.flag)
        {
            cout<<dis[t.x][t.y][1]<<endl;
            return ;
        }
        for(int i=0;i<4;i++)
        {
            int tx=t.x+dx[i],ty=t.y+dy[i];
            int pan=0;
            if(t.flag) pan=1;//上一个点已经过了kfc
            if(a[tx][ty]=='K') pan=1;//要走的那个点为kfc
            if(!calc(tx,ty,pan)) continue;
            if(a[tx][ty]=='K') 
            {
                q.push({tx,ty,1});
                dis[tx][ty][1]=dis[t.x][t.y][t.flag]+1;
            }
            else
            {
                q.push({tx,ty,t.flag});
                dis[tx][ty][t.flag]=dis[t.x][t.y][t.flag]+1;
            }
        }
    }
    puts("-1");
}
int main()
{
    while(~scanf("%d%d",&n,&m))
    {
    for(int i=0;i<n;i++) cin>>a[i];
    for(int i=0;i<n;i++)
    {
        for(int j=0;j<m;j++)
        {
            if(a[i][j]=='S') s={i,j};
            else if(a[i][j]=='E') e={i,j};
        }
    }
    bfs();
    }
    return 0;
}
```

