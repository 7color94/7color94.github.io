---
layout: post
title: pix2pix和CycleGAN
categories:
- Research
tags:
- gan
---

### 1. [pix2pix](https://arxiv.org/pdf/1611.07004.pdf)

#### 1.1 网络结构

**Generator**

遵循Encoder-Decoder，先下采样，再上采样回到input的尺寸，conv后面一般都跟bn、relu。常见Generator的结构有：ResnetGenerator、UnetGenerator（skip-layer）。具体的网络结构细节可以看[代码](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix/blob/master/models/networks.py)。

**Discriminator**

Discriminator会判断input中的每个N x N的patch是real还是fake，也就是所说的[PatchGAN](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix/issues/39)。

#### 1.2 前向

real_A过Generator得到fake_B

#### 1.3 反向

先定义loss：

- D的目的是区分real和fake，所以当输入(real_A, real_B)时，D的输出得分尽量高，当输入(real_A, fake_B)时，D的输出得分尽量低
- G的目的是欺骗D，即当输入(real_A, fake_B)时，D的输出得分尽量高。另外，G的另外一个监督loss是让fake_B尽量接近real_B，即fake_B和real_B之间挂一个l1 loss

反向传播的时候先固定G优化D，再固定D优化G，具体可以看[代码](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix/blob/master/models/pix2pix_model.py#L99)。

#### 1.4 Image pool

Image pool存放一定数量的(real_A, fake_B)对，帮助D去记忆历史信息，而不仅仅是当前迭代的一对。

### 2. [pix2pixHD](https://arxiv.org/pdf/1711.11585.pdf)

先前的的pix2pix生成的image分辨率为256 x 256，pix2pixHD将生成分辨率提高到了2048 x 1024。

#### 2.1 网络结构

**Coarse-to-fine Generator**

生成器由两部分组成：Global generator network和Local enhancer network，两个network均是先上采样后下采样的架构。

- Global采用pix2pix中的ResnetGenerator
- Local分辨率是Global的4x，Local部分先下采样得到的feature map和Global上采样的feature map融合，去做Local的上采样步骤，得到最终G的输出

Local数量可以增加，每增加一个，分辨率x4，一般设为1个。

**Multi-scale Discriminator**

判断器由三个同样网络结构的子判别器组成，每个子判别器的input分别是三种递减的分辨率。

具体网络结构细节可以看[代码](https://github.com/NVIDIA/pix2pixHD/blob/master/models/networks.py)。

#### 2.2 反向

pix2pixHD修改了pix2pix的loss：

- D的目的是区分real和fake，所以当输入(real_A, real_B)时，D的输出得分尽量高，当输入(real_A, fake_B)时，D的输出得分尽量低
- G的目的是欺骗D，即当输入(real_A, fake_B)时，D的输出得分尽量高。另外，G生成fake_B经过D得到的N层feature map要和real_B的尽可能相似，这是feature matching loss。G生成fake_B经过VGG得到的N层feature map要和real_B的尽可能相似（l1 loss），这是perceptual loss

反向传播的时候先固定D优化G，再固定G优化D，具体可以看[代码](https://github.com/NVIDIA/pix2pixHD/blob/master/train.py#L72)。

#### 2.3 数据

[代码](https://github.com/NVIDIA/pix2pixHD/blob/master/models/pix2pixHD_model.py#L113)显示input label是经过one-hot编码处理的。

### 3. [CycleGAN](https://arxiv.org/pdf/1703.10593.pdf)

CycleGAN解决的是unpair image-to-image translation，可以理解为两个domain之间的transfer。

#### 3.1 网络结构

两个Generator G_A和G_B，两个Discriminator D_A和D_B。

Cycle的定义是这样的：real_A经过netG_A生成fake_B，fake_B再经过netG_B生成rec_A，这是其一。real_B经过netG_B生成fake_A，fake_A再经过netG_A生成rec_B，这是另外一个cycle。

具体细节可以看[代码](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix/blob/master/models/cycle_gan_model.py#L80)。

#### 3.2 反向

**Generator**

G_A要欺骗D_A，所以fake_B经过D_A后的得分要尽量高，另外重构的rec_B要和原先的real_B尽可能相似（l1 loss）。同理G_B要欺骗D_B，所以fake_A经过D_B后的得分要尽量高，重构的rec_A要和原先的real_A尽可能相似（l1 loss）。

另外，CycleGAN还引入了[Identity loss](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix/blob/master/models/cycle_gan_model.py#L114)，当输入是real_B时，G_A生成的idt_A要和real_B保持identity（l1 loss），同理，输入是real_A时，G_B生成的idt_B要和real_A保持一致。

**Discriminator**

D_A的目的是区分real_B和fake_B，D_B的目的是区分real_A和fake_A。

反向传播的时候先固定D_A、D_B优化G_A、G_B，再固定G_A、G_B优化D_A和D_B，具体可以看[代码](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix/blob/master/models/cycle_gan_model.py#L136)。