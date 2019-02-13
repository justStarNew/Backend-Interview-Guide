
# 1. 问题描述

TopK Elements 问题用于找出一组数中最大的 K 个的数。

![](https://diycode.b0.upaiyun.com/photo/2019/e96d0bc52e3a0b38fc5f3976ca78e6b3.png)

此外还有一种叫 Kth Element 问题，用于找出一组数中第 K 大的数。

![](https://diycode.b0.upaiyun.com/photo/2019/54a40d9e1d1d4e058d2e633de10523eb.png)


其实要求解 TopK Elements，可以先求解 Kth Element，因为找到  Kth Element 之后，再遍历一遍，大于等于 Kth Element 的数都是 TopK Elements。

# 2. 一般解法

以 [Leetcode : 215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/description/) 为例，这是一道的 Kth Element 问题，不过这道题要找的是从后往前第 K 个元素，而不是从前往后。为了能和传统的 Kth Element 问题一样求解，可以先执行 `k = nums.length - k;`。

```
Input: [3,2,1,5,6,4] and k = 2
Output: 5
```

## 2.1 快速选择

快速排序的 partition() 方法，对于数组 nums 的 [l, h] 区间，会返回一个整数 k 使得 nums[l..k-1] 小于等于 nums[k]，且 nums[k+1..h] 大于等于 nums[k]，此时 nums[k] 就是数组的第 k 大元素。可以利用这个特性找出数组的 Kth Element，这种找 Kth Element 的算法称为快速选择算法。

- 时间复杂度 O(N)、空间复杂度 O(1)
- 只有当允许修改数组元素时才可以使用

![](https://diycode.b0.upaiyun.com/photo/2019/9b146e139d04105c8aeda8dce65dcf6f.png)

```java
public int findKthElement(int[] nums, int k) {
    k = nums.length - k;
    int l = 0, h = nums.length - 1;
    while (l < h) {
        int j = partition(nums, l, h);
        if (j == k) {
            break;
        } else if (j < k) {
            l = j + 1;
        } else {
            h = j - 1;
        }
    }
    return nums[k];
}

private int partition(int[] a, int l, int h) {
    int i = l, j = h + 1;
    while (true) {
        while (a[++i] < a[l] && i < h) ;
        while (a[--j] > a[l] && j > l) ;
        if (i >= j) {
            break;
        }
        swap(a, i, j);
    }
    swap(a, l, j);
    return j;
}

private void swap(int[] a, int i, int j) {
    int t = a[i];
    a[i] = a[j];
    a[j] = t;
}
```

## 2.2 堆

维护一个大小为 K 的最小堆，堆顶元素就是 Kth Element。

使用大顶堆来维护最小堆，而不能直接创建一个小顶堆并设置一个大小，企图让小顶堆中的元素都是最小元素。

维护一个大小为 K 的最小堆过程如下：在添加一个元素之后，如果大顶堆的大小大于 K，那么需要将大顶堆的堆顶元素去除。

- 时间复杂度 O(NlogK) 、空间复杂度 O(K)
- 特别适合处理海量数据

![](https://diycode.b0.upaiyun.com/photo/2019/0a94de372b0d665417844751e00c24c3.gif)

```java
public int findKthLargest(int[] nums, int k) {
    k = nums.length - k + 1;
    PriorityQueue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder()); // 大顶堆
    for (int val : nums) {
        pq.add(val);
        if (pq.size() > k)  // 维护堆的大小为 K
            pq.poll();
    }
    return pq.peek();
}
```

# 3. 海量数据

在这种场景下，单机通常不能存放下所有数据。

- 拆分，可以按照哈希取模方式拆分到多台机器上，并在每个机器上维护最大堆；
- 整合，将每台机器得到的最大堆合并成最终的最大堆。

# 4. 频率统计

Heavy Hitters 问题要求找出一个数据流的最频繁出现的 K 个数，比如热门搜索词汇等。

## 4.1 HashMap

使用 HashMap 进行频率统计，然后使用快速选择或者堆的方式找出频率 TopK。在海量数据场景下，也是使用先拆分再整合的方式来解决空间问题。

## 4.2 Count-Min Sketch

维护 d*w 大小的二维统计数组，其中 d 是哈希函数的个数，w 根据情况而定。

- 在一个数到来时，计算 d 个哈希值，然后分别将哈希值对 w 取模，把对应统计数组上的值加 1；
- 要得到一个数的频率，也是要计算 d 个哈希值并取模，得到 d 个频率，取其中最小值。

该算法的思想和布隆过滤器类似，具有一定的误差，特别是当 w 很小时。但是它能够在单机环境下解决海量数据的频率统计问题。

![](https://diycode.b0.upaiyun.com/photo/2019/1e6f5d5dc2cf7620e3cf0f1229a49b50.png)

```java
public class CountMinSketch {

    private int d;
    private int w;

    private long estimators[][];

    public CountMinSketch(int d, int w) {
        this.d = d;
        this.w = w;
    }

    public void add(int value) {
        for (int i = 0; i < d; i++)
            estimators[i][hash(value, i)]++;
    }

    public long estimateFrequency(int value) {
        long minimum = Integer.MAX_VALUE;
        for (int i = 0; i < d; i++) {
            minimum = Math.min(minimum, estimators[i][hash(value, i)]);
        }
        return minimum;
    }

    private int hash(int value, int i) {
        return 0; // use ith hash function
    }
}
```

## 4.3 Trie

Trie 树可以用于解决词频统计问题，只要在词汇对应节点保存出现的频率。它很好地适应海量数据场景，因为 Trie 树通常不高，需要的空间不会很大。

![](https://diycode.b0.upaiyun.com/photo/2019/e50b2538722fa4dde6f91b7ece4db0e7.png)

# 参考资料

- [Probabilistic Data Structures for Web Analytics and Data Mining](https://dirtysalt.github.io/html/probabilistic-data-structures-for-web-analytics-and-data-mining.html)
- [Trie](https://zh.wikipedia.org/wiki/Trie)

# 原文链接

https://xiaozhuanlan.com/topic/4176082593