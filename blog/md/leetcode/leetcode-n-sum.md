---
title: LeetCode 之 n 个数之和（Sum n）
date: 2018-11-19 16:52:20
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

LeetCode 中有好几道题是求数字之和的，有 `Sum 2`、`Sum 3` 和 `Sum 4` 等。求和这种情况在我们实际开发中也是经常会遇到的，在这不妨拿出来我们把这归并到一起来说说。

无非就是数组中几个数字求和比较是否为目标值。且大多结果中是不能有重复的值。

大致我说下这个题意：
```
给定一个包含 m 个整数的数组 nums 和一个目标值 target，判断 nums 中是否存在 n 个元素，使得这 n 个元素相加的值与 target 相等？找出所有满足条件且不重复的组。

注意：
答案中不可以包含重复的组。

示例：
给定数组 nums = [1, 0, -1, 0, -2, 2]， target = 0，n = 4

满足要求的四元组集合为：
[ [-1, 0, 0, 1],
[-2, -1, 1, 2],
[-2, 0, 0, 2] ]
```

两数之和好办，循环遍历去除重复结果。三数之和也类似。

那要是许多许多数之和呢，遍历就不好使了，那就要采用 BFS 或者 DFS。

`答案中不可以包含重复的组`从这我们知道，首先，我们需要将数组排个序。

由大化小的思想，我们可以采用 DFS + 回溯来解决这一系列问题。

苦恼不已，别挠头了，我们还是需要好好爱护自己的青青草地！

旨在锻炼逻辑思维，思想对了很重要，请上代码：
```
    /**
     * 
     * @param nums
     *            排序后目标数组
     * @param target
     *            累加目标数值
     * @param k
     *            个数
     * @param index
     *            起始下标
     * @return
     */
    private ArrayList<List<Integer>> kSum(int[] nums, int target, int k, int index) {

        ArrayList<List<Integer>> res = new ArrayList<>();

        if (index >= len)
            return res;
        if (k == 2) { // 两数取和
            int i = index, j = len - 1;
            while (i < j) {
                // 满足条件塞入集合
                if (target - nums[i] == nums[j]) {
                    List<Integer> temp = new ArrayList<>();
                    temp.add(nums[i]);
                    temp.add(nums[j]);
                    res.add(temp);
                    while (i < j && nums[i] == nums[i + 1]) // 跳过重复数值
                        i++;
                    while (i < j && nums[j] == nums[j - 1]) // 跳过重复数值
                        j--;
                    i++;
                    j--;
                } else if (target - nums[i] > nums[j])
                    i++;
                else
                    j--;
            }
        } else {
            for (int i = index; i < len - k + 1; i++) {
                // 调用递归 DFS
                ArrayList<List<Integer>> temp = kSum(nums, target - nums[i], k - 1, i + 1);
                // 若是有值返回则将该数塞入，无则不进行任何操作
                if (temp != null && temp.size() > 0) {
                    for (List<Integer> list : temp) {
                        list.add(nums[i]); // 将满足条件数值塞入
                    }
                    res.addAll(temp);
                }
                while (i < len - 1 && nums[i] == nums[i + 1]) // 跳过重复数值
                    i++;
            }
        }
        return res;
    }
```

**注意：这里 `len` 定义的是个全局变量，初始值为 0**

若 n 为 4，那程序应该是这样的：
```
int len = 0;
public List<List<Integer>> fourSum(int[] nums, int target) {
        len = nums.length;
        Arrays.sort(nums); // 先排序
        return kSum(nums, target, 4, 0); // 递归调用

    }
```

这下只要满足 `n > 1` 条件的都可以套用这个方法了。

要注意栈的深度。时间换空间。