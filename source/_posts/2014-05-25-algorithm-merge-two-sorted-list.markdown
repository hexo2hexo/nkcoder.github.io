---
layout: post
title: 算法练习--合并两个有序单链表
date: 2014-05-25 10:42:57 +0800
comments: true
tags: algorithm
categories: Algorithm
---

## 1. 问题描述：

> 给定两个单链表，都是递增有序的，将它们合并，使合并后的链表仍然有序。

<!-- more -->

## 2. 思路：

> 思路：
先比较两个单链表的第一个节点，值小的的节点作为合并后新链表的第一个节点；值小的节点的下一个
节点作为该单链表的新的头节点。相当于第一个节点已排好序，剩下的问题仍然是对两个有序链表进行
合并，则可以用递归的思路解决。

## 3. Java参考代码

    /**
	 * 合并两个有序单链表
	 * @param firstHead     第一个单链表的头节点
	 * @param secondHead    第二个单链表的头节点
	 * @return  合并后的链表的头节点
	 */
	public static ListNode merge(ListNode firstHead, ListNode secondHead) {
		// 退出条件
		if (firstHead == null) {
			return secondHead;
		}
		if (secondHead == null) {
			return firstHead;
		}

		// 比较两个单链表的第一个节点，值小的节点作为合并后的链表的节点
		ListNode currentHead = null;
		if (firstHead.value < secondHead.value) {
			currentHead = firstHead;
			currentHead.next = merge(firstHead.next, secondHead);
		} else {
			currentHead = secondHead;
			currentHead.next = merge(firstHead, secondHead.next);
		}
		return currentHead;
	}
