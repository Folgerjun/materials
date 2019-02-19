---
title: LeetCode 之反转链表（Reverse Linked List）
date: 2019-2-19 10:22:56
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

## 前言
反转链表也是常见的面试算法题了。

何为链表？

    链表（Linked list）是一种常见的基础数据结构，是一种线性表，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的指针(Pointer)。由于不必须按顺序存储，链表在插入的时候可以达到O(1)的复杂度，比另一种线性表顺序表快得多，但是查找一个节点或者访问特定编号的节点则需要O(n)的时间，而顺序表相应的时间复杂度分别是O(logn)和O(1)。


## 正文
我们先来看下题目描述：
```
反转一个单链表。

示例:
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

进阶:
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？
```

小时候都玩过玩具蛇吧，那种可以一节一节拼接的。

我们可以想象下现在面前就有这么一条“蛇”，我们试着把它重新组装一遍，我们简单地 **边拆边装**。

先把它尾巴拆了放一边，再接着拆它的倒数第二块同时把它安装到拆下来的尾巴那，以此下去……

到最后把“蛇头”也给装好，就完事了。

这道题的解题思路也就是这样，**边拆边装**。

`你可以迭代或递归地反转链表。你能否用两种方法解决这道题？`

贴出两种解决方案代码：
```
public class ReverseLinkedList {

    public class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }
    }

    /**
     * 递归
     * 
     * @param head
     * @return
     */
    public ListNode reverseList(ListNode head) {
        ListNode reverseList = null;
        return helper(head, reverseList);
    }

    private ListNode helper(ListNode head, ListNode reverseList) {
        if (head == null) // 反转结束
            return reverseList;
        // 节点指针变换
        ListNode tempNode = head.next;
        head.next = reverseList;
        return helper(tempNode, head);
    }

    /**
     * 迭代
     * 
     * @param head
     * @return
     */
    public ListNode reverseList1(ListNode head) {

        ListNode newHead = null;
        while (head != null) { // 遍历
            ListNode next = head.next;
            head.next = newHead;
            newHead = head;
            head = next;
        }
        return newHead;
    }
}
```
两者都是先用一个空链表然后再进行一步步得组装。

就是指针指来指去，有点绕，借助实物理解起来会容易很多。

还有一个进阶版本的`反转链表 II`，看题：
```
反转从位置 m 到 n 的链表。请使用一趟扫描完成反转。

说明:
1 ≤ m ≤ n ≤ 链表长度。

示例:

输入: 1->2->3->4->5->NULL, m = 2, n = 4
输出: 1->4->3->2->5->NULL
```
区别就是这里不是反转所有的结点了，只需要反转指定位置之间的结点了，重点就是确认反转的指针位置。然后反转的操作还是与上面一样。

代码如下：
```
public class ReverseLinkedListII {

    public class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }
    }

    public ListNode reverseBetween(ListNode head, int m, int n) {
        if (head == null)
            return null;
        // 新建一个节点并指向 head
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode pre = dummy;
        // pre 为需要反转的前节点
        for (int i = 0; i < m - 1; i++)
            pre = pre.next;

        // 需要反转的节点 双指针
        ListNode start = pre.next;
        ListNode then = start.next;

        // 反转节点
        for (int i = 0; i < n - m; i++) {
            start.next = then.next;
            then.next = pre.next;
            pre.next = then;
            then = start.next;
        }
        return dummy.next;
    }
}
```
注释齐全，一目了然。