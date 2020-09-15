## Introduction
* 利用空白频谱，一个基本的问题是识别被占用的和空闲的频道。
* 说了WISER的缺点，it simply uses the anchor sensors’ readings to represent those of other locations, which is likely to lead to errors and loss of white spaces.
* 本文提出一种矩阵补全的方法，作用是减少感知开销并且提高识别的准确率。

## 介绍室内空白频谱的特征
1. low-rank structure
2. temporal stability
3. spatial and spectral correlations

## 实验设备、场地和方法
### 获取实验数据
  * 上交的SEIEE楼的3楼，67个室内感知点，在每个点上记录接收到的45个TV频道的信号强度。
  * 使用一个小推车，分时在67个点记录数据，得到一个45*67的矩阵，叫做RSS矩阵（Received Signal Strength Matrix）
  * 在两个月内，做完20次记录，得到实验用的数据————20个RSS矩阵
### 分析数据的特征
#### low-rank structure
   * 使用PCA，获取RSS矩阵的奇异值，发现只有前5%的奇异值有用，故RSS矩阵可以压缩成一个低秩矩阵
#### temporal stability
   * 计算相邻时间点的矩阵中节点的normalized difference（归一化差值？），发现超过80%的值都非常小，故存在时间稳定性特征
#### spatial and spectral correlations
   1. 空间相关性: there are strong correlations between multiple locations in their spectrum usage patterns.
   2. 频谱相关性: for multiple channels, there are strong correlations in their RSS across different locations.

## WhiFind系统
### 简介
  * 由一个空白频谱的数据库和一定数量的室内传感器组成
  * 每个地点的传感器记录频道的利用率，并发送到数据库
  * 数据库采用矩阵补全的方法来计算没有布置传感器的地点的RSS矩阵
### 具体做法
  * 为了实现此功能，我们首先在所有位置进行频谱分析。 然后，我们在这些位置上执行k-medoids聚类，并选择每个聚类的中心来部署我们的传感器。 数据库定期从传感器接收实时读数，然后执行矩阵完成方法来计算RSS矩阵中的未知条目。
### 矩阵填充方法
  * 矩阵填充理论[系统学习]
  * RIP：有限等距性质（Restricted Isometry Property）[系统学习]
  * 核范数是矩阵奇异值的和
  * F范数是矩阵奇异值平方和的开方
  * 拉格朗日乘数法
  * Toeplize矩阵是对角线为常量的矩阵，matlab中toeplize(1, -1)
  * KNN
  * 线性回归
  * 交替最小二乘法（ALS, alternating least squares）
 ## 评价指标
  * FA and FN rates




