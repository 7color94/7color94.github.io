---
layout: post
title: Face Alignment Summary
categories:
- Research
tags:
- face alignment
- paper
---

以我目前所读论文为依据，现有算法大体上分为两种类型：

- 1.Classifying Search Windows
- 2.Directly Predict Key Point Positions（or Shape Parameters）

暂时没有接触第一种category的paper，第二种还可以细分为两种：

- 1.基于模型（Model Based），代表有AAM、ASM
- 2.基于回归（Regression Based）。还可以细分为基于线性回归的，代表有CPR, LBF等；基于非线性回归的，代表有DCNN、CFAN、TDA等

对每个paper我关注以下几点：

- 属于那种框架或模型：基于Model？Cascaded（级联）模型？基于Deep Learning的非线性回归模型？等等
- 优缺点：算法的Speed、Accuracy；是否利用特征点之间的相对位置关系（Geometric/Shape Constraints）以增强鲁棒性；对Occlusion、Pose Variations、Lighting的是够敏感
- Feature是如何提取的，使用的是Local Feature还是Global Feature

### Model Based

基于模型的论文都对人脸建立了一定的模型，也是论文的核心idea，较为出名的模型有：AAM模型、ASM模型、CLM模型。它们提出的模型可以刻画出人脸图像，所以只需要求得特定人脸对应的模型参数（即Directly Predict Shape Parameters），即可合成相应的人脸图像

ASM（Active Shape Model）是一种基于点分布模型（Point Distribution Model）的算法，AAM（Active Appearance Model）则在ASM基础上，进一步对纹理进行建模，并将形状和纹理两个统计模型进一步融合为表观模型。其中ASM为每个特征点构建局部特征，AAM则使用全局纹理信息

AAM模型拟合过程如下：

