---
layout: post
title: 理一理卷积神经网络中的前向和反向传播
categories:
- Research
tags:
- cnn
---

通过阅读[[1]](https://cs.nju.edu.cn/_upload/tpl/00/ed/237/template237/paper/CNN.pdf)，我们能从数学角度清楚卷积神经网络的工作原理，i.e.，convolution layer、fully connected layer、max pooling layer的前向和反向传播公式。配合Github开源的代码[[2]](https://github.com/Eniac-Xie/PyConvNet)，我们进一步清楚了各层前向和反向的实现细节。下面，我再简单地理一理。

### 1 convolution layer

#### 1.1 卷积层的前向过程

卷积的过程可以用复杂的内嵌循环粗暴地实现。Caffe作者在[[5]](https://github.com/Yangqing/caffe/wiki/Convolution-in-Caffe:-a-memo)中也提到，优化复杂的嵌套循环不是一件容易的事。为了在很短地时间内实现一个速度不错的卷积层，作者采用了一个trick：用`im2col`函数把所有需要和卷积核进行卷积操作的local patches摊平组织成一个二维大矩阵，并把卷积核也摊平组织成一个二维矩阵。这样，卷积的过程成功转换为GEMM（General Matrix Matrix Multiply）操作，并能用高性能的BLAS库去加速GEMM，如下图所示。

![](http://oiqcl4y9s.bkt.clouddn.com/caffe_conv.jpg)

假设第 $$l$$ 层卷积层的输入为 $$x^l \in \mathbb{R}^{H^l{\times}W^l{\times}D^l{\times}N}$$ ，卷积参数为 $$F \in \mathbb{R}^{H{\times}W{\times}D^l{\times}D}$$ ，输出为 $$Y \in \mathbb{R}^{H^{l+1}{\times}W^{l+1}{\times}D{\times}N}$$ ， $$N$$ 表示batch size。

通过`im2col`函数可以将 $$x^l$$ 摊平成大小为 $$(HWD^{l}){\times}(H^{l+1}W^{l+1}N)$$ 的二维矩阵x_col，卷积核 $$F$$ 摊平成大小为 $$D{\times}(HWD^{l})$$ 二维矩阵w_col。w_col.dot(x_col)出来后reshape成 $$H^{l+1}{\times}W^{l+1}{\times}D{\times}N$$ 就是卷积后的输出 $$Y$$，也即 $$x^{l+1}$$ 。

具体如何实现，可以阅读代码[[2]](https://github.com/Eniac-Xie/PyConvNet)中的函数`conv_forward`，其基于numpy的fancy indexing实现了`im2col`。

进一步分析，x_col矩阵的元素均来源于 $$x^l$$ ，即x_col和 $$x^l$$ 的元素之间存在映射关系。因为 $$x^l$$ 中的某个元素可能在卷积核滑动的过程中参与了不止一次的卷积运算，所以 $$x^l$$ 到 x_col 的元素映射是一对多的关系，x_col到 $$x^l$$ 则是一对一的映射关系。

#### 1.2 卷积层的反向过程

卷积层的反向过程需要计算两个信息：loss function $$z$$ 对卷积参数 $$F$$ 的导数 $$\frac{\partial z}{\partial F}$$ 和 $$z$$ 对输入 $$X$$ 的导数 $$\frac{\partial z}{\partial X}$$。前者用于更新卷积层的卷积参数，后者作为error supervision information往前面的层传。

根据[[1]](https://cs.nju.edu.cn/_upload/tpl/00/ed/237/template237/paper/CNN.pdf)：$$\frac{\partial z}{\partial X}$$ 的计算稍复杂些，它通过对 $${\frac{\partial z}{\partial Y}} F^T$$ 矩阵进行某种操作得到，其中 $$ \frac{\partial z}{\partial Y} $$ 是 $$l+1$$ 层传给当前 $$l$$ 层的error supervision information。 $${\frac{\partial z}{\partial Y}} F^T$$ 的大小和 x_col一致，所以之前讲的元素映射关系可以用到 $${\frac{\partial z}{\partial Y}} F^T$$ 和 $$x^l$$ 之间。由[[1]](https://cs.nju.edu.cn/_upload/tpl/00/ed/237/template237/paper/CNN.pdf)的推导可知， $$\frac{\partial z}{\partial X}$$ 矩阵某个元素的值等于其在 $${\frac{\partial z}{\partial Y}} F^T$$ 矩阵中所有映射位置的元素值之和（一对多的映射关系，所以是所有映射元素之和）。

[[2]](https://github.com/Eniac-Xie/PyConvNet)中`conv_backward`通过`numpy.add.at`函数实现了这种相应映射元素之间的相加运算。

### 2 fully connected layer

全连接层完全可以当成卷积层去看待：如果全连接层的输入 $$x^l \in \mathbb{R}^{H^l{\times}W^l{\times}D^l}$$ ，我们可以用大小为 $$H^l{\times}W^l{\times}D^l{\times}D$$ 的卷积核模拟全连接。那么，全连接层的前向和后向完全可以复用`conv_forward`和`conv_backward`实现。

### 3 max pooling layer

#### 3.1 max pooling层的前向过程

`max_pooling_forward`思路和`conv_forward`类似。假设max pooling layer的输入为 $$x^l \in \mathbb{R}^{H^l{\times}W^l{\times}D^l{\times}N}$$，pooling层超参数为 $$H{\times}W$$，输出 $$Y \in \mathbb{R}^{H^{l+1}{\times}W^{l+1}{\times}D^l{\times}N}$$。

将 $$x^l$$ 摊平成大小为 $$HW{\times}H^{l+1}W^{l+1}DN$$ 的二维矩阵 x_col，x_col的每一列表示需要进行`max`运算的一个local patch。摊平运算完全可以复用`im2col`函数。

#### 3.2 max pooling层的反向过程

max pooling layer没有参数，所以只需要计算 $$\frac{\partial z}{\partial X}$$ 。根据[[1]](https://cs.nju.edu.cn/_upload/tpl/00/ed/237/template237/paper/CNN.pdf)， $$\frac{\partial z}{\partial vec(x^l)}$$ 由 $$\frac{\partial z}{\partial vec(y)}$$ 进行元素映射得到。具体过程阅读代码[[2]](https://github.com/Eniac-Xie/PyConvNet)中的`max_pooling_backward`。

### 4 结束语

其他比如relu layer、softmax loss layer也都比较简单。

文章理得也不算清楚，所以还是建议直接阅读[[1]](https://cs.nju.edu.cn/_upload/tpl/00/ed/237/template237/paper/CNN.pdf)，并配合着[[2]](https://github.com/Eniac-Xie/PyConvNet)一起。读代码过程中善于利用`pdb`调试工具，这样理解得更清晰些。

### 5 参考资料

- [[1] Introduction to Convolutional Neural Networks](https://cs.nju.edu.cn/_upload/tpl/00/ed/237/template237/paper/CNN.pdf)
- [[2] A python implementation of convolutional neural network](https://github.com/Eniac-Xie/PyConvNet)
- [[3] CS231n Convolutional Neural Networks for Visual Recognition](http://cs231n.github.io/convolutional-networks/#architectures)
- [[4] 知乎：在Caffe中如何计算卷积](https://www.zhihu.com/question/28385679)
- [[5] Convolution in Caffe: a memo](https://github.com/Yangqing/caffe/wiki/Convolution-in-Caffe:-a-memo)
