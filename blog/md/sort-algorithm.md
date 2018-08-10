---
title: 常用排序算法
date: 2018-8-10 09:21:51
categories: [开发,总结]
tags: [Java,算法]
---

在这里做个记录，将常用的三大排序算法列出来，方便查看复习，算法主要是理清思路，最近在刷leetcode发现自己脑子是有点笨的。 For Java.

---

## 冒泡排序
冒泡是我觉得最简单的一个排序算法了，也是我记的最早的一个排序算法。

**冒泡，顾名思义，泡泡往上冒，也就是每次都将最大值放在末尾，剩余值继续冒泡。**

其时间复杂度为O（n²）。详解：[冒泡排序](https://baike.baidu.com/item/%E5%86%92%E6%B3%A1%E6%8E%92%E5%BA%8F/4602306?fr=aladdin)

`参考代码：`
```
    /**
     * 冒泡排序
     * 
     * @param arr
     * @return
     */
    public static int[] MPSort(int[] arr) {
        // 外循环一次就将最大的值放最后
        for (int i = 1; i < arr.length; i++) {
            // 内循环剩余值比较 找出最大值
            for (int j = 0; j < arr.length - i; j++) {
                // 比较 满足条件互换
                if (arr[j] > arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
        return arr;

    }
```

## 插入排序

插入排序也是比较好理解的一个排序算法，别看这名字挺粗鲁的，其实它还就那么回事。

**插入排序，挨个遍历，发现比前数值还要小的数就把该数前移比较并换值，直到满足的位置为止。**

是挺粗鲁的一个算法，不过这很直接，我喜欢。其时间复杂度为O（n²）。详解：[插入排序](https://baike.baidu.com/item/%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F)

`参考代码：`
```
    /**
     * 插入排序
     * 
     * @param arr
     * @return
     */
    public static int[] insertSort(int[] arr) {
        System.out.println(Arrays.toString(arr));
        // j = ++i 保证了i和j两个数一样
        for (int i = 0, j = i; i < arr.length - 1; j = ++i) {
            int num = arr[i + 1];
            // 有较小值就前移并换值
            while (arr[j] > num) {
                arr[j + 1] = arr[j];
                // 使下标递减去比较
                if (j-- == 0) {
                    break;
                }
            }
            // 较小值新赋值
            arr[j + 1] = num;

        }
        return arr;
    }
```

## 快速排序

快排就是比较经典了，快排是对冒泡的一种改进，由C. A. R. Hoare在1962年提出。

**它的基本思想是：通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。**

详解：[快速排序](https://baike.baidu.com/item/%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)

`参考代码：`
```
    /**
     * 快速排序
     * 
     * @param arr
     * @param lo
     * @param hi
     * @return
     */
    public static int[] quickSort(int[] arr, int lo, int hi) {
        if (lo >= hi)
            return arr;
        int num = partition(arr, lo, hi);
        quickSort(arr, lo, num - 1);
        quickSort(arr, num + 1, hi);
        return arr;
    }

    /**
     * 排序数组 返回基准值
     * 
     * @param arr
     * @param lo
     * @param hi
     * @return
     */
    public static int partition(int[] arr, int lo, int hi) {
        int num = arr[lo];
        if (lo >= hi)
            return 0;
        while (lo < hi) {
            while (hi > lo && arr[hi] >= num) {
                hi--;
            }
            arr[lo] = arr[hi];
            while (hi > lo && arr[lo] <= num) {
                lo++;
            }
            arr[hi] = arr[lo];
        }
        arr[hi] = num;
        return hi;
    }
```

---
 End.