---
layout: post
title: Network Flow Algorithm and its Applications
categories:
- Algorithms
tags:
- network
---

记录网络流算法相关知识点。

### 1. 最大流算法

最大流算法的目标是：在不违反任何容量限制以及遵循流量守恒的情况下，计算从源点到汇点的最大流。

基本的最大流算法是 Ford-Fulkerson 算法。通过在原先的流网络上增加“反悔边”，构成残存网络。在残存网络中寻找从源结点到汇点的一条增广路径，并沿着增广路径重复增加路径上的流量，同时更新“反悔边”（某一条边使用了多少流量，则其反方向设置多少反悔流量）。直到找不到新的增广路，此时得到的流就是该网络的最大流。

Ford-Fulkerson 有时收敛不够快，算法导论给出相应的例子为：

![](http://oiqcl4y9s.bkt.clouddn.com/%E6%9C%80%E5%A4%A7%E6%B5%81%E7%AE%97%E6%B3%95%E6%94%B6%E6%95%9B%E6%85%A2.PNG)

我实践时发现：Ford-Fulkerson 在寻找增广路径时，若按照节点从小到大顺序DFS寻找，不会出现上图“收敛慢”的情况，大概算法导论给出的是最极端的例子。

改进：Edmonds-Karp 算法将每条边的权重视为单位距离，利用SPFA或Bellman-Ford寻找找源点到汇点的最短路径作为增广路径，从而提高Ford-Fulkerson算法的效率。而且，在边权重均为单位距离的情况下，BFS（比DFS）能最快找出最短路径。

Coding：最大流算法编程练习[hiho 1369](http://hihocoder.com/problemset/problem/1369)

### 2. 最大流最小割定理

最小割：流网络的最小切割是整个网络中容量最小的切割。从网络中切除某些边能将网络一分为二，并且这些边容量之和最小。

最大流最小割定理告诉我们：一个流是最大流当且仅当残存网络中不包含任何增广路经。而且，一个网络中最大流的值不会超过该网络最小切割的容量。当网络平稳时，此时流量等于最大流，也等于最小割。

为了算出最小割的点集合，通过最大流算法使网络达到平稳，将网络中和源点有路径相连的点加入集合，得出最小割的点集合。

Coding：最小割编程练习[hiho1378](http://hihocoder.com/problemset/problem/1378)详细地讲述了最大流最小割定理

### 3. 网络流的应用

在相当多的一些网络结构应用中，我们可以通过添加虚拟源点和汇点，以及设置符合题意的边以及边容量，利用最大流算法解决。

- 3.1 最大二分匹配

Coding：[hiho1122](http://hihocoder.com/problemset/problem/1122)。最大二分匹配除了可以利用最大流算法解决，还有知名的匈牙利算法。

- 3.2 二分图多重匹配

Coding：[hiho1393](http://hihocoder.com/problemset/problem/1393)

- 3.3 最小路径覆盖

最小路径覆盖，即给定一个有向无环图，用最少的路径数量去保证所有点都被覆盖住。

Coding：[hiho1394](http://hihocoder.com/problemset/problem/1394)

- 3.4 最大权闭合子图

有向图中，每个节点都有相应的权值，从其中挑选出权值最大的闭合子图。

闭合子图指的是从有向图中选择一些点组成一个点集V，对于V中任意一个点，其后续节点都仍然在V中。

Coding：[hiho1398](http://hihocoder.com/problemset/problem/1398)

---

References

- 《算法导论》网络流
- Stanford [ppt tutorial](https://web.stanford.edu/class/cs97si/08-network-flow-problems.pdf)