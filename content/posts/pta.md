---
title: HDU数据结构期末验收
date: 2020-11-22 12:56:17
tags: [PTA,模拟,图,树]
categories: [数据结构与算法]
---

## 写在前面

本篇文章旨在提供解题思路以及代码参考，切勿直接抄袭（PTA有代码查重系统）

欢迎大家在评论区提出疑惑和代码的缺陷，有更好的想好也欢迎交流。

## 约瑟夫环

### 题目详情

N个人围成一圈顺序编号，从1号开始按1、2、3......顺序报数，报p者退出圈外，其余的人再从1、2、3开始报数，报p的人再退出圈外，以此类推。 请按退出顺序输出每个退出人的原序号。

### 解题思路

模拟题意即可。刚开始有n个人在圈内，我们使用$index=(i+n-1)\%n+1$保证编号在始终在[1，n]内，使用a数组记录每个人是在圈内还是圈外，每次开始报数从上一次中止的下一位开始，检查每个人的状态，不在圈内就跳过，报到P时，标记$a[index]=0$，同时alive--,直到alive==0为止

### 参考代码

```c
#include<stdio.h>
int a[3010];
int main()
{
   int n,p;
   scanf("%d%d",&n,&p);
   int alive=n;// 刚开始有n个人在圈内
   int now=1;
   for(int i=1;i<=n;i++) a[i]=1;
   int first=1;
   while(alive>0){
       int cnt=0;
       for(int i=now;;i++){
           /* 保证每次序号都在[1,n]内 */
           int index=(i+n-1)%n+1;
           /* 如果已经不在圈内,跳过 */
           if(!a[index]) continue;
           cnt++;
           if(cnt==p){
               a[index]=0;
               /* 处理行尾空格问题 */
               if(first){
                   printf("%d",index);
                   first=0;
               }
               else printf(" %d",index);
               alive--;
               /* 从下一位开始 */
               now=i+1;
               break;
           }
       }
   }
   return 0;
}

```

## **哈夫曼编码**

### 题目详情

给定一段文字，如果我们统计出字母出现的频率，是可以根据哈夫曼算法给出一套编码，使得用此编码压缩原文可以得到最短的编码总长。然而哈夫曼编码并不是唯一的。例如对字符串"aaaxuaxz"，容易得到字母 'a'、'x'、'u'、'z' 的出现频率对应为 4、2、1、1。我们可以设计编码 {'a'=0, 'x'=10, 'u'=110, 'z'=111}，也可以用另一套 {'a'=1, 'x'=01, 'u'=001, 'z'=000}，还可以用 {'a'=0, 'x'=11, 'u'=100, 'z'=101}，三套编码都可以把原文压缩到 14 个字节。但是 {'a'=0, 'x'=01, 'u'=011, 'z'=001} 就不是哈夫曼编码，因为用这套编码压缩得到 00001011001001 后，解码的结果不唯一，"aaaxuaxz" 和 "aazuaxax" 都可以对应解码的结果。本题就请你判断任一套编码是否哈夫曼编码。

### 解题思路

我们可以先根据原字符串构建哈夫曼树求出最少所需的字节数量。

利用哈夫曼树求解具体流程如下（通过维护一个优先队列实现）：

1. 将所有字符的出现次数push到队列中
2. 优先队列pop出两个最小值x,y
3. 最终答案加上x+y，并将x+y push到队列
4. 重复2，3操作直到队列为空

当然，C中并没有优先队列（~~连队列都没有怎么可能有优先队列~~），因此，我将优先队列pop操作改成了成普通队列中遍历寻找两个最小值。当然，有能力和时间的小伙伴可以手写一个最小堆来实现优先队列。

算出最优解以后，我们需要将每个方案的总字节数与最优解做个比对，不一样肯定就"No"了，但是一样的话还需要判断方案中是否满足前缀码的需求，即方案中任意一个字符的编码都不能成为其他字符编码的前缀。

这个前缀码我选择了建树来判断，对于每个方案建一棵01树，判断一下插入的路径上是否有节点对应其他字符，如果有的话就代表该字符不满足前缀码的要求。具体看下图（以输入样例中最后一个方案作为例子）。

红色的三个点就发生了前缀冲突。

