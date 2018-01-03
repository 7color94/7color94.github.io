---
layout: post
title: Cycle detection
categories:
- Algorithms
---

给定Task：

> 在一组由迭代函数f生成的值序列中，找出此序列的环。

迭代函数f的定义：函数f指从有限集合S到S本身的一种函数映射。给定初始值x_0，由迭代函数f组成的值序列可以为：

> x_0, x_1 = f(x_0), x_2 = f(x_1), ..., x_i = f(x_i-1), ...

因为S为有限集合，一定存在一对(i, j)，i小于j，是的x_i == x_j。那么，x_i ~ x_j-1 一定形成一道环。e.g 

> 2, 0, 6, 3, 1, 6, 3, 1, 6, 3, 1, ....

![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d7/Functional_graph.svg/660px-Functional_graph.svg.png)

[Wikipedia：]((https://en.wikipedia.org/wiki/Cycle_detection))Tortoise and hare Algorithm（龟兔算法）能有效检测出迭代函数值序列（- - 其实，问题也可以规约成检测一段链表）中是否存在环。

![](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5f/Tortoise_and_hare_algorithm.svg/560px-Tortoise_and_hare_algorithm.svg.png)

顾名思义：龟兔赛跑，兔跑得比乌龟快很多。如果两者同时出发，赛道（链表）有环存在，兔子一定存在一个时刻，能追上乌龟（那一时刻，两者位置一致）。此时，兔子一定比乌龟多跑了好几圈环，多跑的路程也是环程度的倍数。如果赛道不存在环，那么兔子一定能到达链表终点，此时可以判断出链表中不存在环。

然而这时，我们的算法只能判断是否有环。环在哪里呢？

我们先来指定下乌龟和兔子的速度。假设乌龟每次前进一步，我们可以让兔子每次前进两步（两步或者多步效果都是等价的，主要保证兔子能追到乌龟就好）。假设链表起点到环起点的距离为m，环的长度为n，龟兔第一次相遇地点距离环起点为k。当龟兔相遇时，乌龟移动的总距离为：

> l = m + ii * n + k，ii表示乌龟走过的圈数 (1)

此时，由于兔子每次移动速度是乌龟的两倍，兔子移动的总距离为：

> 2l = m + jj * n + k，jj表示兔子走过的圈数 (2)

(2) - (1)得：

> l = (jj - ii) * n

也就是说，乌龟走的路程是环长度的倍数。那么此时让兔子再回到链表起点，乌龟保持原地位置。这时两者移动速度一致，每次移动一步长，等到两者相遇时候。当兔子前进了m步，到达环起点，乌龟距离链表起点为l + m，因为l是环长度的倍数，可以想象成乌龟从链表起点出发走了m步到达环起点，然后绕着环走了几圈（l步）。两者相遇点必然为环起点。

代码实现的话，把乌龟和兔子想象成两个指针，慢指针每次next，快指针每次next->next。

```
def floyd(f, x0):
    tortoise = f(x0) # f(x0) is the element/node next to x0.
    hare = f(f(x0))
    while tortoise != hare:
        tortoise = f(tortoise)
        hare = f(f(hare))
  
    # Find the position μ of first repetition.    
    mu = 0
    tortoise = x0
    while tortoise != hare:
        tortoise = f(tortoise)
        hare = f(hare)   # Hare and tortoise move at same speed
        mu += 1
 
    # Find the cycle length
    lam = 1
    hare = f(tortoise)
    while tortoise != hare:
        hare = f(hare)
        lam += 1
 
    return lam, mu
```

#### References

- [Wikipedia：Cycle_detection](https://en.wikipedia.org/wiki/Cycle_detection)：Tortoise and hare Algorithm.