![](http://7xl2fd.com1.z0.glb.clouddn.com/AAM.png)

基于模型的方法难点在于人脸模型的建立，而且对初始值很敏感，鲁棒性不高

这方面论文大都在1998年左右发表，只看了4篇，理解不多，就简略介绍

### Regression Based

该方法尝试学习 从Deteced Face Region 到 Facial Landmarks的 Mapping，有基于线性和非线性Regressor的两种方案

#### 1.线性Regressor

1.1 Cascaded Pose Regression（CVPR 10）

![](http://7xl2fd.com1.z0.glb.clouddn.com/CPR.png)

级联回归模型在CVPR 10首次提出。它的核心思想是学习多个回归器，每个回归器预测当前形状和真实形状之间的偏差（也叫增量），然后将偏差合成（想加）到当前形状，形成新的预测形状，作为下一个回归器的输入，从而逐步逼近真实值

![](http://7xl2fd.com1.z0.glb.clouddn.com/CPR-Test.png)

每个回归器均不一样，需要预先提取特征，作为回归器的输入，特征采用Pose-Indexed Features（不是很理解特征提取这方面，懂的人可以告知本人，不胜感激），也可以使用SIFT、HOG等人工设计的特征；回归函数采用Random Fern Regressors

CPR有明显的缺点：其对初始化形状非常敏感，论文中使用K次不同的初始化形状来测试，并融合K次的测试结果作为最终预测形状。而且CPR不能有效解决Occlusion问题

CPR出现后，之后的回归模型算法大都基于级联框架，统一称作级联回归模型，只是不同的级联回归算法使用的Regressor和特征不同，别的基本一致，所以级联框架的提出算是Face Alignment发展史的伟大一步

1.2 Face Alignment by Expicit Shape Regression（CVPR 12）

论文设计了Two-Level Boosted Regression增强了鲁棒性，使用Correlation-Based Feature Selection方案来提取Shape Indexed Feature，大大提高了时间效率

Two-Level Boosted Regression具体是指：External-level和Internal-level，即boosted regression和two level cascaded的结合。其中最原始的回归器r基于Random Fern学习

![](http://7xl2fd.com1.z0.glb.clouddn.com/ESR.png)

而且，论文注意到形状约束的重要性。它认为只有那些显著的特征点（眼睛中心、嘴巴边缘）可以由image appearance可靠地得到，其余不显著的Landmarks需要使用特征点间的shape constraint进行约束。先前的工作如ASM、AAM利用参数化的形状模型（PCA）来强制这种约束，而本文则抛弃不灵活的固定模型，使用boosted regression，使得shape constraint adaptively enforced from coarse to fine，论文的结论是最终的shape是初始形状和所有训练形状的线性组合，只要初始形状满足形状约束，得到的regressed shape在由所有训练的shape构成的线性子空间中也满足形状约束（具体证明见论文），所以和之间预先固定的PCA shape model相比，论文非参数化的形状约束adaptively determined

1.3 Using Conditional Regression Forests（CVPR 12）

![](http://7xl2fd.com1.z0.glb.clouddn.com/CRF.png)

论文不同于基于Regression Forest的方法，它的核心是基于Conditional Regression Forests。一般基于Regression Forest学习到facial image patches和location of feature points之间的mapping，而conditional regression forest则基于face image的某些属性预先进行分类，然后基于特定分类下的regression forest学习相应的mapping

如图，可以依据head pose将图片分为5种Label：profile left；left；front；right；profile right

1.4 Robust Face Landmark estimation under occlusion（ICCV 13）

又名：Robust Cascaded Pose Regression（RCPR），旨在解决CPR的遮挡问题，它在预测landmarks的同时预测特征点被遮挡的状态

![](http://7xl2fd.com1.z0.glb.clouddn.com/RCPR.png)

同时RCPR对CPR的初始化敏感问题也提出了改进，即智能重启：随机初始化一组形状，运行至级联模型的前10%回归器，统计形状预测的方差，如果方差小于一定阈值，说明这组初始化不错，则跑完剩下的90%的级联函数，得到最终的预测结果；如果方差大于一定阈值，则说明初始化不理想，选择重新初始化一组形状

1.5 DRMF（CVPR 13）

论文首先用Constrained Local Models（CLM模型）对人脸进行建模。之后DRMF利用SVR对回归器建模，并使用HOG特征（相对的有：SIFT特征）最为回归函数的输入，最终预测出CLM的人脸模型参数，而不是直接预测人脸的形状。由于模型很难刻画出完整的人脸变化形状，所以鲁棒性欠缺

1.6 Regressing Local Binary Features（CVPR 14）

![](http://7xl2fd.com1.z0.glb.clouddn.com/LBF2.png)

![](http://7xl2fd.com1.z0.glb.clouddn.com/LBF1.png)

如图，LBF使用了Random Forest作预测，但LBF并没有直接采用随机树中叶子节点存储的形变量作为最终预测结果，而是将输出组成二值化特征（LBF），再利用这个LBF来作最终的形变预测

总的来说，LBF方法第一步为每个landmark独立建立随机森林，形成各自的Local Binary Feature；然后连接所有的LBF作全局线性回归（这样可以有效利用关键点间的约束信息）得到最终形变量。其中第一步基于random forest学习得到Feature Mapping需要使用到shape indexed feature，也因为LBF非常稀疏，所以计算速度很快

1.7 Supervised Descent Method

SDM将Face Alignment看作非线性最小二乘问题。通常求解非线性最小二乘问题可以通过二阶的牛顿法解决，然而在computer vision任务中，某些函数不一定可微，而且Hessian矩阵位维数很大且不一定正定，所以论文提出了SDM算法来最小化非线性最小二乘函数，即以监督学习方式求解梯度方向

在训练阶段，SDM学习每个特征点的非线性最小二乘函数的梯度方向，组成梯度序列。之后测试阶段，就可以直接使用学习好的梯度方向来最小化非线性最小二乘函数，从而避免直接计算Jacobian或Hessian矩阵

通过论文可以发现，因此需要学习一组梯度序列，该方法属于级联回归模型

1.8 Face Alignment with an Ensemble of Regression Trees（CVPR 14）

论文基于集成回归树学习从input image到landmarks的映射；而且论文为image中每个landmark引入权重W可以解决训练数据标记缺失的问题

#### 2.非线性Regressor：利用CNN进行非线性建模（级联卷积神经网络）

2.1 CUHK的DCNN（CVPR 13）提出了3层框架

![DCNN](http://7xl2fd.com1.z0.glb.clouddn.com/DCNN.png)

第一层使用整个face image作为input region（所以特征是global high-level feature），得到了initial predicitons，所以该层为initialization stage。而且该层同时进行不同key points的预测，所以能有效利用geometric constraints

之后的二三层对第一层的结果进行refine，每层的cnn以当前预测点为中心，提取local feature，预测出当前facial points和ground truth之间的偏差

可以看出DCNN采用coarse-to-fine的步骤，一步一步逼近ground truth，因此也属于cascaded（级联）模型

2.2 CFAN（ECCV 14）

![](http://7xl2fd.com1.z0.glb.clouddn.com/CFAN.png)

与DCNN类似，也是coarse-to-fine的级联模型，第一个CNN预测出初始值，后面的CNN用于refine从而逼近真实值。DCNN二三层是对第一层初始结果的refine，使用没有shape constraint的local features，因此容易得到局部最优值，对occlusion等影响因素的鲁棒性不理想。CFAN解决了这个问题，在同样的第一层处理之后，之后的每层使用point之间的约束信息（constraint），因此CFAN对第一步init值的敏感度要比DCNN低。另外，CFAN的每层的input image的分辨率逐渐提高，第一步用于快速的预测初始值，低分辨率的图片对应于large search step，后面紧跟的refine用作局部调整，则选择使用高分辨率的image作为input

2.3 TDA（ACCV 14）

![](http://7xl2fd.com1.z0.glb.clouddn.com/TDA.png)

TDA与DCNN、CFAN大体类似，其不同之处是先根据Face Image的Topic进行分类，然后使用每个分类下独立特有的CNN网络进行处理

这里的Topic分类是指通过k-means clustering对人脸图片进行分类，来决定人脸图片属于5中Topic中的哪一类，以决定之后使用某具体分类下的CNN网络。和级联回归模型中的Conditional Regression Forests（利用face image的head pose对face进行分类）想法类似

![](http://7xl2fd.com1.z0.glb.clouddn.com/TDA-Topic-Aware.png)

可以看出上面三种级联深度模型都将Image To Shape的mapping分解到级联框架的各个stage分别进行训练或处理，而且基于CNN对非线性映射进行建模，也免于特征的手动提取

### References

- <a href="http://www.cnblogs.com/gavin-vision/p/4829016.html" target="_blank">Cascade Pose Regression</a>
- <a href="http://blog.csdn.net/santo_wong_94/article/details/49330295" target="_blank">阅读Face Alignment by Explicit Shape Regression</a>
- <a href="http://blog.luoyetx.com/2015/08/face-alignment-at-3000fps/" target="_blank">Face Alignment at 3000 FPS via Regressing Local Binary Features</a>
- <a href="http://chuansong.me/n/290740451748" target="_blank">深度大讲堂：面部特征点定位概述及最近研究进展</a>

作为CV新手，文章不免会有许多理解上的错误