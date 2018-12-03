---
title: LeetCode 之三角形最小路径和（Triangle）
date: 2018-12-3 16:54:48
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

看标题不知是否让您想起了有向图中的最短路径，是有些许类似，不过该题比其更简单更加清晰、直观、好理解。相信您看完这个之后，脑回路肯定更加的明亮！

题目描述如下：
```
给定一个三角形，找出自顶向下的最小路径和。每一步只能移动到下一行中相邻的结点上。

例如，给定三角形：

[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
自顶向下的最小路径和为 11（即，2 + 3 + 5 + 1 = 11）。

说明：

如果你可以只使用 O(n) 的额外空间（n 为三角形的总行数）来解决这个问题，那么你的算法会很加分。
```

初看题目，上下有关联，有结束点，是个适合用递归的题目。以三角形的“行数”作为递归的结束判断，行数加上下标作为参数来回穿插。可行，可行！

于是大笔一挥：
```
    int size = 0;
    TreeSet<Integer> ts = new TreeSet<>();
    List<List<Integer>> list;

    /**
     * 超时
     * 
     * @param triangle
     * @return
     */
    public int minimumTotal(List<List<Integer>> triangle) {
        list = triangle;
        size = triangle.size();
        if (size == 0)
            return 0;

        helper(0, 0, 0);
        return ts.first();
    }

    /**
     * 
     * @param row
     *            行数
     * @param index
     *            在该行中下标
     * @param sum_length
     *            路径之和
     */
    private void helper(int row, int index, int sum_length) {
        if (row >= size) {
            ts.add(sum_length);
            return;
        }
        sum_length += list.get(row).get(index);
        helper(row + 1, index, sum_length);
        helper(row + 1, index + 1, sum_length);
    }
```

是的，代码没有问题，可是拿去跑的时候，数据较大的测试用例却给了一个大红色的 `超出时间限制`。这里将 `TreeSet` 换成一个 MIN_LENGTH 整型变量每次进行比较取较小值，结果一样，都是超时。

本来是满怀欣喜，豪气撸码，结果给撞了个豆腐墙。

墙不硬，问题不大。我们换个思路，再摸摸青青草地。

以往出现这种求最小值、最大值啊，需要数据之间相关联相加减乘除的啊，用的都是 **DP** 居多啊！

脑浆乍现，回路高速擦亮，越擦越亮，越擦越闪，终于“吡”得一身，为数不多的小草又飘下几根，成了！

**我也不用额外的空间了，就在你身上肆虐！**

再回头看下那个“三角形”：
```
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
```

既然是要一个最小值，那我就逐步缩减法，自下而上攻之。

根据题目要求我们知道如果上一行的元素下标为 i，那它只能跟它下一行的下标为 i 和 i+1 两元素相加。当然，我们只需要这两者的较小值。

即：
```
 [6,5,7]   // i 行
[4,1,8,3]  // j 行

以这两行为例 （条件：j = i + 1）

i 行中的 6 可以跟 j 行中的 4 或者 1 相加，由于我们结果取的是最小值，所以我们只留较小值
6 + 4 = 10, 6 + 1 = 7
因为 10 > 7，所以 i 行中的 6 我们就随之替换为 7

剩余元素同理，i 行最终便成了：
[7,6,10]

再由此，层层攻上。随着数量越来越少，最终顶上的那位佼佼者便是我们要取的首级！
```

武器献之：
```
    /**
     * DP
     * 
     * @param triangle
     * @return
     */
    public int minimumTotal(List<List<Integer>> triangle) {

        // 从倒数第二行开始往上走
        for (int i = triangle.size() - 2; i >= 0; i--) {
            // 从每行的起始下标开始直到 i
            for (int j = 0; j <= i; j++) {
                // i 行 j 下标的值
                int self = triangle.get(i).get(j);
                // 将 i 行 j 下标的值赋为 ： i 行 j 下标的值 与 i+1 行 j 下标和 j+1 下标值之和的较小值
                triangle.get(i).set(j,
                        Math.min(triangle.get(i + 1).get(j) + self, triangle.get(i + 1).get(j + 1) + self));
            }
        }
        // 层层往上 顶层值便是路径最小值
        return triangle.get(0).get(0);
    }
```

这道题容易理解，对 DP（动态规划） 会有一个较为清晰的认知，我认为还是很不错的，故特意整理之。

---
文笔不好，望见谅！ End.