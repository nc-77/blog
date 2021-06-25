---
title: Rescue
date: 2019-12-16 23:35:30
tags: [BFS,优先队列] 
categories: [数据结构与算法]
---

 天使被MOLIGPY抓住了！他被莫利比（Moligpy）监禁。监狱被描述为N * M（N，M <= 200）矩阵。监狱中有城墙，道路和护卫队。

天使的朋友想救救天使。他们的任务是：接近天使。我们假设“接近天使”是要到达天使停留的位置。当网格中有守卫时，我们必须杀死他（或她？）才能进入网格。我们假设我们上下左右移动要花费我们1个单位时间，而杀死一名警卫也要花费1个单位时间。而且我们足够强大，可以杀死所有警卫。

您必须计算与Angel接触的最短时间。（当然，我们只能将UP，DOWN，LEFT和RIGHT移动到边界内的相邻网格中。） 

 **Input**

 第一行包含两个代表N和M的整数。然后N行，每行包含M个字符。“。” 代表道路，“ a”代表Angel，“ r”代表Angel的每个朋友。处理到文件末尾。 

 **Output**

 对于每个测试用例，您的程序应输出一个整数，代表所需的最短时间。如果不存在这样的数字，则应输出包含“可怜的ANGEL必须终生呆在监狱中”的行。 

 **Sample Input**

```
7 8
#.#####.
#.a#..r.
#..#x...
..#..#.#
#...##..
.#......
........
```

 **Sample Output**

```
13
```

本来以为是一道普通的bfs题，题目为求a到任意一个r的最短距离，后来发现事情并没有这么简单。

看下面这组数据：

 3 3
\#xr
\#..
\#a# 

如果依次加入队列的方向是上下左右的话，那么在最中间（1，1）加入队列后，先是上方（0，1）x加入队列，再是右边（1，2）加入队列，x先入队列会导致下次以x为起点宽搜时直接找到r并返回答案4。显然正确答案应该为3.

那么有没有什么办法来保证每次都是先去搜没有经过x的路呢，也就是每次step少的数先出队列，答案是有的。就是这道题的主角——优先队列。

## 相关定义

优先队列容器与队列一样，只能从队尾插入元素，从队首删除元素。但是它有一个特性，就是队列中最大的元素总是位于队首，所以出队时，并非按照先进先出的原则进行，而是将当前队列中最大的元素出队。这点类似于给队列里的元素进行了由大到小的顺序排序。元素的比较规则默认按元素值由大到小排序，可以重载“<”操作符来重新定义比较规则。 

## 基本操作

empty() 　　  如果队列为空，则返回真

pop()　　　　删除对顶元素，删除第一个元素

push() 　　   加入一个元素

size() 　　　  返回优先队列中拥有的元素个数

top() 　　　　返回优先队列对顶元素，返回优先队列中有最高优先级的元素

与普通队列类似，唯一有区别的地方就是front（）替换成了top（）

##  结构体声明方式 

 struct node {   

　　int x, y;   

　　friend bool operator < (node a, node b)   

　　{     

　　　　return a.x > b.x;  //结构体中，x小的优先级高   

　　}

};

priority_queue<node>q;

  （对于友员函数，重载什么的数据结构内容笔者还不够熟悉，所以不做过多说明，反正先用着，以后再来补坑）

介绍完优先队列，那么这道题就迎刃而解了，只需要在原来的bfs基础上把队列换成优先队列，让步数少的先出列就行了。

AC代码：

```c++
#include<bits/stdc++.h>
using namespace std;
int n,m,vis[205][205];
char a[205][205];
int dx[]={-1,1,0,0},dy[]={0,0,1,-1};
struct pos
{
	int x,y;
	int step;
	friend bool operator <(pos t1,pos t2)
	{
		return t1.step>t2.step;//step小的在前 
	}
}cur,nex;
bool pd()
{
	if(nex.x<=n-1&&nex.x>=0&&nex.y>=0&&nex.y<=m-1&&a[nex.x][nex.y]!='#'&&!vis[nex.x][nex.y])
	return true;
	return false;
}
void bfs()
{
	cur.step=0;
	priority_queue<pos>que;
	que.push(cur);
	while(!que.empty())
	{
		cur=que.top();
		que.pop();
		if(a[cur.x][cur.y]=='r')
		{
			cout<<cur.step<<endl;
			return;
		}
		for(int i=0;i<4;i++)
		{
			nex.x=cur.x+dx[i];
			nex.y=cur.y+dy[i];
			//printf("%d %d\n",nex.x,nex.y);
			if(pd())
			{	vis[nex.x][nex.y]=1;
				//printf("%d %d\n",nex.x,nex.y);
				if(a[nex.x][nex.y]=='x')
				{
					nex.step=cur.step+2;
				}
				else nex.step=cur.step+1;
				que.push(nex);
			}
			
		}
	}
cout<<"Poor ANGEL has to stay in the prison all his life."<<endl;
}
int main()
{
	while(~scanf("%d%d",&n,&m))
	{	
		//printf("here\n");
		for(int i=0;i<n;i++)
			for(int j=0;j<m;j++)
			{	vis[i][j]=0;
				//printf("here\n");
				scanf(" %c",&a[i][j]);
				if(a[i][j]=='a')
				{
					cur.x=i;
					cur.y=j;
					vis[i][j]=1;
				}
			}
		
		bfs();
	}
return 0;
}
```

