---
layout: post
title: "基于JDK TreeMap分析树的后继结点查询算法"
subtitle: "JDK TreeMap successor()方法分析"
author: "iwts li"
date: 2019-02-13 11:11:47
categories: 数据结构
header-style: text
tags:
  - 数据结构
  - 树
description: "基于JDK TreeMap分析树的后继结点查询算法，TreeMap的successor()方法分析"
---

核心在于TreeMap的successor()方法

只要理解二叉搜索树，就能看懂这个方法了。注释为：

> Returns the successor of the specified Entry, or null if no such.

就是说返回当前结点的后继结点。

然而TreeMap底层是红黑树谈何后继结点？这个方法最早出现是在containsValue方法中。也就是传入一个Value，判定是否存在。

也就是说我们需要遍历整个红黑树。但是这样的遍历用递归肯定是不行的，性能太差，而BFS又需要实现queue，不显示，所以就有了successor方法，能够快速获取结点的后继结点。

这里我们先看图。因为虽然TreeMap的底层是红黑树，但是在不涉及平衡的前提下，其他操作其实跟二叉搜索树一样，所以图都按照二叉搜索树来画：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405232356591.png)

我们认为，将BST“压平”，就是一个有序数组。而所谓的后继结点，就是指在“数组状态”下，按顺序遍历的情况，可以理解为迭代。

严格地说，有点类似严格ceil方法：寻找大于该结点的所有结点中，值最小的结点。

可以看到，不管怎样，对于一个结点而言，下一个节点一定是在树的右边，区别在于：比当前树更高还是更低：

1. 比当前结点更低：就是该结点有右子树的情况，那么这个值很好确定：右子树的最小值。可以看下图，如果查找1的下一个结点，就是找右子树的最小值：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405232357266.png)

2. 比当前结点更高：就是该结点没有右子树但是有比该结点大的祖先结点。可以看下面的图，这个也是非常核心的算法：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405232357868.png)

3. 当前结点就是最大值，这就需要返回null了。最大值的时候只有这两种可能：

![](https://cdn.jsdelivr.net/gh/iwts/blog-imgs-repo/202405232358151.png)

这样来说，我们可以先看1的情况如何实现：

```java
if (t.right != null) {
    Entry<K,V> p = t.right;
    while (p.left != null)
        p = p.left;
    return p;
}
```

因为2、3都是建立在不存在右子树的情况，所以存在右子树，一定就是找右子树的最小值。

核心就是2、3——如何判定到底该结点是不是最大值？

我们假设从该节点回溯到根节点是“上山的过程”，那么满足条件2的情况实际上就是在一直遍历父结点的时候“往右拐了”一下。

看2的图和3的图：如果父结点一直都是通过右子树，直到遍历到根，那么说明该节点就是最大值，而如果其中一个父结点，是左子树进入到该节点的，那么这个父结点就是我们要找的值。算法非常简单明了：

```java
Entry<K,V> p = t.parent;
Entry<K,V> ch = t;
while (p != null && ch == p.right) {
    ch = p;
    p = p.parent;
}
return p;
```

这个while的过程就是不断向上回溯的过程。在保证p不为null的时候，一直在判定ch是否是p的右子结点。如果是，则继续向上找，如果不是，说明该结点就是第一个大于值的结点。如果直到p == null都没有找到，那么就说明该结点是整个树的最大值。

源码也很简单：

```java
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```
