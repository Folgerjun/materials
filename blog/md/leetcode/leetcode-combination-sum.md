---
title: LeetCode 之组合总和系列（Combination Sum）
date: 2018-11-14 11:29:02
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

LeetCode 中有多道组合总和（Combination Sum）的题，这些题目都是比较经典的，面试很可能会问到。我这一想，还真是。今天就来简单总结下这一系列题目，总结很重要，还要时而回顾！

## Combination Sum I

第一道题的描述如下：
```
给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的数字可以无限制重复被选取。

说明：

所有数字（包括 target）都是正整数。
解集不能包含重复的组合。 
示例 1:

输入: candidates = [2,3,6,7], target = 7,
所求解集为:
[
  [7],
  [2,2,3]
]
示例 2:

输入: candidates = [2,3,5], target = 8,
所求解集为:
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```

**注意点：**

- **candidates 中的数字可以无限制重复被选取。**
- **解集不能包含重复的组合。**

这时候头脑里很快就会产生一个思路：无限遍历该数组，直到其元素之和满足条件，然后塞入集合中。

这个想法没问题，不过在这过程中我们怎么才能满足`解集不能包含重复的组合。`这个条件呢？

对对对，那就是确定一个 index，我们要保证数组不往回找就是了。

解题代码如下：
```
public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> list = new ArrayList<>();
        Arrays.sort(candidates);
        recursive(list, new ArrayList<>(), candidates, target, 0);
        return list;
    }

    /**
     * 
     * @param list
     *            总的输出 list
     * @param tempList
     *            存放的 list
     * @param nums
     *            数组
     * @param remain
     *            剩余值
     * @param index
     *            数组下标
     */
    private void recursive(List<List<Integer>> list, List<Integer> tempList, int[] nums, int remain, int index) {

        if (remain < 0) // return 或者进行 add 操作后就开始执行弹出尾部元素 塞入下个元素
            return;
        else if (remain == 0)
            list.add(new ArrayList<>(tempList)); // 这里需要注意不能直接 list.add(tempList),最终 tempList 所指向的对象是空的,
                                                    // 所以需要 new 一个新对象，将值复制进去
        else {
            for (int i = index; i < nums.length; i++) {
                tempList.add(nums[i]); // 挨个塞入
                recursive(list, tempList, nums, remain - nums[i], i); // 由于元素可重复 所以是 i
                tempList.remove(tempList.size() - 1); // 挨个弹出
            }
        }
    }
```

我这注释什么的都有，理解起来不成问题吧。这里需要注意的就是两个点：`list.add(new ArrayList<>(tempList));` 和 ` recursive(list, tempList, nums, remain - nums[i], i);`。

这里先对数组进行排序可以使计算更快，当然你也可以不排序。

## Combination Sum II

第二道对第一道稍微加工了一下，描述如下：
```
给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中只能使用一次。

说明：

所有数字（包括目标数）都是正整数。
解集不能包含重复的组合。 
示例 1:

输入: candidates = [10,1,2,7,6,1,5], target = 8,
所求解集为:
[
  [1, 7],
  [1, 2, 5],
  [2, 6],
  [1, 1, 6]
]
示例 2:

输入: candidates = [2,5,2,1,2], target = 5,
所求解集为:
[
  [1,2,2],
  [5]
]
```

**注意点：**

- **candidates 中的每个数字在每个组合中只能使用一次。**
- **解集不能包含重复的组合。**

是的，这里把条件改了，数组中的每个元素在每个组合中只能使用一次！

这就意味着每次递归操作时其下标都需要往后移一位了！

代码如下：
```
public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        List<List<Integer>> list = new ArrayList<>();
        Arrays.sort(candidates);
        recursive(list, new ArrayList<>(), candidates, target, 0);
        return list;

    }

    /**
     * DFS 添加每个满足条件的集合
     * 
     * @param list
     *            最终返回集合
     * @param tempList
     *            每个满足条件的子集合
     * @param candidates
     *            数组
     * @param remain
     *            剩余值
     * @param index
     *            数组下标
     */
    private void recursive(List<List<Integer>> list, List<Integer> tempList, int[] candidates, int remain, int index) {
        if (remain < 0)
            return;
        else if (remain == 0)
            list.add(new ArrayList<>(tempList));
        else {
            for (int i = index; i < candidates.length; i++) {
                if (i > index && candidates[i] == candidates[i - 1]) // 说明两个值相等且之前一个值已经返回
                    continue;
                tempList.add(candidates[i]);
                recursive(list, tempList, candidates, remain - candidates[i], i + 1); // 规定数组中每个数字在每个组合中只能使用一次
                tempList.remove(tempList.size() - 1);
            }
        }
    }
```

