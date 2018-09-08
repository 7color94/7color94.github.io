---
layout: post
title: CNN Base Models
categories:
- Research
tags:
- cnn, deep learning
---

#### 1.LeNet

[Gradient Based Learning Applied to Document Recognition](http://vision.stanford.edu/cs598_spring07/papers/Lecun98.pdf)，1998

paper中input为32x32x1，我实现的时候怎么好像是28x28x1，然后两层的conv（5x5，stride=1）、pooling（2x2，stride=2），之后fc、relu，最基本的cnn，一般用于手写识别

#### 2.Alxnet

ILSVRC 2012 winner 

[ImageNet Classification with Deep Convolutional Neural Networks](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf)，NIPS 2012

输入224x224x3，五层的conv、pooling，使用relu、dropout，还有data augmentation、weight decay

实验一张卡放不下，把model拆成2份放到两张卡训练

#### 3.VGG

[Very Deep Convolutional Networks for Large-Scale Image Recognition](https://arxiv.org/pdf/1409.1556.pdf)，ICLR 2015



#### References

- [lec01_cnn_architectures.pdf](http://slazebni.cs.illinois.edu/spring17/lec01_cnn_architectures.pdf)