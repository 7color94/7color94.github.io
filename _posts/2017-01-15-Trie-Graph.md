---
layout: post
title: Trie图
categories:
- Algorithms
tags:
- trie
---

嘛，[Trie图](http://hihocoder.com/problemset/problem/1036) = AC自动机，旨在解决这类的Task：

> 给定文本串text（长度为M），和词典集合（词语数量N，每个词语长度L），判断文本串text中是否存在词典中的词语。

task的解法有很多：KMP和Trie树。

#### KMP

用KMP算法求解：枚举词典中每个单词，然后利用KMP算法判断text中是否存在当前枚举的单词。时间复杂度O(NML)。

#### Trie树

用Trie树求解：Trie树可以快速计算前缀匹配问题。然而task却是可以在任意位置匹配。所以可以枚举起始点，然后将task 转化为前缀匹配问题。时间复杂度O(ML)。

然而在Trie树基础上，添加一些边，就可以形成**Trie图**，能有效减少Trie树任意位置匹配的冗余计算。

![](http://oiqcl4y9s.bkt.clouddn.com/trie%E5%9B%BE.PNG)

如图，先枚举第一个字符作为起始点，最多匹配到结点A（abc），此时不能再匹配。之后枚举第二个字符作为起始点，匹配到B结点。也就是，我们从根结点走到了A结点，然后回到根结点，再走到B结点。

即：我们从str当前结点开始，匹配了l个长度走到了A结点。如果我们把A结点对应的字符串（abc）去掉第一个字符，形成的新字符串（bc），肯定是和从str下一个起始点开始，长度为l-1的子串是一样的。如果我们能预先计算找到了B'结点，我们就不用像之前那样从根结点走到A结点，然后回到根结点，再走到B结点。而是可以直接从根结点走到A结点，然后直接跳转到B'结点，最后匹配到B结点。

所以问题规约成：

> 给定Trie树，找到其中每一个结点A对应的后缀结点A'。结点A在Trie树中对应的路径去掉第一个字符之后在Trie中对应的结点为A'。

求后缀结点也不难：

> 结点B的后缀结点等于B结点的父结点A的后缀结点 加上 路径AB 得到对应的的后缀结点C

> 若结点C不存在，那么此时找后缀结点的后缀结点B' 加上 路径AB 得到对应的后缀结点C'。因为没有更好的选择，所以只能找“弱”一些的后缀节点

```
void buildAC(TrieNode *root) {
	queue<TrieNode*> qt;
	qt.push(root);
	while (!qt.empty()) {
		TrieNode *current = qt.front();
		qt.pop();
		for (int i = 0; i < CHAR_RANGE; i++) {
			if (current->child[i] != NULL) {
				if (current == root) {
					current->child[i]->suffix = root;
					qt.push(current->child[i]);
					continue;
				}
				TrieNode *currentSuffix = current->suffix;
				while (currentSuffix != NULL) {
					if (currentSuffix->child[i] != NULL) {
						current->child[i]->suffix = currentSuffix->child[i];
						break;
					}
					currentSuffix = currentSuffix->suffix;
				}
				if (currentSuffix == NULL) {
					current->child[i]->suffix = root;
				}
				qt.push(current->child[i]);
			}
		}
	}
}
```

在KMP算法中，我们用指针i和j分别表示text[i-j+1..i]和pattern[1..j]相等。i不断增加，j随着i的增加相应地变化，j满足以text[i]结尾的长度为j的字符串正好匹配模式串pattern[1..j]。若text[i+1]和pattern[j+1]不相等，那么KMP通过next函数调整j（减小j值）使得text[i-j+1..i]和pattern[1..j]保持匹配且新的pattern[j+1]和text[i+1]匹配。

AC自动机的suffix指针，也称作失败指针，功能类似。模式上在Trie中不能继续匹配时，应当前往当前结点的失败指针所指向的结点继续进行匹配。