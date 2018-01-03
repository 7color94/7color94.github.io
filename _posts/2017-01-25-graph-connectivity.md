---
layout: post
title: 用Tarjan算法解决图连通性问题
categories:
- Algorithms
tags:
- graph
---

### 1. 割点与割边

给定一张连通图，求此图的割点集合、割边集合。

- 割点：在连通图中，删除了连通图的某个点以及与这个点相连的边后，图不再连通。
- 割边：在连通图中，删除了连通图的某条边之后，图不再连通。

e.g 点3和点4为割点，边3-4为割边（也称作桥）。

![](http://oiqcl4y9s.bkt.clouddn.com/tarjan_1.png)

其中：

- 树边/父子边：图中蓝色线段所示，表示DFS过程中访问未访问节点时经过的边。
- 回边：图中黄色线段所示，表示DFS过程中访问已访问节点时所经过的边，也称为返祖边、后向边。

显然，暴力地枚举点、边，然后DFS判断图的连通性，复杂度一定不允许。

通过观察我们发现在两种情况下，节点可以成为割点：

- 若节点u为根节点，此时如果根节点有两颗以上的子树，那么根节点必为割点。
- 若节点u为非根节点，如果u的某棵子树（所以，u当然也是非叶子节点，即非根非叶子节点）所有节点均没有指向u祖先节点的回边，说明删除u后，该棵子树所有节点和根节点不再连通，此时u为割点。

Tarjan算法则通过三个数组记录额外信息来解决此类连通性问题。

- 第一种节点u为根节点的情况，只要判断u的孩子节点个数是否大于1。所以通过children[u]数组记录每个节点的dfs过程中的孩子节点个数即可。
- 第二种节点u为非根非叶子节点，判断回边是个难题。如果我们用dfn[u]记录节点u在DFS过程中被访问的次序，low[u]记录节点u或其子树能通过回边追溯到的最早的祖先节点（即DFS次序号最小）。那么此时如果(u,v)为树边，且low[v] >= dfs[u]，说明v及其子树通过回边追溯的最早祖先节点是大于等于u的，也就是说u之前的祖先们，都追溯不到，那么切除节点u，v及其子树就和u之前的祖先们“分开”了，所以u为割点。

对于割边：

- 当(u,v)为树边，且low[v] > dfn[u]，表示v节点只能通过该边(u,v)与u连通，那么(u,v)即为割边。

所以，从节点1开始DFS过程，更新三个数组并根据上面规则判断就好了。

children数组的更新简单的，dfn数组也很简单。low数组更新规则如下，对着例子里的图片理解，很容易理解：

![](http://oiqcl4y9s.bkt.clouddn.com/tarjan_2.png)

```
void tarjan(unsigned u) {
    low[u] = dfn[u] = ++ord;
    bool flag = false;
    for (size_t i = 0; i < edges[u].size(); i++) {
        unsigned v = edges[u][i];
        // 树边/父子边
        if (!dfn[v]) {
            parent[v] = u;
            children[u]++;
            tarjan(v);
            low[u] = min(low[u], low[v]);
            if (low[v] >= dfn[u]) {
                flag = true;
            }
            if (low[v] > dfn[u]) {
                cutEdges.push_back(make_pair(min(u,v), max(u,v)));
            }
        } else {
            // 回边，且因为是无向图所以需要判断v是否为parent[u]
            if (v != parent[u]) {
                low[u] = min(low[u], dfn[v]);
            }
        }
    }
    // 前面情况是根节点（子树含有两个以上）, 后面是非根非叶子节点
    if ((parent[u] == 0 && children[u] > 1) || (flag && parent[u])) {
        cutPoints.push_back(u);
    }
}
```

借助割点和割边的基础，我们可以改造基础的Tarjan算法求图的边双连通分量、点双连通分量和强连通分量。前两种属于无向图，后一种属于有向图。

### 2. 边的双连通分量

边的双连通分量：

> 对于一个无向图的子图，当删除其中任意一条边后，不改变图内点的连通性，这样的子图叫做边的双连通子图。
> 而当子图的边数达到最大时，叫做边的双连通分量。

![](http://oiqcl4y9s.bkt.clouddn.com/tarjan_3.png)

图中{1,2,3}，{4,5,6}，{7}各为一个组，就满足要求。

计算分组的直观做法就是算出图中的所有割边，把割边去除后，剩下的每个区域就是要求的边的双连通分量。这样的做法不免也有些繁琐，先求割边，然后删除所有割边，再DFS求边的双连通分量。

其实只要在Tarjan算法DFS过程中加入一个栈，就能巧妙地解决。

每次，在DFS时将当前访问的节点加入栈，之后DFS过程中我们需要额外关注low[u] == dfn[u]的节点u。

> 若节点u满足low[u] == dfn[u]。对于边(parent[u], u)来说，一定有dfn[u] > dfn[parent[u]]，可以推导出low[u] > dfn[parent[u]]，所以(parent[u], u)一定是个割边咯。

在u之前入栈的节点和u被割边(parent[u], u)分开，所以u和u之后入栈的节点一定是同一组，全部从弹出来当成一组就可以了。所以求解边双连通分量只需要DFS求割边，利用栈保存相应分量的节点即可。

### 3. 点的双连通分量

点的双连通分量：

> 对于一个无向图的子图，当删除其中任意一个点后，不改变图内点的连通性，这样的子图叫做点的双连通子图。而当子图的边数达到最大时，叫做点的双连通分量。

![](http://oiqcl4y9s.bkt.clouddn.com/tarjan_6.png)

图中共有三组点双连通分量：{(1,2),(2,3),(3,1)}、{(4,5),(5,6),(4,6)}、{(3,4)}，两组边双连通分量：{1,2,3}、{4,5,6}。需要注意下二者的区别，点的双连通分量保存边的分组，边的双联通分量保存点的分组。

![](http://oiqcl4y9s.bkt.clouddn.com/tarjan_5.png)

再比如，图中存在两组点双连通分量，分别是{(1,2),(2,3),(3,1)}和{(3,4),(4,5),(3,5)}。如果是边双连通分量的话，就是{1,2,3,4,5}这一组。

边双连通分量算法寻找割边，点双连通分量算法则寻找割点：每存在一个割点，将区域一分为二，点的双连通分量就增加一个，点的双连通分量个数就等于割点数量加1。

在求边的双连通分量时，我们DFS求割边并用栈保存相应分量的节点。然而，在求点的双连通分量时，我们dfs求割点时用栈保存相应分量的边就可。遇到割点，从栈中弹出所有在当前边之后入栈的边就可以了。

```
void tarjan(int u) {
    int children = 0;
    low[u] = dfn[u] = ++ord;
    for (int i = to[u]; i != 0; i = next_edge[i]) {
        if (visitedEdge[edge[i].id]) {
            continue;
        }
        stacks[top++] = edge[i];
        visitedEdge[edge[i].id] = true;
        if (!dfn[edge[i].v]) {
            ++children;
            parent[edge[i].v] = u;
            tarjan(edge[i].v);
            low[u] = min(low[u], low[edge[i].v]);
            if (children > 1 && parent[u] == 0) {
                popStack((edge[i]));
            }
            if (parent[u] != 0 && dfn[u] <= low[edge[i].v]) {
                popStack(edge[i]);
            }
        } else {
            low[u] = min(low[u], dfn[edge[i].v]);
        }
    }
}
```

### 4. 强连通分量

强连通分量：

> 对于**有向图**上的2个点a,b，若存在一条从a到b的路径，也存在一条从b到a的路径，那么称a,b是强连通的。对于有向图上的一个子图，若子图内任意点对(a,b)都满足强连通，则称该子图为强连通子图。非强连通图有向图的极大强连通子图，称为强连通分量。

![](http://oiqcl4y9s.bkt.clouddn.com/tarjan_4.png)

3和6形成的子图组成一个强连通分量。

Tarjan算法利用dfn和low数组以及一个栈就可以求出有向图的所有强连通分量了。和求边的双连通分量算法一样，每次DFS访问时将当前节点加入栈。但，有向图中Low(u)表示u或u的子树能够追溯到的最早的栈中节点的次序号，由定义可得更新规则：

```
for (int i = 0; i < edges[u].size(); i++) {
    int v = edges[u][i];
    if (!dfn[v]) {
        tarjan(v);
        low[u] = min(low[u], low[v]);
    } else if (visited[v]) {
        // visited[v]记录v是否还在栈中，未出栈
        // 如果v不在栈内，说明v属于另一个连通分量了
        low[u] = min(low[u], dfn[v]);
    }
}
```

得到dfn数组和low数组的值，我们仍然去关注这类特殊节点：low[u] == dfn[u]，说明节点u是强连通分量的根，将u和u之后入栈的节点全部弹出作为一组强连通分量。

### References

- [Byvoid: 有向图强连通分量的Tarjan算法](https://www.byvoid.com/zhs/blog/scc-tarjan)
- [Zhihu](https://www.zhihu.com/question/40746887)