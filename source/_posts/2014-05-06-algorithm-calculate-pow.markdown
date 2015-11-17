---
layout: post
title: 算法练习--实现库函数pow(double base, int exp)
date: 2014-05-06 23:31:35 +0800
tags: algorithm
categories: Algorithm
---

## 1. 问题描述：

> 实现函数pow(double base, int exp)，即base的exp次方，不考虑大数问题，不能使用任何库函数。

## 2. 思路：

+ 2.1 首先考虑正常情况，计算b^e，最简单的方法是循环累乘，优点是简单直观，缺点是循环次数较多, 复杂度为(n)；

+ 2.2 为了减少循环的次数，可以采取“折半平方”的思路，
  - 如`x^6=(x^3)^2，x^7=(x^3)^2*x`，可以使用递归实现，优点是减少幂乘的次数，缺点是如果递归层次太深，可能发生栈溢出；复杂度为O(logn)。（递归的本质也是栈，我们可以自己用栈来实现）
  - 异常处理：当指数e为负数的时候，需要判断底数b的值是否为0，而b是double类型，因此涉及到浮点数与0的比较问题；

### 3. java代码：

	private final double THRESHOLD = 1E-6;

	/**
	 * 计算浮点数base的exponent次幂，不考虑大数（溢出）问题
	 * @param base
	 * @param exponent
	 * @return  -1表示异常，否则，返回值为结果值
	 */
	public double getPow(double base, int exponent) {
		if (isEqual(base, 0.0)) {
			if (exponent < 0) {
				logger.info("error input");
				return -1;
			}
			return 1.0;
		}

		// 计算exponent为正值时的结果
		int absExponent = exponent;
		if (exponent < 0) {
			absExponent = -exponent;
		}
		double result = positivePow(base, absExponent);

		// 如果exponent为负值，取其倒数
		if (exponent < 0) {
			result = 1.0 / result;
		}
		return result;
	}

	/**
	 * 通过平方的思路计算幂值（指数exponent为正值）：
	 *  如果指数为偶数, b^e = b^(e/2) * b^(e/2)
	 *  如果指数为奇数, b^e = b^(e/2) * b^(e/2) * b
	 *  这比通过循环求幂值的效率要好一些，可以通过递归来实现。
	 * @param base
	 * @param exponent
	 * @return
	 */
	private double positivePow(double base, int exponent) {
		if (exponent == 1) {
			return base;
		}
		double result = positivePow(base, exponent >> 1);
		result *= result;               // 先计算平方
		if ((exponent & 0x1) == 1) {   // 如果指数exponent为奇数，平方外应该再乘以base
			result *= base;
		}
		return result;
	}

	/**
	 * 比较两个浮点数是否相等
	 * @param d1
	 * @param d2
	 * @return 如果相等，返回true，否则返回false；
	 */
	private boolean isEqual(double d1, double d2) {
		if ((d1 - d2) < THRESHOLD && (d1 - d2) > -THRESHOLD) {
			return true;
		}
		return false;
	}
