---
title: LeetCode 之回文数（Palindrome Number）
date: 2018-11-19 15:52:35
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

回文数想必大家都不陌生吧。什么？你居然不知道何谓“回文数”？

回文数：“回文”是指正读反读都能读通的句子，它是古今中外都有的一种修辞方式和文字游戏，如“我为人人，人人为我”等。在数学中也有这样一类数字有这样的特征，成为回文数（palindrome number）。

OK，来看题：
```
判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。
  
示例 1:
输入: 121 输出: true
 
示例 2:
输入: -121 输出: false 
解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。

示例 3:
输入: 10 输出: false  
解释: 从右向左读, 为 01 。因此它不是一个回文数。

进阶:
你能不将整数转为字符串来解决这个问题吗？
```

这道题看到第一眼就能想到`字符串反转`可以解决。

当然，StringBuilder 就可以实现，如下：
```
    /**
     * 通过字符串反转判断
     * 
     * @param x
     * @return
     */
    public static boolean isPalindrome(int x) {

        if (x == 0)
            return true;

        String s = String.valueOf(x);

        StringBuilder sb1 = new StringBuilder(s);
        sb1.reverse();

        return sb1.toString().equals(s) ? true : false;

    }
```

代码很简短，功能也能实现。但是这并不是我们所追求的！来看进阶:`你能不将整数转为字符串来解决这个问题吗？`

不转成字符串来解决这个问题，挠一挠头。当然，这肯定难不倒聪明才智的你！

不就是反着来吗，问题不大，请看：

```
public static boolean isPalindrome1(int x) {

        if (x == 0 || (x > 0 && x < 10))
            return true;
        if (x < 0 || (x % 10 == 0 && x != 0))
            return false;

        int result = 0, num = x;

        while (num != 0) {
            int i = num % 10;
            result = result * 10 + i;
            num /= 10;
            System.out.println("num :" + num + " result :" + result);
        }

        return result == x ? true : false;

    }
```

**取余，上个余数乘 10 再加上这个余数，数字每次除 10 取整。**

```
if (x == 0 || (x > 0 && x < 10))
            return true;
```
当 `x == 0` 或者 `x > 0 && x < 10` 时，该数肯定是个回文数，这不用多说。

```
if (x < 0 || (x % 10 == 0 && x != 0))
            return false;
```
当 `x < 0` 时，该数肯定不是个回文数，这也不用多说。`x % 10 == 0 && x != 0` 这个条件的意思是 x 的末数是个 0，也就是它是个 10 的倍数，同时 x 不是 0。这也可以想象，满足这个条件的也肯定不是个回文数，因为 0 开头的只能是 0。

乍一看，这么写稳稳的，堪称完美。满意的端起键盘旁的红枣枸杞水，美滋滋嘬了一口。

其实上述代码还可以优化，运行时间能减少一半。滚烫的红枣枸杞水烫着了舌头，忙用口水润润。

```
    /**
     * 反转一半数字进行比较 比上面方法速度快一倍
     * 
     * @param x
     * @return
     */
    public static boolean isPalindrome2(int x) {
        // 特殊情况：
        // 如上所述，当 x < 0 时，x 不是回文数。
        // 同样地，如果数字的最后一位是 0，为了使该数字为回文，
        // 则其第一位数字也应该是 0
        // 只有 0 满足这一属性
        if (x < 0 || (x % 10 == 0 && x != 0)) {
            return false;
        }
        int revertedNumber = 0;
        while (x > revertedNumber) {
            revertedNumber = revertedNumber * 10 + x % 10;
            x /= 10;
        }
        // 当数字长度为奇数时，我们可以通过 revertedNumber/10 去除处于中位的数字。
        // 例如，当输入为 12321 时，在 while 循环的末尾我们可以得到 x = 12，revertedNumber = 123，
        // 由于处于中位的数字不影响回文（它总是与自己相等），所以我们可以简单地将其去除。
        return x == revertedNumber || x == revertedNumber / 10;
    }
```

妙哉妙哉，满意地捋着下巴小胡子，眯眼色道。