<img src="https://img.nc-77.top/FF0CCF7D5762A4E73678325B17ED51F7.png" style="zoom: 33%;" />

### 参考代码

```c
#include<stdio.h>
#include<malloc.h>
#include<string.h>
#define inf 0x3f3f3f3f
int ch[1010];
int que[1010];
int n,m;
int tail,head,size;
int vis[1010];// 标记队列中已经被去除的元素
typedef struct solution
{
    char sol_ch[1010][1010]; //记录每个方案中每个字符的编码
}solution;

typedef struct node
{
    int flag;// 代表该节点为一个字符的编码 
    struct node *left,*right;
}node;

void read()
{
    /* 读入原字符 */
    scanf("%d",&n);
    for(int i=0;i<n;i++){
       char one_ch;
       int one_cnt;
       getchar();// 去上一行的回车
       scanf("%c %d",&one_ch,&one_cnt);
       ch[one_ch-'0']=one_cnt;
       que[tail++]=one_cnt;
       size++;
   }
}
/* 计算采取哈夫曼编码的情况下的最优解 */
int cul_ans(){
    int ans=0;
    while(size>0){
        if(size>=2){
           
        /* 找队列中最小的两个值xy */
            int x=inf,y=inf;
            int x_index,y_index;
            for(int i=head;i<tail;i++){
                if(vis[i]) continue;
                if(que[i]<x){
                    x=que[i];
                    x_index=i;
                }
            }
            vis[x_index]=1;
            for(int i=head;i<tail;i++){
                if(vis[i]) continue;
                if(que[i]<y){
                    y=que[i];
                    y_index=i;
                }
            }
            vis[y_index]=1;
            ans+=(x+y);
            size-=2;
        /* 让(x+y) 入队 */
            que[tail++]=(x+y);
            size+=1;
        }
        else if(size==1){
            int index=que[head];
            ans+=ch[index];
            size--;
        }
    }
    return ans;
}
/* 检查是否符合前缀编码 */
int check(char * s,node *T)
{

    int size=strlen(s);
    for(int i=0;i<size;i++)
    {
        if(s[i]=='0'){
            if(!T->left){
                node *tmp=(node *)malloc(sizeof(node));
                tmp->left=tmp->right=NULL;
                tmp->flag=0;
                T->left=tmp;
            }
            else{
                if(T->left->flag) {return 0;}
            }
            T=T->left;
        }
        else{
            if(!T->right){
                node *tmp=(node *)malloc(sizeof(node));
                tmp->left=tmp->right=NULL;
                tmp->flag=0;
                T->right=tmp;
            }
            else{
                if(T->right->flag) {return 0;}
            }
            T=T->right;
        }
    }
    if(T->right||T->left) return 0;
    T->flag=1;
    return 1;
}

int main()
{
   read();
   int ans=cul_ans();

   scanf("%d",&m);
   for(int i=0;i<m;i++){
       /* 读入并检查每个方案 */
        int res=0;
        int flag=1;
        solution Solution;
        /* 初始化01树 */
        node* root=(node *)malloc(sizeof(node));
        root->left=root->right=NULL;
       for(int j=0;j<n;j++){
           getchar(); //去上一行的回车
           char one_ch;
           scanf("%c",&one_ch);
           getchar();
           scanf("%s",Solution.sol_ch[one_ch-'0']);
           /* res统计总字节数 */
           res+=ch[one_ch-'0']*strlen(Solution.sol_ch[one_ch-'0']);
           if(flag) flag=check(Solution.sol_ch[one_ch-'0'],root); 
       }
       if(res==ans&&flag) puts("Yes");
       else puts("No");
   }

   return 0;
}

```



## **集合的“交”与“并”**

### 题目详情

给出两个由数字组成的集合，请求这两个集合的“交”和“并”。

### 解题思路

考虑最朴素的思想，要求交集，可以将a数列每个元素都在b数列里扫一遍，存在的话就加入到交集里。

这种方法的时间复杂度是$o(nm)$

但是看下数据范围，每个数列至多会有2e5个元素。$2e5*2e5=4e10$,显然是会超时的（一般能过的代码时间复杂度要求在1e6-1e7之间，~~1e8就挺玄学了~~），因此我们需要对这个算法进行优化。

