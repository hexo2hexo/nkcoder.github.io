---
layout: post
title: 算法练习--从链表中删除一个节点
date: 2014-05-16 22:31:03 +0800
comments: true
tags: algorithm
categories: Algorithm
---

## 1. 问题描述：

> 给定一个链表和其中一个节点，删除该节点；

<!-- more -->

## 2. 思路：

> 思路：
节点的差异体现在节点对象的内容不同。要删除当前节点，可以将当前节点与下一节点的值互换，然后删除下一个节点即可。
需要注意的是，如果要删除的节点是最后一个节点，没有下一个节点，此时需要从头遍历了。如果要删除的节点不是最后一
个节点，复杂度为O(1)，否则复杂度为O(n)，平均复杂度为O(1)。

## 3. Java参考代码

    /**
     * 从链表中删除某一个节点
     * @param head  链表的头节点
     * @param toDelete  要删除的节点
     * @return 头节点
     */
	public static ListNode delete(ListNode head, ListNode toDelete) {
		// param error
		if (head == null || toDelete == null) {
			return head;
		}
		// 最后一个节点
		if (toDelete.next == null) {
			// 也是头节点
			if (head == toDelete) {
                return null;
			}
			// 遍历查找前一个节点
			ListNode node = head;
			while (node.next != toDelete) {
				node = node.next;
			}
			node.next = node.next.next;
			return head;
		}

        // 不是最后一个节点，将下一个节点的值覆盖当前节点的值，删除下一个节点
		toDelete.value = toDelete.next.value;
		toDelete.next = toDelete.next.next;
		return head;
	}
