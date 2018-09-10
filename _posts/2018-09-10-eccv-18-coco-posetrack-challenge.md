---
layout: post
title: ECCV-18 COCO and PoseTrack Challenge总结
categories:
- Research
tags:
- human pose estimation
---

### ECCV-18 COCO Keypoint Challenge

[coco keypoints leaderboard](http://cocodataset.org/#keypoints-leaderboard)，浏览了前4-5名的方法：

1.需要使用额外数据（+0.7 ~ 1.0 AP）：AI Challenge。我们使用方法不对，用AI Challenge去pre-train再在COCO上finetune，指标没有提升，也不知道什么原因。

2.需要集成更多的backbone：Resnet-101，Resnet-152，Resnext-152，Se-ResNeXt101，ResNet101-dilation等等。其他队基本都集成了4-5个，我们只集成了Resnet-101和152这两个。原因是我们太过于相信Large Batch，训练时使用了Sync-BN导致训练很慢，然后像SENet-154、Resnext-101都得很久才能训完，然后就放弃了。还有就是深的backbone没train好，还不知道什么原因。还有就是因为有49w的bounding box，当时集成2个backbone都跑了半天，集成>=4的backbone的时候肯定太耗时间了，主要就是拼机器数量了。

3.Inference Trick：我们在inference的时候用了Flip，Rotation，还有[The Sea Monsters]提出的Multi-Scale inference，这个影响应该不会太大。

4.需要集成更多的human detector：虽然没有像MegDet性能那么好的detector，我们后来想了下，应该可以通过集成多个human detector提升性能。

所以这次拿了并列第5，经验不足，炼丹水平不足

### ECCV-18 PoseTrack Challenge

[PoseTrack ECCV 2018 Challenge](https://posetrack.net/workshops/eccv2018/posetrack_eccv_2018_results.html)，AP是第5，MOTA是第3

这个比赛太依赖调参，要过滤很多False Positive，指标才能上去。

而且主办方很多问题，训练数据迟迟不放，最后评测只按照最后一次提交算，很多人都不清楚。