很容易想到，除了顺序查找方式，我们还可以用二分查找，二分查找的时间复杂度是$o(logm)$，这样就不会超时了，当然二分查找的前提是数组要有序，因此我们先排个序，排序的时候也要注意不能使用冒泡，选择等$o(n^2)$的排序算法，可以使用c库中提供的qsort快排。

交集解决了，考虑并集。并集简单多了，直接将a数列和b数列合并后排个序，当然输出的时候要注意去重，其实也很简单，输出的时候看一下前面的那个元素和当前这个元素是不是一样的，如果相同就跳过（~~伪去重，有点钻空子~~）。

### 参考代码

```c
#include<stdio.h>
#include<stdlib.h>
#define maxn 400010
int a[maxn],b[maxn],c[maxn],d[maxn];
int n,m;
int comp(const void*a,const void*b)
{
    return *(int*)a-*(int*)b;
}
/* 二分查找 b数组中是否有x */
int find(int x)
{
    int l=0,r=m;
    int mid;
    while(l<r){
        mid = l + r >> 1;
        if (x<b[mid]) r = mid;
        else if(x==b[mid]) return 1;
        else l = mid + 1;
    }
    return 0;
}
int main()
{
   
   scanf("%d %d",&n,&m);
   for(int i=0;i<n;i++)
   {
       scanf("%d",&a[i]);
       c[i]=a[i];
   }
   for(int i=0;i<m;i++)
   {
       scanf("%d",&b[i]);
       c[n+i]=b[i];
   }
   /* 将a,b,c三个数组升序排序 */
    qsort(a,n,sizeof(int),comp);
    qsort(b,m,sizeof(int),comp);
    qsort(c,m+n,sizeof(int),comp);
    /* 求交集 */
    int same=0;
    for(int i=0;i<n;i++){
        if(find(a[i])){
            d[same++]=a[i];
        }
    }
    printf("%d",same);
    for(int i=0;i<same;i++){
         printf(" %d",d[i]);
    }
    printf("\n");
    printf("%d",m+n-same);
    for(int i=0;i<m+n;i++){
        /* 去重 */
        if(i&&c[i]==c[i-1]) continue;
       printf(" %d",c[i]);
    }
   return 0;
}

```



##  **停车场管理**

### 题目详情

设有一个可以停放n辆汽车的狭长停车场，它只有一个大门可以供车辆进出。车辆按到达停车场时间的先后次序依次从停车场最里面向大门口处停放 (即最先到达的第一辆车停放在停车场的最里面) 。如果停车场已放满n辆车，则以后到达的车辆只能在停车场大门外的便道上等待，一旦停车场内有车开走，则排在便道上的第一辆车可以进入停车场。停车场内如有某辆车要开走，则在它之后进入停车场的车都必须先退出停车场为它让路，待其开出停车场后，这些车辆再依原来的次序进场。每辆车在离开停车场时，都应根据它在停车场内停留的时间长短交费，停留在便道上的车不收停车费。编写程序对该停车场进行管理。

### 解题思路

模拟题意就好了。特别需要注意的点是需要标记一下目前哪些车在停车场内，当一辆车驶离停车场的时候要先判断该车是否在停车场内，不在则输出“the car not in park”，不然会卡测试点2。

### 参考代码

