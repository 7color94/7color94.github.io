---
layout: post
title: 论文《Face Alignment via Regressing Local Binary Features》解读
categories:
- Research
tags:
- face alignment
- paper
---

暑假阅读了《Face Alignment at 3000 FPS via Regressing Local Binary Features》，今天来好好总结下LBF。

在我所读论文中除了传统的SIFT、HOG、"shape-indexed" features（两点间像素值之差，其实LBF是对shape indexed feature的改进）以及deep特征外，就是LBF特征了。而且LBF显式地encode点之间的约束信息。

论文说hand-designed SIFT feature属于通用特征，不适用于特定的face alignment任务，但是以往任务相关的shape indexed feature效果却和SIFT打平手。论文分析ESR和RCPR的shape indexed feature因为使用了entire face region作为输入，所以特征存在许多噪音。

LBF论文则提出自己的见解：

- 最有效的判别纹理信息往往在级联上一步估计出的landmark周围区域（local texture）
- landmarks之间是有约束信息的（shape context），需要encode到特征中

所以，这篇论文最主要的贡献：

> 为每个facial landmark学习出一组局部二值特征（encode local feature）。为了encode点之间的constraint，将所有局部二值特征组合起来，进行全局回归。

![](http://7xl2fd.com1.z0.glb.clouddn.com/LBF1.png)

分析LBF论文的时候，还结合了[https://github.com/luoyetx/face-alignment-at-3000fps](https://github.com/luoyetx/face-alignment-at-3000fps)。经过从头至尾的Debug，让我们来分析下流程：

### 1. data prepare

用VJ检测器进行face detection，得到bbox。中间需要过滤“异常”检测结果。

```
Rect r = rects[i];
if (x_max - x_min > r.width*1.5) continue;
if (y_max - y_min > r.height*1.5) continue;
if (abs(center_x - (r.x + r.width / 2)) > r.width / 2) continue;
if (abs(center_y - (r.y + r.height / 2)) > r.height / 2) continue;
```

从TxT读取img_path以及landmarks坐标，算出所有landmark横纵坐标最小最大值x_min，x_max，y_min，y_max。因为不能拿整张图片作为train data，所以需要crop face image。一则减少程序内存使用，二则减少像素噪音。VJ检测器检测出的face bbox一般不能包含整个face轮廓，所以需要对bbox扩充。

分别向左/向右扩充半个bbox width，向上/向下扩充半个bbox height，然后crop image。

```
x_min = max(0., x_min - bbox[2] / 2);
x_max = min(img.cols - 1., x_max + bbox[2] / 2);
y_min = max(0., y_min - bbox[3] / 2);
y_max = min(img.rows - 1., y_max + bbox[3] / 2);

Rect roi(x_min, y_min, x_max - x_min, y_max - y_min);
imgs[i] = img(roi).clone();
```

bbox和landmark坐标也更新为：距离扩充后人脸区域最左端横坐标的相对差值。（calculate the landmark position aligned to this face region.）

### 2. data augmentation

最简单的data augmentation即flip。landmark的y坐标不变，x坐标对称变换。

```
gt_shape_flipped(k, 0) = w - gt_shapes[i].at<double>(k, 0);
gt_shape_flipped(k, 1) = gt_shapes[i].at<double>(k, 1);

//bbox
x_b = w - bboxes[i].x - bboxes[i].width;
y_b = bboxes[i].y;
w_b = bboxes[i].width;
h_b = bboxes[i].height;
```

flip之后有些对称点顺序会变乱，还需要交换一些landmark点。

LBF model也是cascaded，所以为每张training image生成多个init shape（随机取训练样本中的shape），也算是data augmentation。

### 3. model component

![](http://7xl2fd.com1.z0.glb.clouddn.com/LBF2.png)

假设总共级联5 stages，每个stage配置random forests用于学习lbf，总共5个random forests。每阶段的random forests为每个landmark配置一组random trees，所以每个阶段的random forests由68组random trees组成。每组random trees由6棵tree组成，每棵tree的depth设为5，tree是二叉树。

同时需要为每个stage学习global regression weights **W**，用作lbf的全局回归。**W**矩阵的size为(2 x 68, 68 x 6 x (2^(5-1))，LBF矩阵的size为(68 x 6 x (2^(5-1)))，相乘可得每个stage的(2 x 68, 1)的delta shape（偏移）。

### 4. train data prepare

train的第一步肯定是计算特征（lbf）和label（delta shape）。计算delta shape前需要对current shape和mean shape做[Similarity Transform](https://github.com/luoyetx/face-alignment-at-3000fps/issues/4)，然后计算groundtruth shape和current shape间的delta shape。

#### 4.1 How to calculate delta shape

**And this part is important**

As mentioned [here](https://github.com/luoyetx/face-alignment-at-3000fps/issues/4), imagine that we have a face for different size(e.g 100x100, 400x400, 600x600), and every face has a shape on it. If delta shape is relative, this three faces of different size will be given the same delta shape, which is what we want. Buf if delta shape is absolute, delta shape can not fit to three different scale unless the feature from these faces are very different, which is not what we want to see.

So, we need to use relative delta shape as our regression target. This operation is a bit like normalization, making regression target in a generally uniform range. One specific solution can be: When loading training data, we get groundtruth shapes aligned to the the face bbox(region), and get random current shape(initial shape) for each training data.

```c++
current_shapes[idx] = bboxes_[i].ReProject(bboxes_[k].Project(gt_shapes_[k]));
```

Then, we need to calculate delta shape between current shape and gt shape. Please note that delta shape is in [-1,1] range because of the Project function. And we use Similarity Transform(align to mean shape) to remove size, scale variation. 

(PS: when calculating mean_shape, wo project all training shape to [-1,1], so mean_shape is also in range [-1,1])

```c++
delta_shapes[i] = bboxes[i].Project(gt_shapes[i]) - bboxes[i].Project(current_shapes[i]);
calcSimilarityTransform(mean_shape, bboxes[i].Project(current_shapes[i]), scale, rotate);
delta_shapes[i] = scale * delta_shapes[i] * rotate.t();
```

And the same idea in test process.

```c++
// generate lbf
Mat lbf = random_forests[k].GenerateLBF(imgs[i], current_shapes[i], bboxes[i], mean_shape);
// update current_shapes
Mat delta_shape = GlobalRegressionPredict(lbf, k);
delta_shape = delta_shape.reshape(0, landmark_n);
current_shapes[i] = bboxes[i].Project(current_shapes[i]);
calcSimilarityTransform(current_shapes[i], mean_shape, scale, rotate);
current_shapes[i] = bboxes[i].ReProject(current_shapes[i] + scale * delta_shape * rotate.t());
```

### 5. generate lbfs

lbfs的学习通过random forests完成。每个stage共68 x 6 = 408棵tree，所有tree进行有重复地分享training examples进行训练。

random tree的训练task主要就是split node。树最大的特性就是递归，所以split node递归进行，split选用的特征时pixel-difference features。每个stage有自己特定的超参数feats_m和radius_m。论文中也有提到，feats_m表示split node是pixel-difference features pool的数量，每次随机提取feats_m = 500个pixel-difference features。radius_m表示在当前landmark附近提取pixel-difference features的范围，一个landmark为中心、半径为radius_m的小圆范围。

得到feats_m = 500个pixel-difference features（记作densities）之后，和正常树的node split一样：计算当前结点的总方差variance_all，对densities排序为densities_sorted，随机枚举threshold_

```
int threshold_ = densities_sorted(j, (int)(N*rng.uniform(0.05, 0.95)));
```

然后依据threshold_分为左右子节点样本集合，计算左右子结点的方差。找能最大减少方差（对delta shape计算方差）的threshold_。二叉树的每个结点记录两个坐标点，两点方便测试时计算pixel-difference features。

```
double variance_ = (calcVariance(left_x) + calcVariance(left_y))*left_x.size() + \
    (calcVariance(right_x) + calcVariance(right_y))*right_x.size();

//select a feat which reduces maximum variance
```

下一步的全局回归需要(68 x 6 x (2^(5-1)))的lbfs。所以对于每个测试样本，从tree根结点开始依据结点的threshold和pixel-difference features的两坐标点决定样本往左结点遍历，还是右结点遍历。落在哪个叶子结点，该叶子结点赋值为1，其余为0，最终形成稀疏的二值lbfs。具体代码方面的实现可以这么实现：对所有叶子结点（68 x 6 x 2^(5-1)）编号。从根结点遍历样本直到当前tree的叶子结点，记录此叶子结点在所有叶子结点中的编号就可以。

![](http://oiqcl4y9s.bkt.clouddn.com/local-binary-features.PNG)

#### 5.1 Tricks about generating LBFs

**And this part is also very important**

When we calculate features, firstly we need to select two random points centered around current shape for each landmark. This operation are in the mean shape coordinate system. After that, we should transfer these two random points back to image coordinate system, aimed to sample pixel and calculate pixel-different features.

```c++
calcSimilarityTransform(bbox.Project(current_shape), mean_shape, scale, rotate);
// then use scale and rotate to transfer coordinate system back to image coordinate system.
```

![](http://oiqcl4y9s.bkt.clouddn.com/lbf.png)

### 6. global regression train

全局线性回归用于学习lbfs和delta shape之间的映射，可以用[LIBLINEAR](http://www.csie.ntu.edu.tw/~cjlin/liblinear/)库。将所有记录的叶子结点编号位置设置为1，其余为0。

### 7. cascaded

之后同样的方式进入级联的下一个stage。

在lpfw（68点）dataset上train、test的效果

![](http://oiqcl4y9s.bkt.clouddn.com/lbf-train.PNG)

![](http://oiqcl4y9s.bkt.clouddn.com/lbf-test.PNG)

![](http://oiqcl4y9s.bkt.clouddn.com/lbf-demo.PNG)