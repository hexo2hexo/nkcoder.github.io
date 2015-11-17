---
layout: post
title: 算法练习--顺时针打印矩阵
date: 2014-05-12 22:41:10 +0800
comments: true
tags: algorithm
categories: Algorithm
---

## 1. 问题描述：

> 输入一个矩阵（二维数组），按照顺时针的方向打印矩阵中的元素；比如，输入：

	1   2   3   4   5
	6   7   8   9   10
	11  12  13  14  15
	16  17  18  19  20

> 输出：

	1 2 3 4 5 10 15 20 19 18 17 16 11 6 7 8 9 14 13 12

## 2. 思路：

> 可以将顺时针打印分成四个步骤，从左到右的一行，从上到下的一列，从右到左的一行，从下到上的一列。打印时有两个条件限制：一是不能出界，二是不能重复打印。第一个条件，可以通过矩阵的行列值进行限制，第二个条件，可以设置一个与矩阵同样结构的标志数组，用于标记矩阵的每一个元素是否被遍历过。

> 这里有一个常用的技巧，即试探法，先试探下一个元素是否可遍历，如果是，则遍历，否则终止本轮遍历。如果不是使用试探法，直接通过边界值的限制，循环遍历，当越界时，需要回退。
+ 时间复杂度O(m*n)，空间复杂度O(m*n)，其中m、n分别表示矩阵的行数和列数。

## 3. Java参考代码

	/**
	 * 顺时针打印矩阵，分为四步：左->右，上->下，右->左，下->上，每一步
     * 使用试探法，通过矩阵的行数、列数作为限制，遍历矩阵。
	 * @param matrix    矩阵
	 * @param rows  矩阵的行数
	 * @param cols  矩阵的列数
	 */
	public static void print(int[][] matrix, int rows, int cols) {
		// 标志数组初始化
		boolean[][] visited = new boolean[rows][cols];
		for (int i = 0; i < rows; i ++) {
			for (int j = 0; j < cols; j++) {
				visited[i][j] = false;
			}
		}

		int i = 0;
		int j = -1;
		int k = 0;
		// 遍历矩阵
		while (k++ < rows * cols) {
			// 左到右的一行
			while (j + 1 < cols && !visited[i][j+1]) {
				logger.info("{} ", matrix[i][++j]);
				visited[i][j] = true;
			}
			// 上到下的一列
			while (i + 1 < rows && !visited[i+1][j]) {
				logger.info("{} ", matrix[++i][j]);
				visited[i][j] = true;
			}

			// 右到左的一行
			while (j - 1 >= 0 && !visited[i][j-1]) {
				logger.info("{} ", matrix[i][--j]);
				visited[i][j] = true;
			}
            // 下到上的一列
			while (i - 1 >= 0 && !visited[i-1][j]) {
				logger.info("{} ", matrix[--i][j]);
				visited[i][j] = true;
			}
		}
	}
