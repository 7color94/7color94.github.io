---
layout: post
title: Kaggle facial keypoints detection
categories:
- Research
tags:
- deep learning
- kaggle
- face alignment
---

Kaggle平台有关Facial Keypoints Detection的[competition](https://www.kaggle.com/c/facial-keypoints-detection)，虽然发起时间是13年，但是直到前几天还是有人在上面提交测试数据。

如果说，mnists是deep learning的hello world，那么鄙人觉得这款比赛是face alignment的hello world（难度比hello world稍大一些）。

早在14年就有篇博文对比赛做了非常详细的解剖：[Using convolutional neural nets to detect facial keypoints tutorial](http://danielnouri.org/notes/2014/12/17/using-convolutional-neural-nets-to-detect-facial-keypoints-tutorial/)。如果我能早2个月就看到这篇文章的话，我相信我会少走很多弯路。虽然文章是14的，里面有关数据的预处理和模型的建立和训练思想算是相当经典。

首先是数据的预处理，文章对image pixels / 255 做了[0, 1]的映射处理，对keypoints做了[-1, 1]的映射处理。这些tricks已经是公开的秘密了。如果让我选择的话，我会把pixels映射到[-1, 1]。

起初先采用了简单的线性三层感知器去拟合keypoints和pixels之间的映射关系。之后用了手写识别系统网络LeNets引入cnn。后来，利用flip对数据做了一定的augmentation。再后来介绍如何调learning rate，并引入dropout避免过拟合。

为了使用那些含有缺失值的训练数据，文章提出了multi-task的思想，每个task去预测不同的keypoints集合。而且可以不用从头开始训练，可以在前面训练好的网络上fine tune。

我最感兴趣的是文章中的online augmentation，以及他对缺失值数据的利用。我准备用caffe对两点思想作次实践。

首先，online augmentation和offline augmentation还是有区别的，摘自[Kaggle](https://www.kaggle.com/c/datasciencebowl/forums/t/12597/differences-between-real-time-augmentation-and-preprocess-augmentation)：

> I have done this comparison. I used to do offline data augmentation because it is relatively easier to implement based on open source code, but I finally switch to online data augmentation for the following reasons:

> 1.For online data augmentation, the model see one random generated sample only once, but it is not the case for offline data augmentation (unless you only train one epoch on the offline generated data). As the model see more samples in online training, the model trained with online data augmentation generalize better than the model trained with offline data augmentation. According to my comparison, the score obtained by online augmentation is consistently better than the offline data augmentation. Though the gap will be narrowed if the more times offline data augmentation is applied.

> 2.It is very annoying (sometimes even intractable) to store very large amount of offline augmented data. This issue will become even worse when you try to study various data augmentation strategies.

还有一点个人的理解，online random augemtation能产生无限多的训练集？

Data input layer首先继承caffe.Layer类，之后实现setup, reshape forward和backward。

- setup完成初始化任务，主要从prototxt param_str读取batch_size，phase，data file_name等信息。
- reshape在取数据时，调整top矩阵的shape。我会在这个函数中完成数据的读取工作。
- forward即前向，将数据往前传递。就是简单地将reshape中读取的数据赋值给top，以便前向传播。之后对当前迭代计数idx + 1。
- backward后向在数据层不需要

训练数据预先保存在h5文件中，通过file_name给出文件位置。为了避免重复读取，在小数量的情况下，通过自定义BatchLoader类，在Data input layer setup函数调用时，完成类初始化，并将所有数据读入内存。BatchLoader则负责抛出get_batch(self, idx)接口，根据当前迭代计数，取出训练样本中的具体某段数据，并完成random flip，即online augmentation。

base learning rate 0.001，总共迭代3000次，在200左右基本收敛了。我也没有去刻意调参优化。

![](http://oiqcl4y9s.bkt.clouddn.com/kaggle-caffe-facial-point-predict.PNG)

在validation集上挑选一些照片预测

![](http://oiqcl4y9s.bkt.clouddn.com/validation-img-1.PNG)
![](http://oiqcl4y9s.bkt.clouddn.com/validation-img-3.PNG)
![](http://oiqcl4y9s.bkt.clouddn.com/validation-img-4.PNG)
![](http://oiqcl4y9s.bkt.clouddn.com/validation-img-5.PNG)
![](http://oiqcl4y9s.bkt.clouddn.com/validation-img-7.PNG)

和我预想的差不多，对于那些正脸简单的图片，随便一回归就能预测准确。但是大pose，遮挡这类依然是特征点领域的难题。

目前大多数论文只是通过喂数据的方式缓解这一难题。利用深度学习，狂轰滥炸喂数据，以提高model泛化能力。小实验室的人玩不起...有一些paper从特征角度去考虑点与点之间约束信息，比如LBF特征。还有的论文通过预先的PCA处理，将某图片分类到具体属性模型下，比如针对侧脸的图片单独训练一个model，针对微笑的人脸单独训练一个model，最后通过ensemble整合。

之后，因为有些训练样本会缺少某些keypoint的坐标，所以文章为每个keypoint生成不同的数量的训练集合，将keypoints划分为不同的集合，分别对每个集合训练，训练过程引入early stopping，不用等到max iters次数，根据validation error决定是否early stop。

第二点我没有实践，但是这些思想非常值得借鉴。只做了些微量的实践，代码发布在了[Github](https://github.com/7color94/kaggle-facial-keypoints-detection-caffe)上。