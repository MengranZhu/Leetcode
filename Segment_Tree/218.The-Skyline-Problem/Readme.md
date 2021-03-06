### 218.The-Skyline-Problem

#### 解法1:有序容器

此题需要设置一个multiSet记录所有的当前下降沿的高度，则*prev(Set.end(),1)就是这个Set里的最大值。

首先，将所有的edges放入一个数组，按时间顺序排序，然后顺次遍历考虑：如果是上升沿，则在Set里加入对应高度（即添加一个上升沿）；如果是下降沿，则需要在Set里删除对应的高度（即退出当前的下降沿）。

那何时对results进行更新呢？我们在每次处理edge时，不管是加入上升边沿还是退出下降沿之后，都意味着天际线有可能变动。天际线会变成什么呢？答案是此时Set里的最大值！回想一下，Set里装的是所有当前仍未退出的下降沿，说明他们都在当前可以撑起对应的高度。那么Set里的最大值就是当前天际线的最高值。

所以每次查看一个edges，我们都要比较当前的高度（用cur记录）和Set里的最大值进行比较：一旦不同，就用Set里的最大值去加入results，同时也要更新cur。

有一个细节需要注意，在生成edges数组时，如果某一个位置同时有上升沿也有下降沿，注意要先考察上升沿，再考察下降沿。也就是要先加入一个上升沿，再退出可能的下降沿。否则类似[[0,2,3],[2,5,3]]的测试例子就会有问题。

#### 解法2:线段树

此题的线段树解法，是以```370.Range-Addition```的懒标记版本为基础，稍加改动。

首先，不同于307或者370中已经给出了nums数组。此题中给出的是建筑物高度拐点的pos。这些pos的绝对值对我们而言其实是没有意义的，我们只关心这些pos的相对顺序，也就是idx。所以我们其实需要构造两个映射，pos2idx和idx2pos，方便我们做转换。转换之后，每个builiding相当于一个区间更新。特别注意，如果一个building对应的索引区间是```[i:j]```，那么我们只更新区间```[i:j-1]```。也就是说，线段树中的每个叶子节点的单点info，代表的其实是```[i:i+1)```这段左闭右开的小区间。

然后我们设计一下SegTreeNode的数据结构。除了常规的left,right,start,end之外，此时节点的info表示的是该区间内出现的高度最大值。注意，不一定整个区间都是该恒定的高度。此外，该节点的tag是一个bool量，表示该区间的高度是否都是相同的（也就是等于info）。tag为true时说明我们能断言整个区间的信息是一致的，我们不需要下沉去更新每一块更细的区间。下文会详细说明。

初始化函数```init(root, 0, n-1)```不需要特别的改动。其中n就是离散化之后有多少个数据点。初始化时，每个节点的info设置为0，tag设置为false。

区间更新函数```updateRange(node, a, b, val)```是本题的重点，意思是将区间[a,b]里的所有数据点的最大值更新为val（如果原最大值小于val的话）。我们分几种情况讨论：
1. 边界条件：[a,b]与该节点区间互斥，直接return。
2. 边界条件：该节点的区间是单点，那么```node->info = max(node->info, val)```然后return
3. 边界条件：如果该节点的区间包含在[a,b]内，并且```node->info <= val```，说明该节点区间的数据点可以统一更新为val。并且我们就设置tag标记true，根据之前的分析，我们可以直接返回而不用下沉去更新更细节的区间。
4. 其他情况下，我们就递归处理```updateRange(node->left, a, b, val) && updateRange(node->right, a, b, val)```。但是递归之前，我们别忘了查看当前的tag是否为true。如果是的话，说明当前节点的区间不再有统一的高度，我们需要去掉tag的标记，并且将其影响传递给两个子区间，即
```cpp
        if (node->tag == 1) // if current node tagged lazy, push information down
        {
            node->tag = 0;            
            node->left->info = node->info;
            node->right->info = node->info;
            node->left->tag = 1;
            node->right->tag = 1;            
        }   
```
对于此题，我们其实不需要写区间查询的函数。因为我们最终输出的结果，会要求遍历线段树的每一个节点，写个DFS即可。当然，如果遍历到某个区间的tag依然为true时，其实可以不用继续深入下去，直接输出该区间的信息。最终输出的结果需要整理一下，如果相邻的叶子节点的高度相同，我们再做一下归并。

[Leetcode Link](https://leetcode.com/problems/the-skyline-problem)
