---
title: Lake Counting
date: 2019-12-12 00:04:41
tags: [DFS]
categories: [数据结构与算法]
---

Description

Due to recent rains, water has pooled in various places in Farmer John's field, which is represented by a rectangle of N x M (1 <= N <= 100; 1 <= M <= 100) squares. Each square contains either water ('W') or dry land ('.'). Farmer John would like to figure out how many ponds have formed in his field. A pond is a connected set of squares with water in them, where a square is considered adjacent to all eight of its neighbors.

Given a diagram of Farmer John's field, determine how many ponds he has.

Input

\* Line 1: Two space-separated integers: N and M

\* Lines 2..N+1: M characters per line representing one row of Farmer John's field. Each character is either 'W' or '.'. The characters do not have spaces between them.

Output

\* Line 1: The number of ponds in Farmer John's field.

Sample Input

```
10 12
W........WW.
.WWW.....WWW
....WW...WW.
.........WW.
.........W..
..W......W..
.W.W.....WW.
W.W.W.....W.
.W.W......W.
..W.......W.
```

Sample Output

```
3
```

dfs入门题。

先找到任意一个W，不停地把其邻接部分用'.'代替，在代替的过程中如果碰到W把其邻接部分也用'.'代替。这样一次dfs完就代表一处水洼，到最后遍历完整个方阵没有W时，dfs的次数就是水洼数。

AC代码如下：

```c++
#include<bits/stdc++.h>
using namespace std;
const int MAXN=200;
int n,m;
char f[MAXN][MAXN];
void dfs(int x,int y)
{
	f[x][y]='.';
	for(int i=-1;i<=1;++i)
	{
		for(int j=-1;j<=1;++j)//i，j控制遍历方向
		{	int nx=x+i,ny=y+j;
			if(nx>=0&&nx<n&&ny>=0&&ny<m&&f[nx][ny]=='W')
			dfs(nx,ny);
		}
	}
}
int main()
{
	int ans=0;
	cin>>n>>m;
	for(int i=0;i<n;i++)
		for(int j=0;j<m;j++)
		cin>>f[i][j];
	for(int i=0;i<n;i++)
	{
		for(int j=0;j<m;j++)
		{
			if(f[i][j]=='W')
			{
				dfs(i,j);
				ans++;
			}
		}
	}
	cout<<ans<<endl;
return 0;	
} 
```

