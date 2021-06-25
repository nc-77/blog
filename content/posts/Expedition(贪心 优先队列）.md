---
title: Expedition
date: 2020-01-09 22:21:12
tags: [贪心,优先队列]
categories: [数据结构与算法]
---

## **From**

poj 2431

## **Description** 

 一群母牛抓住卡车，冒险进入丛林深处。不幸的是，作为母牛的司机，这些母牛设法跑过一块岩石，刺穿了卡车的油箱。现在，卡车每行驶一段距离，就会泄漏一单位燃油。

要修理卡车，奶牛需要沿着一条蜿蜒曲折的道路驶向最近的城镇（相距不超过1,000,000单位）。在这条道路上，在城镇和卡车的当前位置之间，有N个（1 <= N <= 10,000）加油站，奶牛可以停下来获取更多的燃料（每站1.100单位）。

丛林对人类来说是一个危险的地方，对奶牛来说尤其危险。因此，奶牛希望在前往小镇的途中尽可能少地停下加油站。幸运的是，他们的卡车上的油箱容量很大，以至于它可以容纳的燃油量实际上没有限制。卡车目前离镇区L单位，有P单位燃料（1 <= P <= 1,000,000）。

确定到达城镇或奶牛根本无法到达城镇所需的最少停靠站数。 

##  **Input** 

 *第1行：一个整数，N

*第2..N + 1行：每行包含两个以空格分隔的整数，用于描述加油站：第一个整数是从城镇到停靠站的距离；第二个是该站的可用燃料量。

*第N + 2行：两个以空格分隔的整数L和P 

##  **Output** 

 *第1行：一个整数，给出到达市区所需的最少燃料停止数量。如果无法到达该镇，则输出-1。 

##  **Sample Input** 

```
4
4 4
5 2
11 5
15 10
25 10
```

##  **Sample Output** 

首先这道题题目的输入是终点到加油站的距离而不是起点到加油站的距离，而且数据并不是一定从小到大的输入。因此要先转化一下并sort。（被坑惨了）

接下来这题只要转化一下思路就行了，因为燃料箱是无限的，所以可以把“路过一个加油站是否加油”等价于“路过一个加油站，在之后永久获得一次随时加油的机会”。

那么只要保证每次油不够时选择已经路过加油站中最大的加油即可，因为每次都是选择最大，所以可以考虑优先队列来实现。

AC代码：

```c++
#include<cstdio>
#include<iostream>
#include<algorithm>
#include<queue>
using namespace std;
int n,l,p;
struct stop
{
    int dis;
    int fue;
}s[10000+5];
bool cmp(stop s1,stop s2)
{
	return s1.dis<s2.dis;
}
void solve()
{
	int tank=p,pos=0,ans=0;
	s[n].dis=l,s[n].fue=0; 
	priority_queue<int>que;
	for(int i=0;i<=n;i++)
	{
		int d=s[i].dis-pos;
		while(tank<d)
		{
			if(que.empty())
			{
				puts("-1");
				return;
			}
			tank=tank+que.top();
			//cout<<que.top()<<endl;
			que.pop();
			ans++;
		}
		tank-=d;
		pos=s[i].dis;
		que.push(s[i].fue);
	}
	cout<<ans<<endl;
}
int main()
{
	cin>>n;
	for(int i=n-1;i>=0;i--)
	{
		cin>>s[i].dis>>s[i].fue;
		
	}
	cin>>l>>p;
	for(int i=0;i<n;i++)
	{
		s[i].dis=l-s[i].dis;
		//cout<<s[i].dis<<" "<<s[i].fue<<endl;
	}
	sort(s,s+n,cmp);
	solve();
} 
```

