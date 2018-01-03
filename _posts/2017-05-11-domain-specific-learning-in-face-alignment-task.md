---
layout: post
title: Domain specific learning in Face Alignment Task
categories:
- Research
tags:
- face alignment
- paper
---

最近读了一些Face Alignment的文章，核心思想都是 “将人脸样本划分成多个域（domain），然后分域进行处理”

Reading Lists:
- 《Unconstrained Face Alignment via Cascaded Compositional Learning》 CVPR 2016
- 《Dynamic Attention-controlled Cascaded Shape Regression Exploiting Training Data Augmentation and Fuzzy-set Sample Weighting》 CVPR 2017

#### 1. 首先，什么叫对人脸样本划分domain，以及为啥要划分domain？

如果对于frontal且没有什么复杂表情的人脸，简单的级联回归其实已经可以完全解决其特征点定位问题。正是由于人脸受姿态、表情等影响，特征点定位问题开始变得复杂，我们需要将这些影响考虑在模型中。比如，针对不同姿态，将人脸样本集划分为frontal、left、right三种子姿态数据集，这就是最简单的域划分。划分后的每个子姿态数据集中face的姿态变化方差相对于整个数据集大幅度减小，子姿态数据集中的初始形状也离最终目标形状更加接近，加速了形状回归收敛的速度。此外，针对每个子姿态数据集训练domain-specific的形状预测模型，这样的模型对于相应特定的子姿态样本有着更强的预测能力。这也就是为什么近几年的文章开始划分域去训练的原因。

#### 2. 怎么划分域？

刚刚也已经讲到，利用head pose estimator可以将face分为frontal、left、right三种子数据集。当然也存在更加细粒度的划分方法，其中常见的方法有利用PCA从face训练样本集合中提取shape和local appearance主成分，然后利用主成分划分域的，下面会介绍。

#### 3. 划分域之后，模型如何训练和测试？

划分为多个域，就相当于将训练样本集合（包括测试样本集合）划分为多个子集合。

- 训练

训练的时候我们针对每个domain都训练其domain特定的model。一种方法是仅利用相应domain的子集合数据去训练；另一种则利用所有子集合样本的加权loss去训练，与当前domain越相关的样本权重越大。也是下面介绍的两篇论文分别采用的方法。

- 测试

测试的时候，可以选择当前测试样本所属domain的model进行预测，也可以利用所有domain的model加权预测作为最终预测值。相应地，与测试样本越相关的domain权重最大。也是下面介绍的两篇论文分别采用的方法。

下面再简单记录这两篇论文的阅读笔记。

### 1. Cascaded Compositional Learning

![](http://oiqcl4y9s.bkt.clouddn.com/CCL_formulations.jpg)

常见的cascaded regression regression形如公式（1），学习得到的线性回归器W代表shape indexed features到shape residual的映射关系。大部分级联方法都将所有样本等同对待，从整个训练集合中学习得到共享的W。然而上面也说了，人脸受large pose、appearance等影响，同等看待的方法不具备鲁棒性，共享的W不会特别适合某一个特定的样本。比如，SDM方法只对相同梯度的space有效，具有相同pose的样本属于同一domain，此domain内样本的W应该相似，不同domain的W应该不会相似的，所以W应通过划分域的方式分开训练。

#### 1.1 CCL的域划分算法

如何划分域？论文利用PCA对训练样本提取shape和local appearance的主成分，每个主成分将训练样本对半分，所以能划分出K个domain，K is a power of 2。head pose并不是唯一也不是最好的划分方式，还有其余一些主导域划分的影响因素，比如shape deformation或者appearance property：wide-open mouth、large facial scaling、large face contour等。

#### 1.2 CCL的预测算法

![](http://oiqcl4y9s.bkt.clouddn.com/CCL.png)

Cascaded Compositional Learnings算法预测时，同时对多个domain预测出的shape residual进行加权组合，作为最终预测的shape residual。至于怎么加权，就是CCL算法的核心思想了。

#### 1.2 CCL的训练算法

如何学习特定域的w：k-th domain具有相同的梯度，那么直接利用ridge regression（公式 4）就可以了，和常见级联方法一样。特定domain的w用特定domain内的样本进行训练。

#### 1.3 如何得到预测时domain的加权系数？

学习每个domain的组合权重P：如果把每个domain看作class，那么公式3中的f可以看成multi-class classifier。这么做存在一定问题：1. 次优的domain也能对样本提供相对准确的shape prediction，但是multi-class classifier将除最优domain外所有其余domain都看作negative class。2. multi-class classifier对不靠谱的domain惩罚太轻。如果一个样本属right-profile-view domain，那么它一定不会属于left-profile domain。 所以论文转而直接去优化公式5。

#### 1.4 CCL采用的特征

CCL算法的特征是在LBF上进行改进。略

### 2. Fuzzy-set Sample Weighting（DAC-CSR）

方法和CCL极其相似，CCL在推断的时候对domain预测出的shape进行加权组合；而DAC-CSR方法则取消了推断时的加权组合，转而在训练时候利用所有domain的样本的加权loss去学习某个特定domain的W。我认为这就是二者的本质区别，一个是在推断过程，一个是在训练过程。

![](http://oiqcl4y9s.bkt.clouddn.com/DAC-CSR.png)

DAC-CSR算法分三步骤：1. face bounding box refinement 2. general CSR 3. attention-controlled CSR

1、2步和常见级联算法没区别，第3步是选择特定domain的CSR对第2步预测出的shape进行refine。

和CCL的不同之处：

- domain划分：DAC-CSR在PCA基础上，划分出K个domain后，又多加入一个domain，也就是图片中间那块domain。文章给出的解释是sub-domains之间有交集能提升model的鲁棒性、容忍度。

![](http://oiqcl4y9s.bkt.clouddn.com/DAC-CSR-domains.png)

- 特定domain的w学习：CCL只使用domain内的样本训练W；而DAC-CSR则使用所有样本加权loss去训练特定domain的W，属于该domain的样本权重大，不属于domain样本的权重小。
- 预测：CCL划分domain后，在推断的过程中对各个domain预测出的shape加权组合。而DAC-CSR则是仅采用特定domain的model的预测值，而且是动态选择方法，在每层级联预测之前，利用上一级预测出的形状重现选择domain。

此外，DAC-CSR还提出了一种新的Data Augmentation方法：2D Profile Face Generation，warp a face image to another pose。和AAM训练时将人脸抠出来，利用shape到mean shape的仿射变换关系和三角剖分，将人脸变形到平均脸。这里则是将人脸变形到另外一个pose。