相比较上一题的代码，这次只是加了一个条件判断以及递归中元素下标加了一。

这里就要对数组先进行排序操作了，新增的条件判断也是为了优化代码执行速度。

## Combination Sum III

再来看第三道变形，描述如下：
```
找出所有相加之和为 n 的 k 个数的组合。组合中只允许含有 1 - 9 的正整数，并且每种组合中不存在重复的数字。

说明：

所有数字都是正整数。
解集不能包含重复的组合。 
示例 1:

输入: k = 3, n = 7
输出: [[1,2,4]]
示例 2:

输入: k = 3, n = 9
输出: [[1,2,6], [1,3,5], [2,3,4]]
```

这次不给出数组了，而是规定了数组中元素的取值范围。

双手一摊，这还不是一样吗，不给我就自己造就是了！

**注意点：**

- **解集不能包含重复的组合。**

解题代码如下：
```
public List<List<Integer>> combinationSum3(int k, int n) {
        int[] nums = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
        List<List<Integer>> list = new ArrayList<>();
        dfs(list, new ArrayList<>(), nums, k, n, 0);
        return list;

    }

    /**
     * 
     * @param list
     *            最终返回的 list
     * @param tempList
     *            作为寻求满足条件的暂存 list
     * @param nums
     *            数组
     * @param reNum
     *            剩余的元素个数
     * @param reSum
     *            剩余的总和
     * @param index
     *            数组下标
     */
    private void dfs(List<List<Integer>> list, List<Integer> tempList, int[] nums, int reNum, int reSum, int index) {
        if (reNum < 0 || reSum < 0) // 两者有其一小于 0 就是不满足条件
            return;
        if (reNum == 0 && reSum == 0) // 两者都为 0 时满足条件，塞入
            list.add(new ArrayList<>(tempList));
        else if (reNum == 0 || reSum == 0) // 两者中有且只有一者等于 0 就不满足条件
            return;
        else {
            for (int i = index; i < nums.length; i++) {
                tempList.add(nums[i]);
                dfs(list, tempList, nums, reNum - 1, reSum - nums[i], i + 1); // 不能重复元素 所以 i + 1
                tempList.remove(tempList.size() - 1);
            }
        }
    }
```

真不愧为机智小少年……

## Combination Sum IV

第四道变形就复杂了一些了，描述如下：
```
给定一个由正整数组成且不存在重复数字的数组，找出和为给定目标正整数的组合的个数。

示例:

nums = [1, 2, 3]
target = 4

所有可能的组合为：
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)

请注意，顺序不同的序列被视作不同的组合。

因此输出为 7。
```

这里数组的元素可以重复，且组合的元素也能相同，但是顺序要不同。

一开始当然就是无脑循环了，然后满足条件就加上一，结果就是超时……

说明这么暴力不行的……

后来一想，无脑循环就相当于每一次都从头进行循环算了一遍，很多都是重复的计算。

对了，DP 浮现在了脑中，在那飘啊飘。

代码如下：
```
private int[] dp;

    /**
     * DP
     * 
     * @param nums
     * @param target
     * @return
     */
    public int combinationSum4(int[] nums, int target) {
        dp = new int[target + 1];
        Arrays.fill(dp, -1);
        dp[0] = 1;
        return helper(nums, target);
    }

    private int helper(int[] nums, int target) {
        if (dp[target] != -1) {
            return dp[target]; // 若是之前的已经计算过了 直接返回
        }
        int res = 0;
        for (int i = 0; i < nums.length; i++) {
            if (target >= nums[i]) { // 所有组合的种数相加
                res += helper(nums, target - nums[i]);
            }
        }
        dp[target] = res; // 计算后赋值用于之后的数字使用
        return res;
    }
```

用空间换取时间的操作，DP + 递归稍微费点脑。

还有一种解法：
```
/**
     * DP
     * 
     * @param nums
     * @param target
     * @return
     */
    public int combinationSum4(int[] nums, int target) {
        int[] comb = new int[target + 1];
        comb[0] = 1;
        for (int i = 1; i < comb.length; i++) {
            for (int j = 0; j < nums.length; j++) {
                if (i - nums[j] >= 0) {
                    comb[i] += comb[i - nums[j]];
                }
            }
        }
        return comb[target];
    }
```

好了，组合总和系列（Combination Sum）的题目总结完了。记住了**回溯**。