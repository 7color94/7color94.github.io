---
layout: post
title: 《算法》第4版相关笔记
categories:
- Algorithms
---

实验室今天停电，翻阅起手边的[《算法》4th](https://book.douban.com/subject/19952400/)，发现很多知识点遗忘了，决定重新温习一次，并把相关代码片段敲下来，下次就可以直接读这篇文章了。

### 第1章 基础

- 二分查找

```c++
int binary_search(vector<int> arr, int key) {
    int lo = 0, hi = arr.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] > key) hi = mid - 1;
        else if (arr[mid] < key) lo = mid + 1;
        else return mid;
    }
    return -1;
}
```

lower_bound：寻找有序数组中第一个大于等于target的元素索引

```c++
int bs_lower_bound(vector<int>& nums, int target) {
    int lo = 0, high = nums.size() - 1, mid;
    while (lo <= high) {
        mid = lo + (high - lo) / 2;
        if (nums[mid] < target) lo = mid + 1;
        else high = mid - 1;
    }
    return lo;
}
```

upper_bound：寻找有序数组中第一个大于target的元素索引

```c++
int bs_upper_bound(vector<int>& nums, int target) {
    int lo = 0, high = nums.size() - 1, mid;
    while (lo <= high) {
        mid = lo + (high - lo) / 2;
        if (nums[mid] <= target) lo = mid + 1;
        else high = mid - 1;
    }
    return lo;
}
```

- 并查集

并查集并不难，union操作用于合并两个连通分量，find操作用来返回当前节点所在连通分析的标识id。书中给出了加权的并查集算法，来保证合并两个连通分量之后的树尽量平衡些。

```c++
int pre[1000]; // 保存节点p的父亲节点编号
int sz[1000];  // 保存节点p的子孙节点数目

int find(int p) {
    while (p != pre[p]) p = pre[p];
    return p;
}

void union(int p, int q) {
    int i = find(p);
    int j = find(q);
    if (i == j) return;
    // 将节点数目少的树连接到数目多的树
    if (sz[i] < sz[j]) { pre[i] = j; sz[j] += sz[i]; }
    else { pre[j] = i; sz[i] += sz[j]; }
}
```

当然，也可以在find时路径压缩，不使用sz[ ]数组，达到同样的效果

```c++
int find(int p) {
    while (pre[p] != p) {
        pre[p] = pre[pre[p]];
        p = pre[p];
    }
    return p;
}

void union(int p, int q) {
    int i = find(p);
    int j = find(q);
    if (i == j) return;
    else pre[i] = j;
}
```

### 第2章 排序

- 归并排序

归并排序核心的操作单元是：归并，即将两个有序的数组归并成一个更大的有序数组。那么，归并排序的思想是递归地将数组分成两半分别排序，然后将排好序的两半子数组归并起来。

```c++
int arr[1000];
int aux[1000]; // 归并所需的辅助数组

// 将a[lo..mid] 和 a[mid+1..hi] 归并
void merge(int lo, int mid, int hi) {
    int i = lo, j = mid + 1;
    for (int k = lo; k < hi; k++) aux[k] = a[k];
    while (int k = lo; k < hi; k++) {
        if (i > mid) a[k++] = aux[j++];
        else if (j > hi) a[k++] = aux[i++];
        else if (aux[i] < aux[j]) a[k++] = aux[i++];
        else a[k++] = aux[j++];
    }
}

void merge_sort(int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    merge_sort(lo, mid);
    merge_sort(mid + 1, hi);
    merge(lo, mid, hi);
}

void main() {
    merge_sort(0, a.length - 1);
}
```

上面介绍的是自顶向下的归并排序方法，将大数组的排序问题递归地分割成两个小数组的排序问题，然后用所有小问题的答案去归并出大问题的答案。实现归并排序的另一种思路是自底向上的方式，先归并微型数组，然后成对地归并得到排好序的子数组，如此反复直到整个数组归并在一起。

```c++
void merge_sort() {
    int N = a.length;
    for (int sz = 1; sz < N; sz += sz) {
        for (int lo = 0; lo + sz < N; lo += sz + sz) {
            merge(lo, lo + sz - 1, min(lo + sz + sz - 1, N - 1));
        }
    }
}
```

- 快速排序

快速排序也是一种基于分治思想的排序算法，它将一个大数组的排序任务划分为两个独立的子数组的排序任务。快排在大数组中选择一个元素作为基准，基于基准元素对大数组进行划分，使得数组左半部分元素均小于基准，数组右半部分元素均大于基准，之后递归地对左/右两部分单独进行排序。快排不需要归并的过程，因为子数组排序好，大数组自然也就是有序的数组了。

```c++
int a[1000];

// 划分算法
int partition(int lo, int hi) {
    int i = lo, j = hi + 1;
    int v = a[lo]; // 基准元素v
    while (true) {
        while (a[++i] < v) {
            if (i == hi) break;
        }
        while (a[--j] > v) {
            if (j == lo) break;
        }
        if (i >= j) break;
        swap(i, j);
    }
    swap(lo, j);
    return j;
}

void quick_sort(int lo, int hi) {
    if (lo >= hi) return;
    int j = partition(lo, hi);
    quick_sort(lo, j - 1);
    quick_sort(j + 1, hi);
}

void main() {
    quick_sort(0, a.length - 1);
}
```

- 优先队列：堆

这里我们就只讲最简单的二叉大顶堆：在二叉堆的数组中，每个元素都大于等于它的两个子节点。堆的核心操作是：上浮和下沉。如果堆的有序状态因为某个节点比其父节点更大，那么需要通过上浮来交换它和它的父节点进行修复。如果堆的有序状态因为某个节点比它的两个子节点中之一更小，那么需要通过下沉来交换它和它的子节点中较大者来修复。

```c++
// 上浮
void swim(int k) {
    while (k > 1 && a[k] > a[k / 2]) {
        swap(k, k / 2);
        k = k / 2;
    }
}

// 下沉
void sink(int k) {
    while (2 * k <= N) {
        int j = 2 * k;
        if (j < N && a[j] < a[j + 1]) j++;
        if (a[k] >= a[j]) break;
        swap(k, j);
        k = j;
    }
}
```

### 第3章 查找

- 2-3查找树

这个之前有专门整理过：https://7color94.github.io/blog/2017/07/2-3-search-tree-notes/

- 红黑树

之后有时间再整理这部分。

- 散列表（hash表）

暂时先不做整理。

### 第4章 图

- 图的深搜和广搜

不多说。

- 最小生成树

不多说，知名的两种算法有：[prim](https://zh.wikipedia.org/wiki/%E6%99%AE%E6%9E%97%E5%A7%86%E7%AE%97%E6%B3%95) 和 [kruscal](https://en.wikipedia.org/wiki/Kruskal%27s_algorithm)。

- 最短路径

这部分很有意思，常用的最短路径算法有dijkstra、bellman-ford、spfa(bellman-ford的队列改进版)、floyd。

1.[dijkstra](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)

dijkstra采用了和prime类似的方法来计算最短路径树。首先将distTo[s]初始化为0，distTo[ ]中其他元素初始化为正无穷。然后将distTo[ ]中最小的非树顶点加入树中，同时用此顶点去松弛其余顶点的distTo[ ]值，直到所有顶点都在树中或所有有非树顶点的distTo[ ]值均为无穷大。其中，从distTo[ ]中找出元素最小的非树顶点可以用优先级队列来管理。

dijstra可以用于无向图，也可以用于有向图，但是不能用于含有负权边的图，可以读一读stackoverflow给出的例子：https://stackoverflow.com/questions/6799172/negative-weights-using-dijkstras-algorithm

2.[bellman-ford](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm)

bellman-ford算法流程很简单，重复以任意顺序对图的每条边松弛V轮。bellman-ford可以用于无向图/有向图，也可以用于含有负权重边的图，在算法的最后一步，还有判断图中是否存在负权环的功能。

3.[spfa](https://en.wikipedia.org/wiki/Shortest_Path_Faster_Algorithm)

在bellman-ford每一轮松弛中，只有distTo[ ]值发生变化的顶点指出的边才能改变其他顶点的distTo[ ]值，为了记录这些点，spfa利用队列记录。

三种算法代码实现：

```c++
int N, M, S, T;
int edgenum;
int first[100010];
int dis[100010];
int vis[100010];
int cnt[100010];

struct edge {
    int u, v;
    int w;
    int next;
} e[3000000];

struct CompareByFirst {
    bool operator()(pair<int, int> const &a, pair<int, int> const &b) {
        return a.first < b.first;
    }
};

void addEdge(int u, int v, int w) {
    e[edgenum].u = u;
    e[edgenum].v = v;
    e[edgenum].w = w;
    e[edgenum].next = first[u];
    first[u] = edgenum++;

    e[edgenum].u = v;
    e[edgenum].v = u;
    e[edgenum].w = w; 
    e[edgenum].next = first[v];
    first[v] = edgenum++;
}

void dijkstra() {
    priority_queue< pair<int, int>, vector< pair<int, int> >, CompareByFirst > Q;
    for (int i = 1; i <= N; i++) dis[i] = (i == S ? 0 : INT_MAX);
    Q.push(make_pair(0, S));
    while (!Q.empty()) {
        pair<int, int> t = Q.top();
        Q.pop();
        int u = t.second;
        if (t.first != dis[u]) continue;
        for (int k = first[u]; k != -1; k = e[k].next) {
            int v = e[k].v;
            if (dis[v] > dis[u] + e[k].w) {
                dis[v] = dis[u] + e[k].w;
                Q.push(make_pair(dis[v], v));
            }
        }
    }
}

bool bellman_ford() {
    for (int i = 1; i <= N; i++) dis[i] = (i == S ? 0 : INT_MAX);
    for (int i = 1; i <= N; i++) {
        bool changed = false;
        for (int j = 0; j < edgenum; j++) {
            if (dis[e[j].v] > dis[e[j].u] + e[j].w && dis[e[j].u] != INT_MAX) {
                dis[e[j].v] = dis[e[j].u] + e[j].w;
                changed = true;
            }
        }
        if (!changed) return true;
        if (i == N && changed) return false;
    }
    return false;
}

bool spfa() {
    queue<int> Q;
    memset(vis, 0, sizeof(vis));
    memset(cnt, 0, sizeof(cnt));
    for (int i = 1; i <= N; i++) dis[i] = (i == S ? 0 : INT_MAX);
    vis[S] = 1;
    Q.push(S);
    while (!Q.empty()) {
        int u = Q.front();
        vis[u] = 0;
        Q.pop();
        for (int k = first[u]; k != -1; k = e[k].next) {
            int v = e[k].v;
            if (dis[v] > dis[u] + e[k].w) {
                dis[v] = dis[u] + e[k].w;
                if (0 == vis[v]) {
                    Q.push(v);
                    vis[v] = 1;
                    if (++cnt[v] > N) return false;
                }
            }
        }
    }
    return true;
}

int main() {
    freopen("in.txt", "r", stdin);
    scanf("%d %d %d %d", &N, &M, &S, &T);
    edgenum = 0;
    memset(first, -1, sizeof(first));
    int u, v, w;
    for (int i = 1; i <= M; i++) {
        scanf("%d %d %d", &u, &v, &w);
        addEdge(u, v, w);
    }
    // dijkstra();
    // bellman_ford();
    spfa();
    printf("%d\n", dis[T]);
    return 0;
}
```

4.[floyd](https://zh.wikipedia.org/wiki/Floyd-Warshall%E7%AE%97%E6%B3%95)

floyd算法用于计算一张图的全源最短路径算法，可用于负权图。

```c++
int N, M;
int e[110][110];

int main() {
    freopen("in.txt", "r", stdin);
    scanf("%d %d", &N, &M);
    int u, v, w;
    for (int i = 1; i <= N; i++) {
        for (int j = 1; j <= N; j++) {
            if (i == j) e[i][j] = 0;
            else e[i][j] = 100000;
        }
    }
    for (int i = 1; i <= M; i++) {
        scanf("%d %d %d", &u, &v, &w);
        e[u][v] = e[v][u] = min(w, e[u][v]);
    }
    
    for (int k = 1; k <= N; k++) {
        for (int i = 1; i <= N; i++) {
            for (int j = 1; j <= N; j++) {
                if (e[i][j] > e[i][k] + e[k][j]) e[i][j] = e[i][k] + e[k][j];
            }
        }
    }

    for (int i = 1; i <= N; i++) {
        for (int j = 1; j < N; j++) printf("%d ", e[i][j]);
        printf("%d\n", e[i][N]);
    }
    
    return 0;
}
```

### 第5章 字符串

- 字符串排序

待整理

- 单词查找树

最基本的Trie树实现：

```c++
#define CHAR_RANGE 26

struct TrieNode {
    bool flag;
    int count;
    TrieNode *childs[CHAR_RANGE];
    TrieNode *parent;
    TrieNode() :parent(NULL), flag(false), count(0) {
        for (int i = 0; i < CHAR_RANGE; i++) childs[i] = NULL;
    }
};

void insertWord(TrieNode *p, char *word) {
    if ((*word) == '\0') {
        p->flag = true;
        return;
    }
    int index = word[0] - 'a';
    if (p->childs[index] == NULL) p->childs[index] = new TrieNode();
    if (p->childs[index]->parent == NULL) p->childs[index]->parent = p;
    p->childs[index]->count++;
    insertWord(p->childs[index], word + 1);
}

int findPrefix(TrieNode *p, char *word) {
    if ((*word) == '\0') {
        return p->count;
    }
    int index = word[0] - 'a';
    if (p->childs[index] == NULL) return 0;
    return findPrefix(p->childs[index], word + 1);
}
```

- 字符串查找

其中KMP算法之前有整理过：https://7color94.github.io/blog/2017/01/kmp/

其余字符串查找算法有待整理

- 正则表达式和数据压缩

这部分待整理