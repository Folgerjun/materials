---
title: 二分搜索之搜索数组中目标元素的首尾下标
date: 2018-10-15 17:02:16
categories: [开发,算法]
tags: [算法,Java,二分搜索]
---

今天总结一下二分搜索。**假设这里的数组已经是升序排序好了的。**

我们知道二分搜索的效率很高，它充分利用了元素间的次序关系，采用分治策略，可在最坏的情况下用 O(log n) 完成搜索任务。它的基本思想：将 n 个元素分成个数大致相同的两半，取 a[n/2] 与需要查找的目标值 x 作比较，如果 x=a[n/2] 则找到 x，算法运算终止。详情可跳转[百度百科](https://baike.baidu.com/item/%E4%BA%8C%E5%88%86%E6%90%9C%E7%B4%A2%E6%B3%95/6230633)。

- 我们通常最基本的二分搜索是这样实现的(Java):
```
    /**
     * 二分搜索
     * 
     * @param arr
     *            已升序排序数组
     * @param key
     *            目标查找值
     * @return
     */
    public static int commonBinarySearch(int[] arr, int key) {
        int low = 0;
        int high = arr.length - 1;
        int middle = 0; // 定义middle

        if (key < arr[low] || key > arr[high] || low > high) {
            return -1;
        }

        while (low <= high) {
            middle = (low + high) / 2;
            if (arr[middle] > key) {
                // 比关键字大则关键字在左区域
                high = middle - 1;
            } else if (arr[middle] < key) {
                // 比关键字小则关键字在右区域
                low = middle + 1;
            } else {
                return middle;
            }
        }

        return -1; // 未找到结果，返回-1
    }
```
若是数组中存在搜索目标元素，则只要查找到任意一个便会返回该值；若是没有找到即返回 -1。

- 来看下一种，只返回目标元素第一次出现的位置下标(伪代码):
```
l = -1; u = n
while l+1 != u
    m = (l + u) / 2
    if x[m] < t
        l = m
    else
        u = m
p = u
if p >= n || x[p] != t
    p = -1
```
n 为数组的长度，p 就是最终我们需要的下标。若是 while 循环出来的最终结果 `u >= n` （其实最大也只会等于 n）或者 `x[u] != t`（t 为我们的目标元素），那么也就是无结果，返回 -1。

- 我们再来看最后一个 function(该方法参数不同使得结果可以返回元素第一个出现的下标或者最后一个下标，也是 Java):
```
    /**
     * 
     * @param nums
     *            已经升序排序好的数组
     * @param target
     *            搜索目标元素
     * @param left
     *            是否是查找第一个元素下标。true:查找目标元素第一个出现下标, false:查找目标元素最后一个出现下标
     * @return
     */
    private int extremeInsertionIndex(int[] nums, int target, boolean left) {
        int lo = 0;
        int hi = nums.length;

        while (lo < hi) {
            int mid = (lo + hi) / 2;
            if (nums[mid] > target || (left && target == nums[mid])) {
                hi = mid;
            } else {
                lo = mid + 1;
            }
        }
        return lo;
    }
```
目标元素出现的第一个下标：`int leftIdx = extremeInsertionIndex(nums, target, true);`<br>
目标元素出现的最后一个下标：`int rightIdx = extremeInsertionIndex(nums, target, false) - 1;`这里需要减一，需要注意。

---

**总结一下**：二分搜索只需要注意它的边界值，原先的数组下标范围是 [0..n-1]，当你用 `lo = 0, hi = n-1` 去运行时，对应条件满足情况下值的赋值应该是 `lo = mid + 1, hi = mid - 1`；而若是用 `lo = -1, hi = n` 去作为条件运行时，对应条件满足情况下值的赋值就应该为 `lo = mid, hi = mid`，因为它两个值都是作为边界值。[0, n]、[-1, n-1]也是如此。

---
相关链接：

- [编程珠玑（第二版）第四章习题](https://github.com/Folgerjun/Programming-Pearls/blob/master/Chapter-Four.md#2)
- [LeetCode - Find First and Last Position of Element in Sorted Array](https://github.com/Folgerjun/leetcode-cn/blob/master/src/com/leetcode_cn/medium/FindFirstAndLastPositionOfElementInSortedArray.java)