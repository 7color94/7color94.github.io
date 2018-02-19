---
layout: post
title: 读中位数和顺序统计量
categories:
- Algorithms
---

今天读某公众号推送的一篇文章，题目是：求无序数组中的中位数.最简单的方法莫非就是排序，然后直接print中位数.

《算法导论》第9章 中位数和顺序统计量 正好讲到了这个问题，可以在不排序的情况下求解，并且时间复杂度接近O(n)

#### 概念

找出数组中的最大值、最小值和中位数问题都可以一般化为**选择问题**：从一个由n个互异的元素构成的集合中选择第i个顺序统计量问题

第i个顺序统计量指集合中第i小的元素，所以：

- 最小值是第1个顺序统计量（i=1）
- 最大值是第n个顺序统计量（i=n）
- 当n为奇数时，中位数的i=(n+1)/2;当n为偶数时，中位数的i=n/2和n/2+1

#### 期望为线性时间的选择算法

该算法以快速排序算法为原型，采用分治法，基本思路是：任意选择一个元素作为key，基于key将数组分为两部分.左部分元素均小于等于key，右部分元素均大于key.如果key的下标idx正好等于(n+1)/2，那么key即为中位数.否则若idx<(n+1)/2，那么递归去处理右部分，反之处理左部分

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from random import randint

def randomized_partition(A, low, high):
    rand_n = randint(low, high)
    
    key = A[rand_n]
    A[rand_n] = A[high]
    A[high] = key

    i = low - 1
    j = low
    tmp = 0
    while j < high:
        if A[j] < key:
            i += 1
            tmp = A[i]
            A[i] = A[j]
            A[j] = tmp
        j += 1
    A[high] = A[i + 1]
    A[i + 1] = key
    return i + 1

def randomized_select(A, low, high, i):
    if low == high:
        return A[low]
    q = randomized_partition(A, low, high)
    k = q-low+1
    if i == k:
        return A[q]
    elif i < k:
        return randomized_select(A, low, q-1, i)
    else:
        return randomized_select(A, q+1, high, i-k)

if __name__ == '__main__':
    A = [4,2,3,1,7]
    # i取4，表示求解A中的中位数
    print randomized_select(A, 0, len(A)-1, 4)
```

不同于快速排序会递归处理划分的两边，而randomized_select只处理换分的一边.经证明randomized_select的期望运行时间为O(n)

#### 最坏情况为线性时间的选择算法

和randomized_select一样，select算法也是通过递归划分来寻找所需元素.但是该算法能保证得到对数组的一个好的划分.根据《算法导论》中描述，select算法的步骤为：

- 1.将n个元素的输入数组划分为[n/5]组，每组5个元素，且至多只有一组由剩下的n mod 5个元素组成
- 2.寻找[n/5]组中每一组的中位数：首先对每组元素进行插入排序，然后确定每一组的有序元素的中位数
- 3.对第2步中找出的[n/5]个中位数，递归调用select以找出其中位数x（如果有偶数个中位数，为了方便，取较小那个中位数）
- 4.利用修改过的partition，按中位数的中位数x对输入数组进行划分，确定x在数组中的位置k
- 5.如果i==k，返回x.否则，i<k，处理低区.反之在高区寻找i-k小的元素

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

def partition(A, low, high, key):
    idx = 0
    for i in xrange(low, high):
        if A[i] == key:
            idx = i
            break
    swap(A, idx, high)

    i = low - 1
    j = low
    while j < high:
        if A[j] < key:
            i += 1
            swap(A, i, j)
        j += 1
    swap(A, i+1, high)
    return i + 1

def insert_sort(A, low, high):
    i = low + 1
    while i <= high:
        key = A[i]
        k = i - 1
        while k >= low and A[k] > key:
            A[k + 1] = A[k]
            k -= 1
        A[k + 1] = key
        i += 1

def swap(A, a, b):
    tmp = A[a]
    A[a] = A[b]
    A[b] = tmp

def select(A, low, high, i):
    if high-low < 5:
        insert_sort(A, low, high)
        return A[low + i - 1]
    group = (high - low + 5) / 5
    for j in xrange(group):
        left = low + j*5
        right = (low + j*5 + 4) if (low + j*5 + 4) < high else high
        mid = (left + right)/2
        insert_sort(A, left, right)
        swap(A, low+j, mid)
    key = select(A, low, low+group-1, (group+1)/2)
    key_idx = partition(A, low, high, key)
    k = key_idx - low + 1
    if k == i:
        return A[key_idx]
    elif k > i:
        return select(A, low, key_idx-1, i)
    else:
        return select(A, key_idx+1, high, i-k)

if __name__ == '__main__':
    A = [32,23,12,67,45,78,10,39,9,58]
    for i in xrange(1, 11):
        print select(A, 0, len(A)-1, i)
```

#### 脑洞大开：利用最小堆

（依据待字闺中微信公众号推送的文章）

首先，将数组的前(n+1)/2个元素建立一个最小堆.然后对于下一个元素，和堆顶元素比较，如果小于等于就丢弃之.接着看下一个元素，如果大于，则用该元素取代该顶，再调整堆.重复直至数组为空时，堆顶元素即为中位数

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import heapq

def heap_select(A, lens):
    h = []
    for j in xrange((lens+1)/2):
        heapq.heappush(h, A[j])
    top = 0
    for j in xrange((lens+1)/2, lens):
        top = heapq.heappop(h)
        if A[j] <= top:
            heapq.heappush(h, top)
            continue
        else:
            heapq.heappush(h, A[j])
    return heapq.heappop(h)

if __name__ == '__main__':
    A = [4,5,1,3,2]
    print heap_select(A, len(A))
```

---

参考：

- 《算法导论》Chapter 9
-  微信公众号：待字闺中