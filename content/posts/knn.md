---
title: knn
date: 2020-08-10 22:19:14
tags: [python,kNN]
categories: [机器学习]
---

# K-近邻算法

## 原理

入坑学的第一个算法，理论看不懂的我直接来实战了（枯）

其实k-近邻算法（以下简称knn）的原理蛮简单的。

比如说现在有一个训练样本集，并且都有标签。当我们遇到一个未标签过的新数据时，将新数据与每个已有的样本进行特征比较，然后选择其中特征差异最小的前k个，最后选择这k个相似数据中出现最多的标签作为新数据的标签。（通常$k<20$）

一句话概括其实就是谁近选谁。

## k-近邻算法的一般流程

1. 收集数据：可以使用任何方法。

2. 准备数据：距离计算所需要的数值，最好是结构化的数据格式。

3. 分析数据：可以使用任何方法。

4. 训练算法：此步骤不适用于k-近邻算法。

5. 测试算法：计算错误率。

6. 使用算法：首先需要输入样本数据和结构化的输出结果，然后运行k-近邻算法判定输
   入数据分别属于哪个分类，最后应用对计算出的分类执行后续的处理。

   ​																											——转自《机器学习实战》

## 算法实现

 对未知类别属性的数据集中的每个点依次执行以下操作： 

1.  计算已知类别数据集中的点与当前点之间的距离； 
2.   按照距离递增次序排序； 
3.  选取与当前点距离最小的k个点；  
4.   确定前k个点所在类别的出现频率； 
5.  返回前k个点出现频率最高的类别作为当前点的预测分类。  

```python
from numpy import *
import operator
def createDataSet():
    group=array([[1.0,1.1],[1.0,1.0],[0,0],[0,0.1]])
    labels=['A','A','B','B']
    return group,labels
def classify0(inX,dataSet,labels,k):
    #计算新数据与每个点的距离
    dataSetSize=dataSet.shape[0]
    diffMat=tile(inX,(dataSetSize,1))-dataSet
    sqDiffMat=diffMat**2
    sqDistances=sqDiffMat.sum(axis=1)
    distances=sqDistances**0.5
    #argsort返回下标
    sortedDistances=distances.argsort() 
    classcount={}
    #统计前k个group的label出现次数
    for i in range(k):
        voteIlabel=labels[sortedDistances[i]]
        classcount[voteIlabel]=classcount.get(voteIlabel,0)+1
   sortedclasscount=sorted(classcount.items(),key=operator.itemgetter(1),reverse=True)
    return sortedclasscount[0][0]
```

### 测试方法：

进入到包含这个KNN.py的文件夹的python工作模式下，运行

```python
import KNN
group,label=KNN.createDataSet()
KNN.classify0([0,0],group,label,3)
```

得到的结果为“B”标签

## 海伦约会：

训练集：海伦约会过的人属性特性以及海伦对他们的喜欢程度

训练方法：knn

归一特征值处理：由于属性之间无法横向比较，将三个属性的特征值差都归一为[0,1]之间

训练误差（错误率）：5%

