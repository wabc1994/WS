# xgboost的理解情况



## GBDT与xgboost的区别情况


GBDT 是以CART 为基础的，决策树为基础的


GBDT 梯度提升决策树的基本区别

当前一次的学习是上一次学习的残差以及优化值


![Vt2zK1.png](https://s2.ax1x.com/2019/06/04/Vt2zK1.png)


1. 传统GBDT 是以CART 作为基分类器， xgboost还支持线性分类器， L


这次的预测是为了拟合上一次的残差情况 

残差residual的问题= 真值- 预测值


## 如何split

split finding algorithms 分支决策算法

- 最后根据公式的情况，最后树该如何成长，分类

是一个np问题的情况，

## 二阶导为何会可能比一阶导数

1. 实际上引入xgboost的话主要是为了能够解耦，能够自定义损失函数的问题，我们主要损失函数有多种分类的情况 
2. 

## 自己的一些简单理解情况

1. G 代表什么，一阶导数的情况

2. gmma参数， T 叶子节点前面的情况
3. lamda ，每个叶子节点前面的权重参数，防止过拟合的情况发生， 



在公式推导过程当中，主要是当前的预测函数f和后面w 同一个东西，索性就把这两个东西在最后的公式里面合并起来了, 和一阶倒数和二阶倒数一起合并起来的情况，

gmma和lamda参数也是两个参数


 当前样本预测值= 依赖前面的y预测情况 当前的预测情况等于前面的预测情况 + 当前这颗树的预测值，
 
当前的G 是怎么得到的， 


xgboost 
主要关注点在于二阶导，


X是 之前的预测值叠加, 三角型X差值是当前的树的预测值f(t)，Y是样本点


当前树的

G 代表损失函数降低的最大为多少， 在分支之后和没有没有分支之前的情况，说白了就是增益的问题， 使用分裂后的某种值减去分裂去的某种值，从而得到增益比的情况

优势所在的情况

w是叶子节点的权重，T是叶子节点的数量，

1. 一阶导数，使用到了二阶导数
2. 使用到了score L2 膜参数情况


决策的过程就是找出最优的决策树， 