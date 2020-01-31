---
layout: post
title: decoder阶段的生成方式
description: "3种decoder搜索方式"
modified: 2020-01-31
tags: [decoder, search]
image:
  feature: abstract-5.jpg
---

>生成式模型在decoder阶段是一个单词一个单词蹦出来的，但是怎么个蹦法，有很多种方式。以seq2seq模型为例，
![](http://latex.codecogs.com/gif.latex?p(E|F\)=P(e_1^N|f_1^M)=\prod_{i=1}^NP(e_i|e_1^{i-1},f_1^M)),
那么在decoder阶段，我们的搜索复杂度是多大？
假设输出序列![](http://latex.codecogs.com/gif.latex?e_1,e_2,e_3),
target sequence的词汇表大小是![](http://latex.codecogs.com/gif.latex?V_E)，
那么对于每个单词的搜索空间都是![](http://latex.codecogs.com/gif.latex?V_E)，
那么长度为N的序列，搜索复杂度就是![](http://latex.codecogs.com/gif.latex?V_E^N)，
可以看出，如果词汇量很大的话，N越长的话，效率太低。

# decoder阶段搜索算法

## 贪心算法
贪心算法很简单，每一步选择概率最大的路径，就是以当前步为起点，找到下一步最大概率的边。

## viterbi算法 
viterbi算法经常被用于HMM的decoder阶段，使用动态规划的方法求解最优路径。

动态规划思想，在第t步的第k个结点，它的概率等于所有前驱节点概率加连接线概率和的最小值。
整个过程可以抽象成一个填表的过程，![](http://latex.codecogs.com/gif.latex?V_E*N)。
找到最优路径后，然后再回溯父节点，就可以输出最优路径。
整个过程时间复杂度为![](http://latex.codecogs.com/gif.latex?O(V_E^2*N\))，
空间复杂度为![](http://latex.codecogs.com/gif.latex?O(V_E*N\))。

>缺点：算法效率提升很高，当V的大小很小的时候，效率还是很高的。但是，如果词典V很大的话，还是有点高的。

## beam search
beam search就是viterbi方法的近似解，可以降低一下时间复杂度，在第t步的第k个节点计算的时候，
前驱节点从V降低到beam search size大小B，B << VE。
第t步的所有节点计算完之后，排序，选择top B个概率最高的点，用于计算第t+1步。
整个过程，每一步时间复杂度降低到![](http://latex.codecogs.com/gif.latex?BN+BV_Elog_{V_{E}}),
因为每一步排序所需要的时间复杂度![](http://latex.codecogs.com/gif.latex?V_Elog_{V_{E}})。
这样，整体的时间复杂度就是![](http://latex.codecogs.com/gif.latex?O(NBV_Elog_{V_{E}}\))，
空间复杂度是![](http://latex.codecogs.com/gif.latex?O(BN\)).

>viterbi算法的过程是填一个V x N的表格，那么beam search算法的过程就是填一个B x N的表格,因此效率高了很多。

# 相关的搜索算法

## 广度搜索
算法思路：从起点开始，遍历周围邻近的点，阿然后再遍历已经遍历过的点的邻近节点，逐步向外扩散，直到找到终点

>缺点：算法过程会遍历图中所有的点，但是没有必要，对于有明确终点的问题，可能会尝试了很多种路径。

## Dijkstra算法
广度优先搜索过程有一个假设，就是邻近节点之间的权重是相同的，如果节点之间的边的权重不完全相同，那么在遍历的过程中，需要考虑搜索的总代价，这就是
Dijkstra算法。

>算法思路：在扩算的过程中，考虑每个节点距离起点的总代价；将当前遍历到的节点放到优先级队列中，按照优先级排序，每次都取出优先级队里中总代价最小的
节点，直到遍历到终点。

>重点：优先级队列

## 最佳优先搜索（Best First算法）
>前提假设：预先计算出每个节点到终点的距离

>算法思路：和Dijkstra一样使用优先队列，区别在于，使用节点到终点的距离作为优先级；然后再遍历的过程中，选择最高优先级的节点遍历

>好处：加快搜索速度

>缺点：如果起点和终点之间存在障碍物，那么最佳搜索算法找到的可能并不是最短路径

## A*算法
>算法核心：A*算法是结合了Dijkstra算法和最佳优先搜索算法的优点，使用优先级队列，每个节点的优先级既考虑节点到起点的代价，也考虑节点到终点的代价，
即f(n) = g(n) + h(n)，其中：f(n)是节点n的优先级，g(n)是节点n距离起点的代价，h(n)是节点n距离终点的代价

代价函数对算法过程的影响总结：
* 如果h(n)=0，那么A*算法等价于Dijkstra算法；
* 如果h(n)始终小于等于节点n到终点的代价，那么算法一定能够保证找到最短路径；但是当h值越小，算法将遍历更多的节点，即到达终点的速度越慢；
* 如果h(n)完全等于节点到终点的代价，那么算法将找到最佳路径，并且速度很快；
* 如果h(n)比节点n到终点的代价大，那算法并不保证找到最短路径，不过此时会很快；
* 如果h(n)比g(n)大很多，那么此时仅h生效，就变成了最佳优先搜索

# 距离函数
网状结构的图中，搜索算法常使用的距离函数
* 如果只有上下左右四个方向，可以使用曼哈顿距离；
* 如果运行向8个方向移动，使用对角距离；
* 如果可以向任意方向易懂噢，使用欧几里得距离；

```python
def manhattan_distance(node, goal):
    """D是一个常数，表示两个相邻节点之间的移动代价，可以取常数1
    """
    dx = abs(node.x - goal.x)
    dy = abs(node.y - goal.y)
    return D * (dx + dy)
    
def diagonal_distance(node, goal):
    '''D2是一个常数，表示两个斜着相邻节点之间的移动代价。如果是每个节点都是正方形，那么可以D2=根号2*D
    '''
    dx = abs(node.x - goal.x)
    dy = abs(node.y - goal.y)
    return D * (dx + dy) + (D2 - 2 * D) * min(dx, dy) 
    
def euclidean_distance(node, goal):
    dx = abs(node.x - goal.x)
    dy = abs(node.y - goal.y)
    return D * sqrt(dx * dx + dy * dy)
```

## A*算法变种 （续）

[参考文章](https://www.jianshu.com/p/a3951ce7574d)