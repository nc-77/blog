---
title: logistic回归
date: 2020-08-30 16:02:39
tags: [python,logistic,分类]
categories: [机器学习]
---

## 问题引入

![](https://img.nc-77.top/20200830162219.png)

如上图，我们将每个点看成一个样本，$x_1,x_2$为样本的两个特征。可以看到所有点都可划分为class0或class1。（本文只讨论二分类问题）

当我们要对一个新的样本进行分类，可以采用回归预测的方法。也就是拟合一个假设函数$h_\theta(x)$,该函数以样本的所有特征为参数，返回该样本的类别。但是，对于一个分类的问题，常规的回归预测并不适用，因为我们经过假设函数$h_\theta(x)$预测往往返回的是一个连续值，但对于样本而言，其标签非0即1，是一个离散值。所以，就有了一种用于分类的回归算法logistic。

在继续阅读前，希望您已经有了线性回归和梯度下降法的基础。

本篇文章相关代码以及数据集已上传至[github](https://github.com/nc-77/Machine_Learning/tree/master/logistic%E5%9B%9E%E5%BD%92)

## 假设函数

我们一般定义$h_\theta(x)$=$\theta_0+\theta_1\times x_1+\theta_2\times x_2+...+\theta_n\times x_n$ (别问为什么这么定义，问就是奥卡姆剃刀)

我们的最终目的，就是去拟合参数$\theta=[\theta_0,\theta_1...\theta_n]^T$来提升分类器的性能。

对于一个分类问题而言，我们需要设定阈值来比较样本与阈值的大小进行分类。但是对于这个朴素的$h_\theta(x)$,返回值本身没有意义且阈值较难设定。所以从某种角度上来说，我们希望$h_\theta(x)$的返回值有意义，比如表示该样本属于class0的概率，那就要求$h_\theta$的值域为$[0,1]$，这样当$h_\theta(x)$>0.5时，我们将其分类为class0，当

$h_\theta(x)$<0.5时，我们将其分类为class1.于是，大佬们就找到了一种函数sigmoid。

### sigmoid

sigmoid函数原型为$g(z)=\frac{1}{1+e^{-z}}$

![](https://img.nc-77.top/20200830223544.png)

该函数有良好的特性适应我们的需求。

1. 其定义域为$(-\infty,+\infty)$,相应的其值域为$(0,1)$，正好满足概率分布的要求。

2. 它是单调上升的函数，具有良好的连续性，不存在不连续点。而且由于$g(0)=0.5$因此可以选取0作为分界点。

因此将原来的$h_\theta(x)=\theta^Tx$作为sigmoid函数的输入，得到

$h_\theta(x)=g(\theta^Tx)=\frac{1}{1+e^{-\theta^Tx}}$

好了，确定了假设函数的形式，接下来的任务就是如何通过样本数据来拟合参数$\theta$.

## 代价函数

在来拟合参数前，我们先来讲讲如何我们对函数拟合好坏的评价标准。我们的方法是设计一个代价函数。

$J(\theta)=\frac{1}{m}\sum_{i=1}^{m}\frac{1}{2}(h_\theta(x^i)-y^i))^2$

这是普通线性回归中会用到的代价函数$Cost(h_\theta(x),y)=\frac{1}{2}(h_\theta(x)-y))^2$

但是由于logistic中的$h_\theta(x)$的是一个很复杂的函数，且非凸，

会有很多局部最小值，我们如果还是采取这个函数为代价函数，在使用梯度下降法时就无法保证收敛至全局最小值。所以，针对logistic回归，我们需要设计一种新的代价函数。

$Cost(h_\theta(x),y)=\begin{cases}-log(h_\theta(x)),  & \text{if $y$ = 1} \\ -log(1-h_\theta(x)), & \text{if $y$ = 0}\end{cases}$

<img src="https://img.nc-77.top/20200830235546.png" style="zoom:67%;" />

如图这两个函数当输入在趋于0或1时都表现的很好，例如当实际$y=1$时，如果$h_\theta(x)=1$，完全预测正确，那么代价值就为0，反之若$h_\theta(x)=0$，完全预测错误，那么代价就会趋向于无穷。

同时代价函数可以简化为$Cost(h_\theta(x),y)=-y\times log(h_\theta(x))-(1-y)\times log(1-h_\theta(x))$

所以$J(\theta)=\frac{1}{m}\sum_{i=1}^{m}Cost(h_\theta(x^i),y^i)$

​		$=-\frac{1}{m}(y^i\times log(h_\theta(x^i))+(1-y^i)\times log(1-h_\theta(x^i)))$

这就是logistic的代价函数了，由于该函数为凸函数，我们就可以使用梯度下降法来求取最优解了。

## 梯度下降法

众所周知，梯度下降法的更新公式为$\theta=\theta- \alpha\times\frac{\partial J(\theta)}{\partial\theta}$

其中，$\alpha$为我们设置的学习率。

那么，代入logistic的$J(\theta)$,得到

$\theta_j=\theta_j-\alpha\sum_{i=1}^m(h_\theta(x^i)-y^i)x_j^i$

$=\theta_j+\alpha\sum_{i=1}^m(y^i-h_\theta(x^i))x_j^i$

令m表示样本数,n表示特征数

向量化，得到$\theta_{n\times1}=\theta_{n\times1}-\alpha x^T_{n*m}\times(h(x)_{m*1}-y_{m*1})$

有了公式，我们就能写出代码了。

```python
import numpy as np
def sigmoid(x):
    return 1.0/(1+np.exp(-x))
def gradAscent(dataMatIn,labelMatIn):
    # dataMat为m*n的样本矩阵
    # labelMat为m*1的标签矩阵
    dataMat=np.mat(dataMatIn) 
    labelMat=np.mat(labelMatIn).transpose()  
    alpha=0.001
    maxCycles=500
    # weights(代求参数)初始化为n*1的矩阵
    weights=np.ones((dataMat.shape[1],1))
    # 进行maxCycles次迭代
    for i in range(maxCycles):
        # h为m*1矩阵,每行表示h(x)
        h=sigmoid(dataMat*weights)
        error=labelMat-h
        weights=weights+alpha*dataMat.transpose()*error
    # 将weights转换为array返回
    return weights.getA()
```

接下来让我们画出分界曲线来直观的感受一下拟合的好坏吧。

由sigmoid函数可知分界点$h_\theta(x)=0.5$

$\theta^Tx=0$

在这个测试集上，得到 $\theta_0+\theta_1x_1+\theta_2x_2=0$

```python
# plotDataSet 画出原始的散点图
def plotDataSet():
    dataMat,labelMat=loadDataSet()
    x0_set=[];y0_set=[];x1_set=[];y1_set=[]
    for i in range(len(dataMat)):
        if(labelMat[i]):
            x1_set.append(dataMat[i][1]);y1_set.append(dataMat[i][2])
        else:
            x0_set.append(dataMat[i][1]);y0_set.append(dataMat[i][2])
    fig,ax=plt.subplots()
    ax.scatter(x0_set,y0_set,color='green',label='class0')
    ax.scatter(x1_set,y1_set,color='red',label='class1')
    ax.set_xlabel('X1')
    ax.set_ylabel('X2')
    ax.legend()
    return fig,ax
# plotfit 在原始散点图上添加拟合直线
def plotfit(weights):
    fig,ax=plotDataSet()
    x=np.arange(-3,3,0.1)
    y=(-weights[0]-weights[1]*x)/weights[2]
    ax.plot(x,y)
    plt.show()
weights=gradAscent(dataMat,labelMat)
plotfit(weights)
```

得到下图

<img src="https://img.nc-77.top/20200831131131.png" style="zoom:67%;" />

可以看到，拟合的还算不错。

但是这样的梯度下降法存在一个问题:效率过低。因为每进行一次迭代，其都需要对整个样本数据集进行矩阵操作，像我们使用的数据集中只有几百个样本和较少的特征。但是当我们碰到成千上万个样本以及较多的特征时，梯度下降法的效率就不足以支撑我们处理这些数据了。

因此为解决效率问题，下面介绍一种随机梯度下降法。

## 随机梯度下降法（SGD）

随机梯度下降法本质上和批量梯度下降一样，唯一的区别在于每次更新时随机选取数据集中的一个样本进行更新，而不是遍历整个数据集再进行更新，这样就大大降低了算法的时间复杂度。但是，带来的代价就是准度会降低，由于是随机选取一个进行更新，没有考虑全局，就像在黑暗中盲目摸索，无法保证每次更新都是朝着正确的方向，所以相比批量梯度下降的准确率肯定会下降。

```python
import random
def stocGradAscent(dataMat,labelMat,maxCycles=10):
    m,n=np.shape(dataMat)
    weights=np.ones(n)
    for i in range(maxCycles):
        # 创建一个用于随机选取的列表
        dataIndex=list(range(m))
        for j in range(m):
            alpha=4/(1.0+j+i)+0.01
            # 随机选取一个样本
            randindex=int(random.uniform(0,len(dataIndex)))
            h=sigmoid(sum(dataMat[randindex]*weights))
            error=labelMat[randindex]-h
            weights=weights+alpha*error*dataMat[randindex]
            # 删除选取的样本序号
            del(dataIndex[randindex])
    return weights
weights=stocGradAscent(np.array(dataMat),labelMat,10)
plotfit(weights)
```

效果如下：

<img src="https://img.nc-77.top/20200831153626.png" style="zoom:67%;" />

可以看到，SGD算法和BGD算法内循环的时间复杂度是一样的，但是SGD只需要大迭代10次就能达到BGD迭代500次的效果，原因就在于SGD每次随机选取一个样本计算代价后就对参数进行了更新。

## 使用logistic回归预测病马的死亡率

我们先来看一下数据集：

该数据集中有很多缺失值，一般对于数据集有缺失的值，有几种做法：

- 使用可用特征的均值来填补缺失值

- 使用特殊值来填补确实值 如-1

- 忽略有缺失值的样本

- 使用相似样本的均值添补缺失值

- 使用另外的机器学习算法预测缺失值

所以在预处理阶段我们可以使用0来代替所有缺失值，因为在logistic回归中，如果某个特征值为0，应用该更新公式:$weights=weights+\alpha\times error \times dataMat[i]$,$weights$将不会得到更新。

然后我们在看一下各个特征属性，显然第3行的医院编号对病马的死亡率是没有影响的，所以我们将这行去掉。同时我们将所有特征属性归一化。

```python
import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler
def loadData(filename):
    df=pd.read_csv(filename,header=None,delimiter="\s+")
    df=df.replace('?',0.0)
    x=df.iloc[:,np.r_[0:2,3:26]]
    y=df.iloc[:,-1]
    # 进行特征值归一化
    ss=StandardScaler()
    x=ss.fit_transform(x)
    return x,y
x_train,y_train=loadData('horse-colic.data')
x_test,y_test=loadData('horse-colic.test')
```

预处理完成，我们使用logistic回归进行一下预测，这里我直接使用sklearn的logistic回归算法（偷个懒）。

```python
from sklearn.linear_model import LogisticRegression
classifier=LogisticRegression(solver='liblinear',max_iter=500).fit(x_train,y_train)
test_accury=classifier.score(x_test,y_test)
print(f'准确率为{test_accury}')
```

结果如下：

```
准确率为0.8382352941176471
```

