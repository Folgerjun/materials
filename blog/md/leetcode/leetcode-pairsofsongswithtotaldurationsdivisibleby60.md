---
title: LeetCode 之总持续时间可被 60 整除的歌曲（Pairs of Songs With Total Durations Divisible by 60）
date: 2019-4-19 16:59:27
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

题目虽然有点长，不过可以化简为同一个类型的，就是两两配对其和是某个数的倍数。

原题描述如下：
```
在歌曲列表中，第 i 首歌曲的持续时间为 time[i] 秒。

返回其总持续时间（以秒为单位）可被 60 整除的歌曲对的数量。形式上，我们希望索引的数字  i < j 且有 (time[i] + time[j]) % 60 == 0。

示例 1：
输入：[30,20,150,100,40]
输出：3
解释：这三对的总持续时间可被 60 整数：
(time[0] = 30, time[2] = 150): 总持续时间 180
(time[1] = 20, time[3] = 100): 总持续时间 120
(time[1] = 20, time[4] = 40): 总持续时间 60

示例 2：
输入：[60,60,60]
输出：3
解释：所有三对的总持续时间都是 120，可以被 60 整数。
 
提示：

1 <= time.length <= 60000
1 <= time[i] <= 500
```

初看题目，脑子都不要动，我相信大多人也是和我一样，两个 for 搞定：

```
    /**
     * 超时
     * 
     * @param time
     * @return
     */
    public int numPairsDivisibleBy60(int[] time) {
        int result = 0;
        for (int i = 0; i < time.length - 1; i++) {
            for (int j = i + 1; j < time.length; j++) {
                int value = (time[i] + time[j]) % 60;
                if (value == 0)
                    result++;
            }
        }
        return result;
    }
```

果然不意外，提交显示超时了。

我们静下心来想想，两数之和是某个数的倍数。既然这样循环查找过于繁琐，那有什么方式可以快速精准查找呢？

哐...当然有了。

首先我们会想到数组，还有键值对形式存储的 Map

这里我使用的是数组，那要怎么使用它呢，再细想。

数组精准查找，那得要根据下标，两数之和为某数，自然，那就将数字作为下标存储到对应块中，值就是其对应的个数。

思路有了，代码如行云流水，请看：
```
    public int numPairsDivisibleBy601(int[] time) {
        // 每个元素取60余
        time = Arrays.stream(time).map(x -> x % 60).toArray();
        int[] arrCount = new int[60];
        // 统计对应下标数
        Arrays.stream(time).forEach(x -> {
            arrCount[x]++;
        });
        // 余数是 0 或 30 的两两配对
        int result = qiuhe(arrCount[0]) + qiuhe(arrCount[30]);
        // 其余互相配对
        for (int i = 1; i < 30; i++) {
            int count1 = arrCount[i];
            int count2 = arrCount[60 - i];
            result += count1 * count2;
        }
        return result;
    }

    /**
     * 求两两配对数
     * 
     * @param num
     * @return
     */
    private int qiuhe(int num) {
        if (num < 2)
            return 0;
        // (num-1)!
        return num * (num - 1) / 2;
    }
```

很好理解，不过速度还是慢，通过阅读其它人的代码后发现了另一种 O(n) 解法，cool~

```
    public int numPairsDivisibleBy602(int[] time) {
        int result = 0;
        int[] arrCount = new int[60];
        for (int t : time) {
            // 对应的下标
            int index = t == 0 ? 0 : 60 - t % 60;
            // 与已经统计的数进行匹配 可防止两两重复匹配
            result += arrCount[index];
            // 对应数字加一
            arrCount[t % 60]++;
        }
        return result;
    }
```

一个 for 完美解决问题，想必这就是算法的魅力吧。