```python
from numpy import *  # 导入科学计算包
from matplotlib.font_manager import FontProperties
import matplotlib.lines as mlines
import matplotlib.pyplot as plt  # 导入绘图工具包
import operator  # 导入运算符模块
 
 
def file2matrix(filename): #读取文本数据
    fr = open(filename)  # 打开文件
    arrarOLines = fr.readlines()  # 读取内容
    numberOfLines = len(arrarOLines)  # 解析有多少行
    returnMat = zeros((numberOfLines, 3))  # 创建行数*3的矩阵，以0填充
    classLabelVector = []
    index = 0
    for line in arrarOLines:
        line = line.strip()  # 删除空白字符
        listFromLine = line.split('\t')  # 以空格来分割
        returnMat[index, :] = listFromLine[0:3]  # 前三位放入矩阵
        classLabelVector.append(listFromLine[-1])  # 最后一位存入标签
        index += 1
    return returnMat, classLabelVector
 
 
def showData(datingDataMat, datingLabels): #绘制散点图
    font = FontProperties('SimHei')   #设置字体为黑体
    fig, ax = plt.subplots(nrows=2, ncols=2, sharex=False, sharey=False, figsize=(15, 10))   #将fig画布分为2*2四个区域，x轴和y轴不共享，每个画布13*8大小
    LabelsColors = []
    for i in datingLabels:
        if i == 'didntLike':          #区分三种标签的颜色
            LabelsColors.append('black')
        if i == 'smallDoses':
            LabelsColors.append('orange')
        if i == 'largeDoses':
            LabelsColors.append('red')
    """左上角[0][0]"""
    ax[0][0].scatter(x=datingDataMat[:, 0], y=datingDataMat[:, 1], color=LabelsColors, s=15, alpha=.5)      #设置x轴数据和y轴数据，颜色，圆点大小，透明度
    ax00_title = ax[0][0].set_title(u'每年获得的飞行常客里程数与玩视频游戏所消耗时间占比', FontProperties=font)    #设置标题和字体
    ax00_xlabel = ax[0][0].set_xlabel(u'每年获得的飞行常客里程数', FontProperties=font)
    ax00_ylabel = ax[0][0].set_ylabel(u'玩视频游戏所消耗时间占比', FontProperties=font)
    plt.setp(ax00_title, size=10, weight='bold', color='red')         #设置图例
    plt.setp(ax00_xlabel, size=8, weight='bold', color='black')
    plt.setp(ax00_ylabel, size=8, weight='bold', color='black')
    """右上角[0][1]"""

    ax[0][1].scatter(x=datingDataMat[:, 0], y=datingDataMat[:, 2], color=LabelsColors, s=15, alpha=.5)
    ax01_title = ax[0][1].set_title(u'每年获得的飞行常客里程数与每周消费的冰激淋公升数', FontProperties=font)
    ax01_xlabel = ax[0][1].set_xlabel(u'每年获得的飞行常客里程数', FontProperties=font)
    ax01_ylabel = ax[0][1].set_ylabel(u'每周消费的冰激淋公升数', FontProperties=font)
    plt.setp(ax01_title, size=10, weight='bold', color='red')
    plt.setp(ax01_xlabel, size=8, weight='bold', color='black')
    plt.setp(ax01_ylabel, size=8, weight='bold', color='black')
    """左下角[1][0]"""
    ax[1][0].scatter(x=datingDataMat[:, 1], y=datingDataMat[:, 2], color=LabelsColors, s=15, alpha=.5)
    ax10_title = ax[1][0].set_title(u'玩视频游戏所消耗时间占比与每周消费的冰激淋公升数', FontProperties=font)
    ax10_xlabel = ax[1][0].set_xlabel(u'玩视频游戏所消耗时间占比', FontProperties=font)
    ax10_ylabel = ax[1][0].set_ylabel(u'每周消费的冰激淋公升数', FontProperties=font)
    plt.setp(ax10_title, size=10, weight='bold', color='red')
    plt.setp(ax10_xlabel, size=8, weight='bold', color='black')
    plt.setp(ax10_ylabel, size=8, weight='bold', color='black')
 
    didntLike = mlines.Line2D([], [], color='black', marker='.', markersize=6, label='didntLike')
    smallDoses = mlines.Line2D([], [], color='orange', marker='.', markersize=6, label='smallDoses')
    largeDoses = mlines.Line2D([], [], color='red', marker='.', markersize=6, label='largeDoses')
    ax[0][0].legend(handles=[didntLike, smallDoses, largeDoses])   #添加图例
    ax[0][1].legend(handles=[didntLike, smallDoses, largeDoses])
    ax[1][0].legend(handles=[didntLike, smallDoses, largeDoses])
    plt.show()     #显示图表
def autonorm(dataSet): #归一化特征值  newvalue=(oldvue-minvals)/(maxvals-minvals) [0,1]
    minvals=dataSet.min(0) #从列中选取最小值
    maxvals=dataSet.max(0)
    ranges=maxvals-minvals
    normdata=dataSet-tile(minvals,(dataSet.shape[0],1))
    normdata=normdata/tile(ranges,(dataSet.shape[0],1))
    return normdata
def classify0(inx,dataSet,labels,k):  #原始分类器
    diffmat=tile(inx,(dataSet.shape[0],1))-dataSet
    diffmat=diffmat**2
    dismat=diffmat.sum(axis=1)   #axis=0(默认)矩阵按列相加 axis=1 矩阵按行相加
    dismat=dismat**0.5           #计算距离
    sorteddismat=dismat.argsort()  #argsort()默认从小到大排序，返回下标
    count_class={}
    for i in range(k):
        label=labels[sorteddismat[i]]
        count_class[label]=count_class.get(label,0)+1
    sortedclasscount=sorted(count_class.items(),key=operator.itemgetter(1),reverse=True) #按label数量从小到大排序
    return sortedclasscount[0][0]
def datatest(): #测试错误率
    datamat,labels=file2matrix("datingTestSet.txt")
    normdata=autonorm(datamat) #归一特征值
    m=normdata.shape[0]
    num_test=int(m*0.1)  #测试数据占10%
    error_count=0 
    for i in range(num_test): #计算错误率
        res_classify=classify0(normdata[i,:],normdata[num_test:m,:],labels[num_test:m],3)
        print("the classifier came back with %s,the real answer is %s"%(res_classify,labels[i]))
        if(res_classify!=labels[i]):
            error_count=error_count+1
    error_rate=error_count/num_test
    print("error_count is ",error_count)
    print("the total error rate is ",error_rate)
    return error_rate
def classifperson(): # 最终预测器
    games=float(input("please input the time of playing video games weekly:"))
    fly=float(input("please input the flier miles yearly:"))
    icecream=float(input("please input the icecream consumed yearly:"))
    dataSet,labels=file2matrix("datingTestSet.txt")
    res=classify0(array([games,fly,icecream]),dataSet,labels,3)
    print("You will probably like this person:",res)
    return res
classifperson()

```