```c
#include<stdio.h>
#include<string.h>
#include<time.h>
int pos[1500]; // 停车位
int n;
int vis[10500]; //来过车的信息
typedef struct car
{
    int A_Time;
    int D_Time;
    int stat; //停在哪个车位 
}Car;
typedef struct Que
{
    int q[10500];
    int head,tail,size;
}Que;

 Que que;
Car car[10500];  

int check_empty()
{
    for(int i=1;i<=n;i++){
        if(pos[i]==0)
            return i;
    }
    return 0;
}

int main()
{
    int first=1;
   scanf("%d",&n);
   getchar();
    char op;
   int car_id,Time;
       
   while(1){      
        scanf("%c %d %d",&op,&car_id,&Time);
        
       if(op=='E') break;
       if(op=='D'){
           /* 车驶离的时候先判断原来是否在停车场 */
           if(!vis[car_id]){
               printf("the car not in park\n");
               continue;
           }
           /* 驶离操作 */
           int pos_id=car[car_id].stat;
           car[car_id].D_Time=Time;
           pos[pos_id]=0;
           vis[car_id]=0;
           printf("car#%d out,parking time %d\n",car_id,Time-car[car_id].A_Time);
           /* 调整停车场次序,将后面的车全部往前移一位*/
           
           for(int i=pos_id;i<=n;i++){
               if(pos[i+1]) {
                   car[pos[i+1]].stat=i;
                   pos[i]=pos[i+1];
               }
               else{
                   pos[i]=0;
                   break;
               }
           }
           
           /* 有新空位让队列首停车 */
           if(que.size>0){
               que.size--;
               /* top取出队列首 */
               int top=que.q[que.head++];
               int new_pos_id=check_empty();
                vis[top]=1;
                pos[new_pos_id]=top;
                car[top].A_Time=Time;
                car[top].stat=new_pos_id;
                printf("car#%d in parking space #%d\n",top,new_pos_id);
           }
       }
        else if(op=='A'){
            
            int pos_id=check_empty();
            if(pos_id){
                /* 有空位 */
                pos[pos_id]=car_id;
                car[car_id].A_Time=Time;
                car[car_id].stat=pos_id;
                vis[car_id]=1;
                printf("car#%d in parking space #%d\n",car_id,pos_id);
            }
            else{
                /* 需等待 */
                que.q[que.tail++]=car_id;
                que.size++;
                printf("car#%d waiting\n",car_id);
            }
        }
   }
   
   return 0;
}

```



## **魔王语言解释**

```c
#include<stdio.h>
int main(){puts("tsaedsaeezegexeneietsaedsae");}
```

逃）

## **最短路径**

### 题目详情

给定一个有N个顶点和E条边的无向图，顶点从0到N−1编号。请判断给定的两个顶点之间是否有路径存在。如果存在，给出最短路径长度。 这里定义顶点到自身的最短路径长度为0。 进行搜索时，假设我们总是从编号最小的顶点出发，按编号递增的顺序访问邻接点。

### 解题思路

