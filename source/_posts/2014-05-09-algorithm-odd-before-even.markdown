---
layout: post
title: 算法练习--重排数组，使奇数位于偶数之前
date: 2014-05-09 23:40:24 +0800
comments: true
tags: algorithm
categories: Algorithm
---

## 1. 问题描述

> 给定数组，调整数组中元素的顺序，使得所有的奇数都位于数组的前部分，所有的偶数都位于数组的后部分。
输入：数组
输出：重排后的数组

## 2. 思路

- 使用两个指针，第一个指针指向数组首元素，第二个指针指向数组尾元素，在两个指针相遇前，第一个指针向后移动，直到遇到偶数，第二个指针向前移动，直到遇到奇数，然后两个指针指向的元素交换；重复上述步骤，直到两个指针相遇为止；复杂度O(n)。
- 这里，奇偶重排只是同类型问题中的一种，因此可以奇偶判断提取成一个方法，降低耦合度，可以提高代码的重用性。

## 3. java参考代码

	/**
     * 调整数组中元素的顺序，使所有的奇数位于数组的前部分，偶数位于数组的后部分；
     * 定义首尾指针，首指针前进，尾指针后退，当首指针遇到偶数且尾指针遇到奇数时，元素互换。
	 * @param data  输出数组
	 * @param length    数组的长度
	 */
	public static int[] adjust(int[] data, int length) {
		int first = 0;
		int last = length - 1;
		while (first < last) {
			// 首指针寻找第一个偶数
			while ((first < last) && (isOdd(data[first]))) {
				first++;
			}
			// 尾指针寻找第一个奇数
			while ((first < last) && (!isOdd(data[last]))) {
				last--;
			}
			// 首尾元素互换
			if (first < last) {
				int tmp = data[first];
				data[first++] = data[last];
				data[last++] = tmp;
			}
		}
        return data;
	}

    /**
     * 判断一个数是否为奇数
     * @param num
     * @return
     */
	public static boolean isOdd(int num) {
		if ((num & 0x01) == 1) {
			return true;
		}
		return false;
	}
