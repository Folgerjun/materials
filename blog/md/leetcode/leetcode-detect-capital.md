---
title: LeetCode 之检测大写字母（Detect Capital）
date: 2020-5-12 11:08:12
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

## 前言
这是一道难度为简单的题，确实常规去解决一点也不难，但当我看到有大神在评论区发解题思路的时候，脑壳就突然有一种被敲开往里灌清凉油的感觉……让我们来走进清凉世界！

## 正文
题目描述：
```
给定一个单词，你需要判断单词的大写使用是否正确。

我们定义，在以下情况时，单词的大写用法是正确的：

全部字母都是大写，比如"USA"。
单词中所有字母都不是大写，比如"leetcode"。
如果单词不只含有一个字母，只有首字母大写， 比如 "Google"。
否则，我们定义这个单词没有正确使用大写字母。

示例 1:
输入: "USA"
输出: True

示例 2:
输入: "FlaG"
输出: False
注意: 输入是由大写和小写拉丁字母组成的非空单词。
```

猛男一看，此题共有三种条件（单词字母全为大写、单词只有首字母大写、单词全小写），若满足其中之一则返回 true，否则都返回 false.

看完题目，大笔一挥，洋洋洒洒写下解题方法：
```
public boolean detectCapitalUse(String word) {

        // 全大写
        if (word.toUpperCase().equals(word))
            return true;
        // 只有首字母大写 | 全小写（只需要判断除了首字母其余是否都为小写，因为首字母不管大写小写都不影响都满足要求）
        if (word.substring(1).toLowerCase().equals(word.substring(1)))
            return true;
        return false;

    }
```

写完眉眼舒展嘴角微扬，好久没有写过这么舒坦的题了。三种条件，我还把两种归并一起写了，实乃天才是也！

写完提交，不出意外通过，不过一看这耗时 2ms，只击败了 30% . 我不服！遂翻其余人上传的解题思路，看到其中一条，突觉四肢麻木，天灵盖犹如被灌了风油精一般……

代码如下：
```
public boolean detectCapitalUse1(String word) {
        int l = word.length();
        // 统计大写字母出现次数
        int count = 0;
        for(int i = 0; i < l; i++) {
            // 是大写字母且数量是否一致
            if(word.charAt(i) < 91  && count++ < i) {
                return false;
            }
        }
        // count
        // 0:全是小写字母 1:只有首字母大写 l:都是大写
        return count <= 1 || count == l;
    }
```

**思路：从单词第一个字符开始遍历，遍历过程中统计出现的大写字母并与当前字符下标进行比较，即若是出现大写字母，则要么只有第一个是大写字母，要么全都是大写字母，其余情况都直接返回 false.**

```
System.out.println((int)'A'); // 65
System.out.println((int)'Z'); // 90
System.out.println((int)'a'); // 97
System.out.println((int)'z'); // 122
```

若是大写字母则满足 [65,90].

妙不可言，平时工具用多了难免第一反应就是去利用现成封装好的方法去做，脑子经常换一换会发现世界好清晰。