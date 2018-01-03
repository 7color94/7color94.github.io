---
layout: post
title: 读后缀数组
categories:
- Algorithms
tags:
- suffix array
- paper
---

通过阅读NOI[后缀数组](https://en.wikipedia.org/wiki/Suffix_array)的[论文](http://www.newsjz.com/wxqgr/xxas/UploadFiles_3822/201411/2014111809241352.pdf)，对其中相关内容总结

首先明确后缀：指从原串r某个位置i开始到串末尾的整个子串，记作Suffix(i) = r[i..len(r)]。

后缀数组当然最核心的就是后缀数组，和名次数组

![](http://oiqcl4y9s.bkt.clouddn.com/suffix-array.PNG)

- 后缀数组SA。SA具有一定的顺序，保证Suffix(SA[i]) < Suffix(SA[i+1]) (1 <= i < n)。就是将S的n个后缀按字符串序从小到大排列后，将排好序的后缀开头位置顺次放入后缀数组SA。
- 名次数组Rank[i]保存的是后缀Suffix(i)在所有后缀中从小到大排列的“名次”。

粗暴点说，后缀数组“排第几的是谁？”，名次数组“你排第几？”

知道了名次数组，就可以通过遍历原串，算出后缀数组。名次数组的生成算法有倍增算法和DC3算法。我主要实践了相对简单的倍增算法

![](http://oiqcl4y9s.bkt.clouddn.com/%E5%80%8D%E5%A2%9E%E6%B3%95%E5%90%8E%E7%BC%80%E6%95%B0%E7%BB%84.PNG)

具体来说，按照图中算法流程，利用一个结构体Rank_记录x, y, rank，以及id（id为后缀子串在原串中开始的位置i）。每次排序时，先根据x增序排，同等情况下，按照y增序排列，遍历排序完成的Rank_数组，记录相应rank。然后需要依据id排列，将Ranks_数组恢复成id顺序。

我没能理清论文给出的倍增算法代码，所以自己根据上述理解完成倍增算法代码。

得到rank数组后，遍历原串，即可得到后缀数组。后缀数组sa和名次数组rank互为逆运算。

前面sa和rank的计算比较简单，最终还有块比较难啃的硬骨头：heights数组。

height数组：定义height[i]为排名i-1和i的两后缀的最长公共前缀长度，即suffix(sa[i-1])和suffix(sa[i])最长公共前缀的长度。

```
void calHeights(int n) {
	for (int i = 1, j = 0; i <= n; i++) {
		if (j) {
			j--;
		}
		while (arrays[i + j] == arrays[sa[ranks[i].rank - 1] + j]) {
			j++;
		}
		heights[ranks[i].rank] = j;
	}
}
```

计算heights数组需要用到一点非常重要的性质：

![](http://oiqcl4y9s.bkt.clouddn.com/height%E6%95%B0%E7%BB%84%E6%80%A7%E8%B4%A8.PNG)

> height[rank[i]] >= height[rank[i-1]]-1 （1）

这就是程序每次循环j-1的原因。通过debug，我们会发现绝大多数情况下，公式（1）取等号。出现大于号的情况是边界条件：若rank[i]=1，height[rank[i]=1] = 0，所以height[rank[i+1]]一定大于height[rank[i]=1]-1=0-1。而且程序j++的循环，正常情况是进不来的，只有刚初始化和边界条件时，才会进行公共前缀的匹配（j++），通常情况height[rank[i]] = height[rank[i-1]]-1。

![](http://oiqcl4y9s.bkt.clouddn.com/height%E6%8E%A8%E5%AF%BC.PNG)

#### 1.重复子串问题

利用后缀数组和heights数组，可以很方便地解决重复子串问题。首先明确重复子串：字符串R在字符串L中至少出现两次，则称R是L的重复子串。

只要出现不少于2次就可称为重复子串，而且问题一般让求解最长的重复子串，所以问题可以化简为原字符串中出现两次的子串的最长长度。因为当出现次数越多（大于2），长度会小于只出现两次的最长重复子串的长度。

##### 1.1 可重叠最长重复子串

求height数组中最大值即可。因为求最长重复子串，等价于求两个后缀的最长公共前缀的最大值。时间复杂度O(n)。

##### 1.2 不可重叠最长重复子串

这题稍微复杂一点。通过阅读论文才知道算法的流程，对于算法如何来的，渣渣也不知道..表示这类算法真的是神人创造出来的..

算法对答案进行二分查找。答案指的是不可重叠最长重复子串的最长长度k，其范围[1, n/2]。n/2的原因是不可重叠，所以不可重叠最长重复子串的长度 <= n/2。

对于某猜测的最长长度k，我们需要验证k是否满足要求。算法的核心是利用height数组对排序后的后缀分成若干组，其中每组的后缀之间的height值都不小于k。如k=2时的分组情况。此时，有希望成为最长公共前缀不小于k的两个后缀一定处于同一组。所以对每组后缀，只需判断每个后缀的sa值的最大值和最小值只差是否不小于k（保证不可重叠性质）。若有一组满足，则猜测的答案k符合要求，需要进一步验证比k大一些的长度。若没有一组满足要求，需将k二分缩小，时间复杂度O(nlogn)。

![](http://oiqcl4y9s.bkt.clouddn.com/%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%8F%A0%E6%9C%80%E9%95%BF%E9%87%8D%E5%A4%8D%E5%AD%90%E4%B8%B2.PNG)

```
//不可重叠最长重复子串 的长度
//给定一个字符串，求最长重复子串，这两个子串不能重叠
int solve(int n) {
	int left = 1, right = n / 2;
	while (left < right) {
		int k = left + (right - left + 1) / 2;
		if (nooverlap_long_string(k, n)) {
			left = k;
		} else {
			right = k - 1;
		}
	}
	return left;
}

bool nooverlap_long_string(int k, int n) {
	int min_ = 20005, max_ = 0;
	for (int i = 1; i < n; i++) {
		if (heights[i + 1] >= k) {
			min_ = min(min(sa[i], sa[i+1]), min_);
			max_ = max(max(sa[i], sa[i + 1]), max_);
			if (max_ - min_ >= k) {
				return true;
			}
		} else {
			min_ = 20005;
			max_ = 0;
		}
	}
	return false;
}
```

##### 1.3 可重叠的k次最长重复子串

给定一个字符串，求至少出现k次的最长重复子串，这k个子串可以重叠。

和1.2做法类似。二分寻找答案，还是将后缀数组分成若干组，不同之处是判断有没有一个组的后缀个数不小于k，时间复杂度O(nlogn)。这道题对应[hihocoder](http://hihocoder.com/problemset/problem/1403)。

```
//可重叠的k次最长重复子串 的长度
//给定一个字符串，求至少出现k次的最长重复子串，这k个子串可以重叠。
bool overlap_k_long_string(int k, int l, int n) {
	int count = 1;
	for (int i = 1; i < n; i++) {
		if (heights[i + 1] >= l) {
			count += 1;
			if (count >= k) {
				return true;
			}
		} else {
			count = 1;
		}
	}
	return false;
}

int solve(int k, int n) {
	int left = 1, right = n;
	while (left < right) {
		int l = left + (right - left + 1) / 2;
		if (overlap_k_long_string(k, l, n)) {
			left = l;
		} else {
			right = l - 1;
		}
	}
	return left;
}
```

#### References

感谢GLS同学与我讨论

- <a href="http://www.newsjz.com/wxqgr/xxas/UploadFiles_3822/201411/2014111809241352.pdf" target="_blank">《后缀数组--处理字符串的有力工具》</a>
- hihocoder