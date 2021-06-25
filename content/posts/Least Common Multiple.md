---
title: Least Common Multiple
date: 2019-12-09 17:02:38
tags: [模拟]
categories: [数据结构与算法]
---

**Problem Description**

The least common multiple (LCM) of a set of positive integers is the smallest positive integer which is divisible by all the numbers in the set. For example, the LCM of 5, 7 and 15 is 105.<br><br>

 **Input**

Input will consist of multiple problem instances. The first line of the input will contain a single integer indicating the number of problem instances. Each instance will consist of a single line of the form m n1 n2 n3 ... nm where m is the number of integers in the set and n1 ... nm are the integers. All integers will be positive and lie within the range of a 32-bit integer.<br>

 **Output**

For each problem instance, output a single line containing the corresponding LCM. All results will lie in the range of a 32-bit integer.<br>

 **Sample Input**

```
2
3 5 7 15
6 4 10296 936 1287 792 1
```

 **Sample Output**

```
105
10296
```

 利用公式：x*y=lcm(x，y)*gcd(x，y)即可

本题需要注意点就是x乘y时可能爆int，所以可以x/gcd（x，y）再乘y防止溢出

求gcd（x，y）时，可以考虑递归辗转相除法使得代码简洁

```c++
int gcd(int x,int y)
{
	if(y>x) swap(x,y);
	if(x%y==0) return y;
	else return gcd(y,x%y);
}
```

AC代码如下：

```c++
#include<bits/stdc++.h>
using namespace std;
int gcd(int x,int y)
{
	if(y>x) swap(x,y);
	if(x%y==0) return y;
	else return gcd(y,x%y);
}
long long lcm(int  x,int y)
{
	
	return x/gcd(x,y)*y;
}
int main()
{
	int n;
	cin>>n;
	while(n--)
	{
		int len,begin;
		cin>>len;
		cin>>begin;
		int ans=begin,k;
		len--;
		while(len--)
		{
			cin>>k;
			ans=lcm(ans,k);
			
		}
		cout<<ans<<endl;
	}
}
```

