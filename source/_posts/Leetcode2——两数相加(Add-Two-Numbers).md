---
title: Leetcode2——两数相加(Add Two Numbers)
date: 2018-09-19 21:45:06
tags: 
  - LeetCode
categories: 
  - LeetCode
---

### 题目：
给定两个非空链表来表示两个非负整数。位数按照逆序方式存储，它们的每个节点只存储单个数字。将两数相加返回一个新的链表。
你可以假设除了数字 0 之外，这两个数字都不会以零开头。


<!--more-->

示例：
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
### 思路分析：
大致思路就是依次从两条链表对应位取出节点进行相加，然后将结果存储到新的结果链表中。在这个过程中，首先要考虑进位的问题，所以需要一个变量carry来记录进位值，当然无进位时carry为0，有进位时为1；其次我们需要考虑两个链表存在对应位为空的情况。若某一链表对应位为空，应将其值定为0。若两条链表的对应位均为空，则运算终止。注意最后一位的进位。
### 解法：

    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
    }