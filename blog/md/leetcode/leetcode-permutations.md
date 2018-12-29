---
title: LeetCode 之全排列（Permutations）
date: 2018-12-29 15:29:47
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

全排列问题在这里有两个版本，其中略有差异。看完就会感觉似曾相识，一种莫名的熟悉感从心底喷涌上来。

第一个版本：
```
给定一个没有重复数字的序列，返回其所有可能的全排列。

示例:

输入: [1,2,3]
输出:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]

```
有什么感觉？

这不就是暗箱摸球，箱子里有不同颜色的球 n 个，列出你可能会摸出球的所有顺序，不放回。

先贴上大神的详细解析：[链接](https://leetcode.com/problems/permutations/discuss/18436/Java-Clean-Code-Two-recursive-solutions)

利用 List.add(int index, E element) 方法可以将元素插入指定的位置，可以满足题意。

**整体分析：根据可插入的地方不同从而展开不同的分支，典型的树形结构。**

代码如下：
```
    public List<List<Integer>> permute(int[] nums) {

        List<List<Integer>> permutations = new ArrayList<>();
        if (nums.length == 0)
            return permutations;
        helper(nums, 0, new ArrayList<>(), permutations);
        return permutations;

    }

    /**
     * 
     * @param nums
     *            原数组
     * @param start
     *            选择填充数字下标
     * @param permutation
     *            单个集合
     * @param permutations
     *            目标返回集合
     */
    private void helper(int[] nums, int start, List<Integer> permutation, List<List<Integer>> permutations) {

        // 满足条件添加 返回
        if (permutation.size() == nums.length) {
            permutations.add(permutation);
            return;
        }
        // 分别插入不同的位置
        for (int i = 0; i <= permutation.size(); i++) {
            // 避免在原集合上操作 需新集合
            List<Integer> newPermutation = new ArrayList<>(permutation);
            newPermutation.add(i, nums[start]);
            helper(nums, start + 1, newPermutation, permutations);
        }
    }
```

**注意每次插入前要 new 一个新集合对象，不能在原集合对象上操作。**

第二个版本：
```
给定一个可包含重复数字的序列，返回所有不重复的全排列。

示例:

输入: [1,1,2]
输出:
[
  [1,1,2],
  [1,2,1],
  [2,1,1]
]
```
与第一个版本区别是：**该版本数组中可以有重复元素，且返回的集合中不能有重复的元素（即重复的集合排列顺序）。**

> 去重，我首先想到的是这样：不管你序列中有没有重复的数字，我就当满足条件的时候判断下是否已经存过了该排列顺序不就行了。

然后我就将使用过的数字全都按顺序拼接成字符串，too young ...

指定位置插入元素，List 很好实现，可是 String 怎么办。方法肯定有啊，比如每个元素之间用自定义分隔符拼接，再转成数组找到指定的位置插入，再转。只要功夫深，铁杵磨成针。可是麻烦啊，算暴力解法，遂弃之。

解题代码如下：
```
    public List<List<Integer>> permuteUnique(int[] nums) {
        List<List<Integer>> permuteUniques = new ArrayList<>();
        if (nums == null || nums.length == 0)
            return permuteUniques;
        // 要去重 先排序
        Arrays.sort(nums);
        boolean[] used = new boolean[nums.length];
        dfs(nums, used, permuteUniques, new ArrayList<>());
        return permuteUniques;
    }

    /**
     * 
     * @param nums
     *            原数组
     * @param used
     *            标记数字使用状态
     * @param permuteUniques
     *            目标集合
     * @param permuteUnique
     *            单个集合
     */
    private void dfs(int[] nums, boolean[] used, List<List<Integer>> permuteUniques, List<Integer> permuteUnique) {
        if (permuteUnique.size() == nums.length) {
            permuteUniques.add(new ArrayList<>(permuteUnique));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) // 已经标记过的略过
                continue;
            // 数字有重复的 说明刚释放的值与该值一样 略过
            if (i > 0 && nums[i - 1] == nums[i] && !used[i - 1])
                continue;
            used[i] = true; // 标记使用
            permuteUnique.add(nums[i]); // 该数字加入集合
            // 重复操作 选择剩余数字
            dfs(nums, used, permuteUniques, permuteUnique);
            // 当出栈时将最后一个数从集合中删除 同时该数恢复未使用状态 继续操作
            used[i] = false;
            permuteUnique.remove(permuteUnique.size() - 1);
        }

    }
```

由于要去重，所以先将数组排序。与第一版本的大致思路一样，遍历数组将未标记的值插入。

```
if (i > 0 && nums[i - 1] == nums[i] && !used[i - 1]) // 数字有重复的 说明刚释放的值与该值一样 略过
    continue;
```
这行代码至关重要，与出栈时候的 `used[i] = false;` 相对应，实现了重复操作不执行的功能。

同样，`permuteUniques.add(new ArrayList<>(permuteUnique));` 也是需要 new 一个新对象塞入。