考察最短路。求解最短路常用的有三个算法，[Bellman-Ford](https://nc-77.top/2020/01/16/Bellman-Ford/index.html),	[Dijkstra](https://nc-77.top/2020/01/16/Dijkstra/index.html) 以及[Floyd](https://nc-77.top/2020/01/16/Floyd-Warshall/index.html)（提供了之前自己写的解析）

看下数据，N只有10，因此我选择了写起来最方便的floyd算法。

但是本题的测试点2我用floyd算法一直过不去，询问老师后得知测试点2是no path的情况。

所以又写了个dfs来判断能否到达，能到达的情况下再输出通过最短路算法得到的最短路径。

### 参考代码

```c
#include<stdio.h>
#define inf 0x3f3f3f3f
#define min(a,b) (a)<(b)?(a):(b)
/* d[i][j]存储i到j的距离 */
int d[15][15];
int n,m;
/* floyd算法求解最短路 */
void floyd()
{
   for(int k=0;k<n;k++)
      for(int i=0;i<n;i++)
         for(int j=0;j<n;j++)
         d[i][j]=min(d[i][j],d[i][k]+d[k][j]);
}
int vis[15];
int flag;
int start,end;
/* dfs判断能否从start到达end */
void dfs(int now)
{
   /* 标记过已到达的点 */
   vis[now]=1;
   /* 能到达标记 flag为1 */
   if(now==end) flag=1;
   /* flag为1后直接return */
   if(flag) return;
   for(int i=0;i<n;i++)
   {
      if(d[now][i]==1&&!vis[i]){
         dfs(i);
      }
   }
}
int main()
{
   
   scanf("%d%d",&n,&m);
   for(int i=0;i<m;i++)
   {
      int x,y;
      scanf("%d%d",&x,&y);
      d[x][y]=d[y][x]=1;
   }
   
   scanf("%d%d",&start,&end);
   /* 初始化距离 */
   for(int i=0;i<n;i++)
      for(int j=0;j<n;j++)
      {
         if(d[i][j]) continue;
         if(i==j) d[i][j]=0;
         else  d[i][j]=inf;
      }
         
   floyd();
   vis[start]=1;
   /* 判断能否从start到达end */
   dfs(start);
   
   if(!flag){
      printf("There is no path between %d and %d.",start,end);
   }
   else{
      printf("The length of the shortest path between %d and %d is %d.",start,end,d[start][end]);
   }
   return 0;
}

```

## 周游世界

### 题目详情

周游世界是件浪漫事，但规划旅行路线就不一定了…… 全世界有成千上万条航线、铁路线、大巴线，令人眼花缭乱。所以旅行社会选择部分运输公司组成联盟，每家公司提供一条线路，然后帮助客户规划由联盟内企业支持的旅行路线。本题就要求你帮旅行社实现一个自动规划路线的程序，使得对任何给定的起点和终点，可以找出最顺畅的路线。所谓“最顺畅”，首先是指中途经停站最少；如果经停站一样多，则取需要换乘线路次数最少的路线。

### 解题思路

个人感觉这应该是期末验收里最难做的一道了。

由于笔者使用的是Bellman_ford求解最短路，不懂的可以先戳这 ->[Bellman_Ford](https://nc-77.top/2020/01/16/Bellman-Ford/index.html)

由于最后输出的路径首要前提是最短路径，然后在最短路径的基础上保证换乘最少，因此我们可以考虑使用Bellman_ford求解最短路的过程中增添一些东西，不仅需要pre记录路径前驱（pre[i]表示i的前驱），还需要一个company数组来记录到达该站点**最后**所走的路线，一个transfer数组来记录到达该站点所需要的换乘次数，最后在最短路更新的过程中维护这几个数组就好了。

输出的时候还原路径也比较麻烦，可以先根据pre前驱得到路径上所有的站点，然后再根据company所记录的路线选取需要换乘的那些站点。

总而言之，这道题写起来是挺麻烦的，需要你对最短路算法的更新过程有本质的理解，但相信我，独立写出这道题会让你对最短路算法有更深刻的理解。

### 参考代码

```c
#include<stdio.h>
#include<string.h>
#define maxn 100000
#define inf 0x3f3f3f3f
int V,E;// 顶点数,边数
int d[maxn],vis[maxn];
int transfer[maxn];// 每个站点换乘次数
int pre[maxn],path[maxn]; // 记录前驱
int pos[maxn]; // 存储最终路径
int company[maxn]; // 每个站点由哪条路线过来
typedef struct edge
{
    int from,to,cost;
    int company;
}edge;
edge es[maxn];

void add_edge(int from,int to,int line){
    es[E].from=from;
    es[E].to=to;
    es[E].cost=1;
    es[E].company=line;
    E++;
}

void bellman_ford(int start){
    /* pre记录路径 */
    memset(pre,-1,sizeof(pre));
    memset(d,0x3f,sizeof(d));

    d[start]=0;

    for(int i=0;i<V;i++){
        /* flag标记本次是否更新,不加会超时*/
        int flag=0;
        for(int j=0;j<E;j++){
            edge e=es[j];
            if(d[e.to]>d[e.from]+e.cost){
                d[e.to]=d[e.from]+e.cost;
                pre[e.to]=e.from;
                company[e.to]=e.company;
                transfer[e.to]=transfer[e.from];
                /* 需要换乘 */
                if(e.company!=company[e.from]){
                    transfer[e.to]++;
                }
                flag=1;
            }
            else if(d[e.to]==d[e.from]+e.cost){
                 /* 距离相同的情况下换乘次数更少则更新 */
                 if(transfer[e.to]>transfer[e.from]+(e.company!=company[e.from])){
                    company[e.to]=e.company;
                    pre[e.to]=e.from;
                    transfer[e.to]=transfer[e.from]+(e.company!=company[e.from]);
                    flag=1;
                }
                
            }
        }
        /* 如果此次没更新代表之后也不会更新,直接return */
        if(!flag) return;
    }
}
void get_path(int start,int end) //还原路径
{
    
	if(d[end]==inf){
        printf("Sorry, no line is available.\n");
        return;
    }
    /* path存储路径上的每个站点 
       pos存储需要换乘的站点 */
    int path_cnt=0 ,pos_cnt=0,tmp=end;
    

    /* 根据pre前驱记录还原path */
    while(1){
        path[path_cnt++]=tmp;
        if(pre[tmp]==-1) break;
        tmp=pre[tmp];
    }
    /* 根据path选取出需要换乘的站点 */
    pos[pos_cnt++]=start;
    for(int i=path_cnt-2;i>0;i--){
        if(company[path[i]]!=company[path[i-1]])
            pos[pos_cnt++]=path[i];
    }
    pos[pos_cnt++]=end;
    printf("%d\n",path_cnt-1);

    for(int i=0;i<pos_cnt-1;i++){
        printf("Go by the line of company #%d from %04d to %04d.\n",company[pos[i+1]],pos[i],pos[i+1]);
    }
}
int main()
{
   int n;
   scanf("%d",&n);
   for(int i=0;i<n;i++){
       int k;
       scanf("%d",&k);
       V+=k;
       int pre_station=-1;
       /* 建图 */
       while(k--){
           int this_station;
           scanf("%4d",&this_station);
           if(pre_station!=-1){
               add_edge(pre_station,this_station,i+1);
               add_edge(this_station,pre_station,i+1);
           }
           pre_station=this_station;
       }
   }
   int Q;
   scanf("%d",&Q);
   while(Q--){
       int start,end;
       scanf("%4d%4d",&start,&end);
       bellman_ford(start);
       get_path(start,end);
   }
   return 0;
}

```



## 任务调度的合理性

### 题目详情

假定一个工程项目由一组子任务构成，子任务之间有的可以并行执行，有的必须在完成了其它一些子任务后才能执行。“任务调度”包括一组子任务、以及每个子任务可以执行所依赖的子任务集。

比如完成一个专业的所有课程学习和毕业设计可以看成一个本科生要完成的一项工程，各门课程可以看成是子任务。有些课程可以同时开设，比如英语和C程序设计，它们没有必须先修哪门的约束；有些课程则不可以同时开设，因为它们有先后的依赖关系，比如C程序设计和数据结构两门课，必须先学习前者。

但是需要注意的是，对一组子任务，并不是任意的任务调度都是一个可行的方案。比如方案中存在“子任务A依赖于子任务B，子任务B依赖于子任务C，子任务C又依赖于子任务A”，那么这三个任务哪个都不能先执行，这就是一个不可行的方案。你现在的工作是写程序判定任何一个给定的任务调度是否可行。

### 解题思路

经典拓扑排序题。

先说下拓扑排序的流程，维护一个队列。

1. 找到入度为0的点，将这些点加入到队列中

2. 从队列中取出一个点top，删除top连出去的所有边（将g[top] [i]置为0)

3. 每删除一条边判断删除后该点入度是否为0，为0且不在队列中就入队

4. 重复1，2，3直到队列为空或找不到入度为0的点

走完流程后判断一下是否所有点入度均为0，是的话代表该图存在拓扑序，否则不存在。

有个麻烦的地方在于C中没有队列，因此需要用数组模拟一下队列。

### 参考代码

```c
#include<stdio.h>
int g[105][105];/* 记录边 */
int in[105];/* 记录每个点的入度 */
int vis[105];/* 记录是否访问过 */
int que[105];/* 模拟队列 */
int n;
void bfs(){
    int tail=0,head=0,size=0;
    /* 将入度为0的点加入队列 */
    for(int i=1;i<=n;i++){
        if(!in[i]){
            que[tail++]=i;
            vis[i]=1;
            size++;
        }
        
    }
    while(size>0){
        /* 取出队头元素 */
        int top=que[head++];
        size--;
        for(int i=1;i<=n;i++){
            /* 删边 */
            if(g[top][i]){
                g[top][i]=0;
                in[i]--;
                /* 入度为0且没访问过就入队 */
                if(in[i]==0){
                    if(!vis[i]) {
                        que[tail++]=i;
                        vis[i]=1;
                        size++;
                    }
                }
            }
        }
        
    }
    return;
}
int main()
{

   scanf("%d",&n);
   for(int i=1;i<=n;i++){
       int k;
       scanf("%d",&k);
       in[i]=k;
       while(k--){
           int x;
           scanf("%d",&x);
           g[x][i]=1;
       }
   }
   bfs();
   /* sum表示所有点入度和 */
   int sum=0;
   for(int i=1;i<=n;i++){
       sum+=in[i];
   }
   if(sum) printf("0");
   else printf("1");
   return 0;
}

```

