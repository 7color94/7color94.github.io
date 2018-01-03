---
layout: post
title: 解读Constrained Local Model
categories:
- Research
tags:
- face alignment
- paper
---

Constrained Local Model(CLM) is class of methods of locating sets of points. Given a face image, CLM is used to find facial landmarks. In the one hand, Similar to Active Shape Model(ASM) and Active Apperance Model(AAM), CLM use Shape Model to constrain the shape. In the other hand, AAM searches by using the texture residual betwwen model and the target image to predict parameters to obtain the best possible match. And ASM only uses the Shape Model to update the feature locations after computing the best match of each detector, for example, ASM find the best match having minimum Mahalanobis distance. However, CLM uses Patch Model to find the best match for each facial landmarks.

In summary, during training, CLM uses training set to build Shape Model and Local Patch Model. And while testing new image, we use CLM to search in local region around where the corresponding item might appaer. We use Shape Model to constarin the search, and use Local Patch Model to search for individual items, hence the name Constrained Local Models.

![](http://oiqcl4y9s.bkt.clouddn.com/CLM.png)

A more detailed conceptual diagram of CLM implementation given below.

![](http://oiqcl4y9s.bkt.clouddn.com/CLM-Architecture.png)

### 1. Model Building

Before we use CLM to search facial landmarks, we need to build a CML model first. 

**Shape Model** building is the same as ASM and AAM, using PCA for shape vectors of all training samples. **Before doing PCA, we need to use e.g Procrustes analysis to ensure all training shapes represented are in the same co-ordinate frame. The shape of a face is normally considered to be independent of the position, orientation and scale of that face image.** And CLM shape model is concerned with finding shape variation, not the translation, scale and rotation information, we need to find a way to remove those from the feature points coordinates.

**Patch Model** describes how the image around each feature point should look like. Patch Model can be build in several ways. We can build template model by linear Support-Vector-Machine. For each feature point, we train a linear SVM to recognize the local patch model around the feature point, and will later use it in the search process.

Suppose we have 1000 training images, each image contain 68 feature points. For each feature point, we already have 1000 positive examples. We can randomly sample elsewhere in the training image, to generate as many as negative examples as we like for each point, but we choose to sample at positions near the feature point position, because later we want to train SVM to be able discriminate between patch from exactly the given feature point position, and those from nearby position. Then we train a linear SVM with positive/negative examples to tell whether a patch is positive or negative during search process.

But there is a complication: **we have to take into account that these faces can be of different sizes, and orientations. A 16x16 patch on small face image might cover half a face, while in the case when a face image is large, a 16x16 patch can only cover a tiny face. So we need to scale and rotate them into roughly the same size before can do cropping.** 

And, building Patch Model should remove all shape variation. One Solution can be: when loading training data, choose the first shape as reference, then align all training data to the first shape coordinate system and build shape model. At the same time, align all training images to frist one. So the Patch Model has nothing to do with the shape variation.

### 2. Search Process

After building CLM Model, we can use it to find the position of facial landmarks. Below lists steps in the search process.

- 1.Make initial guess of feature point position.
- 2.For each feature point, use SVM to search in the local region of its current postion, to obtain SVM response image.
- 3.Fit each response image with a quadratic function.
- 4.Find best feature point position by optimizing quadratic functions and shape constrains.
- 5.Repeat step 2-4 until converge.

We now explain these steps.

#### 2.1 step 1-2

To do a search, we make a guess of initial position, possibly be initializing to the mean shape, then for each point, we use template matching method(SVM) to search the local region around current postion. The result is, for each feature point, we obtain a response iamge corresponding the local region searched.

We then use these response images to determine the position of each feature point. You might be
tempted to allow each point to jump to the position where highest response is found. But this might lead to a weird shape, violate shape constraint. So the job is to find the
best position for each point from the response image obtained, taking into account the allowed shape
variation, as described in the shape model. There are many ways to do this right.

#### 2.2 step 3-4

Here is the crudest method: first find all positions with highest match score, check to see if it
violates shape constraints, and if so, find a position that has a little lower match score, check the shape
constraints again … until you find a position that does not violate shape constraint. This gives you the position
that does not violate shape constraint, and still have a high match score. This method is crude, and NP hard,
or mission impossible, in plain English.

Now think this: if the response image exhibits some type of feature, then maybe we can exploit such
feature to speed up our search. For example, if the response image is in fact a 2-D Gaussian function, or a
2-D quadratic function, then we can exploit its properties when doing a search(heuristic search?). Of course few, if any, of the
response image is actually such a 2-D function. But if we can fit a function to the image, we can speed up the
search. This will definitely degrade the search result, but we are hoping if we do the fit wisely, the result
might be close to the real result.

So, we fit a 2-D quadratic function to each response image. We add these quadratics
together, and call it the response function. We also write the
shape constraints as a function of feature point positions, call it the shape constraint function. Adding these
two functions together, we obtain a giant function, whose variables are position of each feature points. By
optimizing this function (using off-the-shelf algorithms like quadratic programming), we can find the best
position for each feature point.

#### 2.3 step 5

Now that we have found the position of each feature point, it may seem we’re done with the job.
But there’s another issue: if our SVM search is done on the whole image, maybe we can rest for now, but
since our search is only done in a local region around current position, the result we obtain may only be a
best one locally (local optimum). If we want to find the global best position, our safe bet would be to repeat
step 2-4, until all points reaches a stable position. 

The truth is, unfortunately, you are still not guaranteed to
get a global optimum, due to the nature of face alignment problem. But this is as much as we can do for
now.

### 3. Tricks in search process

There are many tricks in the search process, and this is the most exciting part.

**This part has too many formulations, and blog can not record in details. But this part is the most important, please refer to 《Constrained Local Model for Face Alignment, a Tutorial》By Xiaoguang Yan.**

### 4. Coordinate Transformation

Particularly, I will mention the **coordinate transformation**, using Procrustes analysis method(procrustes function in Matlab). 

1.Since shape model is only concerned with shape variation, not the translation, scale and rotation information. So we will use procrustes method to make all training shapes align to the same shape(eg. select the first training shape as reference, and align the rest shapes to the first one).

2.Coordinate Transformation Tricks in CLM search process: Firstly we use the mean shape as our initial shape, and add some scale, rotate, translation to the mean shape, making initial shape close(not intended to be accurate) to the groundtruth shape of the test face image. Please note that the mean shape is in the shape model coordinate system, but after adding scale&rotate&translation, the initial shape is in the test face image coordinate system, which is important to remember. Then, when searching by the CLM Patch Model, we need crop image centered at current facial point. But cropped image patch should only be related to the pixel information, removing the shape variation informations. So, we need to align the initial shape(origin test image coordinate system) to the mean shape(shape model coordinate system). And calculate coordinate points in the shape model coornidate system, which making up the cropped images. In addition, we should transform these coordinate points in the shape model coornidate system to the real points in the image coordinate system, to get the real pixel information(or image patches) for the SVM classifier. Then as mentioned in the step 3-4, fitting 2-D quadratic function, use giant function to cal the updated points(in the shape model coordinate system), and transform new points back to origin image coordinate system, to imshow in the screen.

### References

- 《Constrained Local Model for Face Alignment, a Tutorial》By Xiaoguang Yan. Version 7
- 《Feature Detection and Tracking with Constrained Local Models》By Tim Cootes.