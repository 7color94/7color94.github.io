---
layout: post
title: 最近几篇Attention CNNs
categories:
- Research
tags:
- cnn
---

### 1.BAM

[BAM: Bottleneck Attention Module](https://arxiv.org/pdf/1807.06514.pdf)，BMVC 2018

对SENet的改进：1）channel attention和SENet一样；2）SENet只关注channel attention，BAM增加了spatial attention，说白了过几个conv，其中有两个dilated conv去学spatial context information

### 2.CBAM

[CBAM: Convolutional Block Attention Module](https://arxiv.org/pdf/1807.06521.pdf)，ECCV 2018

对SENet的改进：1）SENet采用global average pooling来embed spatial global information，CBAM同时使用了avg/max pooling；2）SENet只关注channel attention，CBAM增加了spatial attention，也是采用avg/max pooling去embed channel global information

无论是SENet，BAM，CBAM，无论是channel/spatial attention branch，相应的branch总得做点什么（比如pooling，dilated conv）去学该branch的context information，而不能简单地过conv和sigmoid ？？

### 3.Non-Local 系列

*“capture long-range dependencies directly by computing interactions between ay two positions, regardless of their positional distance”*

[Non-local Neural Networks](https://arxiv.org/pdf/1711.07971.pdf)，CVPR 2018

[A^2-Nets: Double Attention Networks](https://arxiv.org/pdf/1810.11579.pdf)，NIPS 2018

两篇用Non-Local做分割的，感觉没啥区别：

[Dual Attention Network for Scene Segmentation](https://arxiv.org/pdf/1809.02983.pdf)，AAAI 2019

[OCNet: Object Context Network for Scene Parsing](https://arxiv.org/pdf/1809.00916.pdf)，arxiv 2018

最先有Non-Local思想应该是Attention Is All You Need：

[Attention Is All You Need](https://arxiv.org/pdf/1706.03762.pdf)，NIPS 2017

### 4.PAN

[Pyramid Attention Network for Semantic Segmentation](https://arxiv.org/pdf/1805.10180.pdf)，BMVC 2018

*“Furthermore, high-level features with abundant category information can be used to weight
low-level information to select precise resolution details.”*

*“Our Global Attention Upsample module performs global average pooling to provide
global context as a guidance of low-level features to select category localization details.”*
