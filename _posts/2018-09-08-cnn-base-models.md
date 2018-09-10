---
layout: post
title: CNN Base Models
categories:
- Research
tags:
- cnn, deep learning
---

### 1.LeNet

[Gradient Based Learning Applied to Document Recognition](http://vision.stanford.edu/cs598_spring07/papers/Lecun98.pdf)，1998

一般用于手写识别，paper中input为32x32x1，我实现时的mnist数据集好像是28x28x1，然后两层的conv（5x5，stride=1）、pooling（2x2，stride=2），之后fc、relu，最基本的cnn

### 2.Alxnet

[ImageNet Classification with Deep Convolutional Neural Networks](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf)，NIPS 2012

输入224x224x3，五层的conv、pooling、relu，为了防过拟合使用了dropout和data augmentation（1.extract 224x224 from 256x256，2.horizontal reflections，3.alter the intensities of the RGB channels），还有weight decay

实验时一张卡放不下，把model拆成2份放到2张卡训练

### 3.VGG

[Very Deep Convolutional Networks for Large-Scale Image Recognition](https://arxiv.org/pdf/1409.1556.pdf)，ICLR 2015

使用3x3 conv构成两种sequence结构：VGG-16和VGG-19

VGG的方法是一层一层地堆conv，继续增加深度会有训练困难、参数量增加等问题

### 4.GoogleNet系列

#### 4.1 Inception v1

[Going Deeper with Convolutions](https://www.cs.unc.edu/~wliu/papers/GoogLeNet.pdf)，CVPR 2015

设计一种较宽的Inception module，（a）naive版本：将1x1，3x3，5x5的conv和3x3的pooling，都stack在一起，一方面增加了网络的width，另一方面增加了网络对尺度的适应性. (主要是因为1x1, 3x3, 5x5, 3x3 pooling的作用都不一样，索性都stack在一起，让模型自己选)（b）降维版本：在计算量大的conv之前，先用1x1降维，减少计算量

网络的组成：在低层的时候仍用传统的卷积方式（sequence结构），高层开始堆Inception module；中间loss监督防止梯度消失；使用global average pooling

#### 4.2 Inception v2

[Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/pdf/1502.03167.pdf)

使用Batch normalization，将每层输入归一化到N(0,1)的高斯分布

网络结构方面学习VGG用2个3x3代替一个5x5，参数了变少，但感受视野一样

#### 4.3 Inception v3

[Rethinking the inception architecture for computer vision](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Szegedy_Rethinking_the_Inception_CVPR_2016_paper.pdf)，CVPR 2016

卷积进一步分解，5x5用2个3x3卷积替换，7x7用3个3x3卷积替换，3x3卷积核可以进一步用1x3和3x1的卷积核组合来替换，7x7分解成两个一维的卷积（1x7,7x1），进一步减少计算量，好处，既可以加速计算（多余的计算能力可以用来加深网络），又可以将1个conv拆成2个conv，使得网络深度进一步增加，增加了网络的非线性

#### 4.4 Inception v4

[Inception-v4, Inception-ResNet and the Impact of Residual Connections on Learning](https://arxiv.org/pdf/1602.07261.pdf)，AAAI 2017

探索residual connection对Inception module的影响：收敛加速，但是最终效果好像提升很少。paper中提出了一些residual和 non-residual Inception networks

#### 4.5 Xception

[Xception: Deep Learning with Depthwise Separable Convolutions]()，CVPR 2017

Xception将分解的思想推到了极致：跨通道的相关性和空间相关性是完全可分离的，最好不要联合映射它们，先pointwise + relu再depthwise + relu（和mobilenet相反）

### 5.ResNet

[Deep Residual Learning for Image Recognition](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf)，CVPR 2016 Best Paper

提出Residual Learning，两种bottleneck：1.3x3xc，3x3xc ; 2.1x1x(c/4)，3x3x(c/4)，1x1xc

[Identity Mappings in Deep Residual Networks](https://arxiv.org/pdf/1603.05027.pdf)，ECCV 2016

做实验探索residual bottlneck里面是conv，bn，relu还是bn，relu，conv，实验效果后者好，但是一般我还是用前者，TF和Pytorch放出来的Imagenet pretrained model基本都是前者

### 6.DenseNet

[Densely Connected Convolutional Networks](http://openaccess.thecvf.com/content_cvpr_2017/papers/Huang_Densely_Connected_Convolutional_CVPR_2017_paper.pdf)，CVPR 2017

DenseNet将residual connection思想推到极致，每一层输出都直连到后面的所有层，可以更好地复用特征，每一层都比较浅，融合了来自前面所有层的所有特征，很容易训练。缺点是显存占用更大并且反向传播计算更复杂一点

### 7.ResNeXt

[Aggregated Residual Transformations for Deep Neural Networks](http://openaccess.thecvf.com/content_cvpr_2017/papers/Xie_Aggregated_Residual_Transformations_CVPR_2017_paper.pdf)

借鉴了Inception加宽的思想，使用分组卷积，所以计算量减少，bottleneck的维度可以适当增加，效果提升：1x1x(c/2)，3x3x(c/2)，1x1xc

### 8.DPN

[Dual Path Networks](https://papers.nips.cc/paper/7033-dual-path-networks.pdf)，NIPS 2017

把ResNeXt（feature re-usage）和DenseNet（new features exploration）合并

### 9.WRN

[Wide Residual Networks](https://arxiv.org/pdf/1605.07146.pdf)，BMVC 2017

把ResNet变宽：增加output channel的数量来使模型变得更wider，深度可以不用太深了

### 10.SENet

[Squeeze-and-Excitation Networks](https://www.robots.ox.ac.uk/~vgg/publications/2018/Hu18/hu18.pdf)，CVPR 2018

Feature map的channel-wise attention

### 11.NASNet

[Learning Transferable Architectures for Scalable Image Recognition](https://arxiv.org/pdf/1707.07012.pdf)

Google的AutoML

### 12.MobileNet

#### 12.1 MobileNet v1

[MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications](https://arxiv.org/pdf/1704.04861.pdf)，CVPR 2017

用depth wise conv + point wise conv替代标准conv，减少计算量

#### 12.2 MobileNet v2

[MobileNetV2: Inverted Residuals and Linear Bottlenecks](https://arxiv.org/pdf/1801.04381.pdf)，arXiv 2018

Paper在探索这样的问题：如何把residual bottleneck应用到mobile net v1？一种改进是通过先扩张在收缩的方式，让depth wise conv提取的特征更丰富些，还有就是在和indentify mapping元素相加时去掉了relu，因为paper做实验证明relu只适合用于维度多的feature map的激活。具体可以阅读：[mobilenet v2解读](https://blog.csdn.net/u011995719/article/details/79135818)

### 13.ShuffleNet

#### 13.1 ShuffleNet v1

[ShuffleNet: An Extremely Efficient Convolutional Neural Network for Mobile Devices](http://openaccess.thecvf.com/content_cvpr_2018/CameraReady/0642.pdf)，CVPR 2018

进一步用group conv + channel wise替代mobilenet中的point wise conv

#### 13.2 ShuffleNet v2

[ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design](https://arxiv.org/pdf/1807.11164.pdf)，ECCV 2018

### 14.Analysis

[An Analysis Of Deep Neural Network Models For Practical Applications](https://arxiv.org/pdf/1605.07678.pdf)

从paper的Figure. 2可以看出，比较划算的是Inception、Resnet系列

### References

- [从LeNet到SENet——卷积神经网络回顾](https://www.leiphone.com/news/201802/31oWxcSnayBIUJhE.html)
- [lec01_cnn_architectures.pdf](http://slazebni.cs.illinois.edu/spring17/lec01_cnn_architectures.pdf)
- [mobilenet v2解读](https://blog.csdn.net/u011995719/article/details/79135818)