---
title: K-Means
date: 2020-09-22 13:58:52
tags: [python,无监督,聚类]
categories: [机器学习]
---

## 问题引入

在2000年和2004年的美国总统大选中，候选人的得票数比较接近或者说非常接近。任一候选 人得到的普选票数的最大百分比为50.7%，而最小百分比为47.9%。如果1%的选民将手中的选票 投向另外的候选人，那么选举结果就会截然不同。实际上，如果妥善加以引导与吸引，少部分选 民就会转换立场。尽管这类选举者占的比例较低，但当候选人的选票接近时，这些人的立场无疑 会对选举结果产生非常大的影响。如何找出这类选民，以及如何在有限的预算下采取措施来吸 引他们？答案就是聚类。

显然，在无监督学习中，由于训练样本的标记是未知的，但是我们可以通过训练样本的内在性质以及特征规律，将样本分成若干类来为数据分析提供基础，这种分类方法就称作聚类。

<img src="https://img.nc-77.top/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202020-09-22%20141414.png" style="zoom: 67%;" />

如图，我们的目标就是将图中若干个二维数据点进行分类。下面介绍一种应用广泛的原型聚类算法：K—Means。

注：本篇文章所有代码及数据集已上传至[github](https://github.com/nc-77/Machine_Learning)

## K—Means

K—Means的工作流程如下。

1. 以上图为例，我们可以将所有样本大致分为k个簇。因此，我们在原来的样本域中随机选取k个点作为初始簇心。

   ```python
   #数据预处理(准备工作):
   #loadDataSet: 读取样本集,转换成array
   #calDist: 求两个向量的欧氏距离
   #randCent: 随机初始化簇心
   import pandas as pd
   import numpy as np
   from math import *
   import random 
   import matplotlib.pyplot as plt
   plt.style.use('ggplot')
   def loadDataSet(filename):
       df=pd.read_csv(filename,sep='\t',header=None)
       return df.to_numpy()
   
   #cnt 存储每次训练结果png的编号
   def plotDataSet(dataMat,cnt,centroids=None,clusterAssess=None):
       colors=['green','red','black','brown']
       fig,ax=plt.subplots()
       if(centroids is None):
           ax.scatter(dataMat[:,0],dataMat[:,1])
       else:
           colorsList=[colors[int(x)] for x in clusterAssess[:,0]]
           ax.scatter(dataMat[:,0],dataMat[:,1],color=colorsList)
           ax.scatter(centroids[:,0],centroids[:,1],color='blue',marker='x')
       ax.set_xlabel('X1')
       ax.set_ylabel('X2')
       plt.savefig(f"kmeans{cnt}")
       
       
   def calDist(vecA,vecB):
       return sqrt(sum(np.power(vecA-vecB,2)))
   
   
   def randCent(dataMat,k):
       m,n=dataMat.shape
       centroids=np.zeros((k,n))
       for i in range(n):
           #print(min(dataMat[:,i]),max(dataMat[:,i]))
           dataMin=min(dataMat[:,i])
           dataMax=max(dataMat[:,i])
           # 为每个簇心随机生成[dataMin,dataMax]的坐标
           for j in range(k):
               centroids[j,i]=random.uniform(dataMin,dataMax)
       return centroids
   ```

   

2. 我们为所有样本找到距离最近的簇心，并将该样本分配到这个簇中。（X表示簇心，相同颜色的点表示分到同一个簇中）

   <img src="https://img.nc-77.top/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202020-09-22%20143500.png" style="zoom:67%;" />

3. 取每个簇中所有样本特征的平均值，重新确定簇心的位置。

4. 回到2，迭代更新

5. 当所有样本的颜色不变时，退出更新，训练结束

   ```python
   from operator import itemgetter
   def KMeans(dataMat,k,calDis=calDist,creatCent=randCent):
       m,n=dataMat.shape
       # clusterAssess 一列记录簇索引，一列记录误差（距离簇心距离）
       clusterAssess=np.zeros((m,2))
       centroids=creatCent(dataMat,k)
       flag=True
       cnt=1
       while flag:
           flag=False
           # 循环遍历每个数据点
           for i in range(m):
               bestDis=inf
               bestIndex=-1
               # 循环遍历每个簇心，寻找距离最短的
               for j in range(k):
                   dis=calDis(dataMat[i],centroids[j])
                   if(dis<bestDis):
                       bestDis=dis
                       bestIndex=j
               if(bestIndex!=clusterAssess[i][0]):flag=True
               clusterAssess[i]=[bestIndex,bestDis**2]
           # 重新计算簇心坐标
           for i in range(k):
               filterDataMat=dataMat[clusterAssess[:,0]==i]
               centroids[i]=np.mean(filterDataMat,axis=0)
           plotDataSet(dataMat,cnt,centroids,clusterAssess)
           cnt+=1
       return centroids,clusterAssess
   ```

   下面是main测试函数（会生成每次训练结果的图片保存至代码目录下，testSet在[github](https://github.com/nc-77/Machine_Learning)上）

   ```python
   dataMat=loadDataSet("testSet.txt")
   #calDist(dataMat[0],dataMat[1])
   centroids=randCent(dataMat,4)
   #print(centroids)
   plotDataSet(dataMat,0)
   centroids,clusterAssess=KMeans(dataMat,4)
   ```

   最终训练结果如下图：

<img src="https://img.nc-77.top/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202020-09-22%20144024.png" style="zoom:67%;" />

## 性能度量

聚类性能度量大致有两种：

1. 外部指标——将聚类结果与某个参考模型进行比较

2. 内部指标——直接考察聚类结果而不适用任何参考模型

我们在这选用的是SSE（误差平方和）指标来度量

$J=\sum_{i=1}^{m}dis(x_i,c_{cluster[i]})(cluster[i]表示x_i所属的簇)$

其中的dis可以采取欧式距离，曼哈顿距离或者加权距离，此处选取的为欧式距离的平方（为了更加重视远离中心的点）

```python
def costJ(clusterAssess):
    return np.sum(clusterAssess[:,1])
```



## 细节探讨

介绍完k-means的基本工作流程，下面还有几个具体算法实现的细节处理部分。

### 选取原始簇心位置

我们来看一个原始簇心随机的不那么好的例子。

训练前：

<img src="https://img.nc-77.top/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202020-09-22%20152050.png" style="zoom:67%;" />

训练后：

<img src="https://img.nc-77.top/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202020-09-22%20152113.png" style="zoom:67%;" />

可以看到，当原始随机的簇心落在图上的情况时，训练后的结果并不是我们预期的那样，且训练前后的结果相差不大。这是由于在一开始收敛时就陷入了局部最优解，从而无法收敛到全局最优解。

因此，为解决这一问题，我们可以将随机初始化多执行几次来避免一次随机恰好陷入局部最优解的尴尬或者采取一种名为二分k-means的改进算法（通过开始将所有样本点归于一个簇类来避免随机，之后每次选取划分后SSE减小最大的一个簇进行k=2的k-mean分割直到满足所需的簇类为止）。



### 确定k的大小

先让我们看一个例子来明白确定k（聚类数量）的大小意义。

你是一个衣服店的老板，你需要根据手头上已有的顾客身高体重信息来确定T-shirt的尺码定制。

<img src="https://img.nc-77.top/2020-09-22%2015-34-05-1.png" style="zoom: 50%;" />

很显然，我们需要确定一个合适的聚类数量来满足大多数顾客的需求以及避免过多的尺寸供过于求产生浪费。

那么，我们可以通过训练不同的k后，比较SSE指标的大小，利用“肘部法则”来确定合适的聚类数量。

![](https://img.nc-77.top/2020-09-22%2016-03-05-1.png)

如左图，显然k=3是图中的一个拐点，（形如人的肘部，因此称为肘部算法）k从2到3带来的性能提升是很大的,但是从3到4就没有这么明显了。（毕竟分的类越多，SEE肯定越小）。因此，为了减少成本，我们就可以选取3类尺码作为我们的T-shirt尺寸。

当然，并不是所有的情况下我们都能得到左图，也有部分情况下得到的是右图（此时需要换个方法来求解）。

## 总结

k-means作为聚类入门的基本算法，其原理简单，容易实现。但同时也有着k值很难确定，聚类效果依赖于聚类中心的初始化，容易陷入局部最优等缺点。

同时除了k-means算法，聚类还有基于层次聚类的AGNES算法等，故这篇文章仅仅作为聚类入门来使用。

## Refences

1. 哈林顿李锐. 机器学习实战 : Machine learning in action[M]. 人民邮电出版社, 2013.
2. 周志华. 机器学习 : = Machine learning[M]. 清华大学出版社, 2016.
3. 吴恩达  机器学习入门视频 

