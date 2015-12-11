---
layout: post
title: 算法练习--扑克牌构成顺子
date: 2014-06-04 23:20:29 +0800
comments: true
tags: algorithm
categories: Algorithm
---

## 1. 问题描述：

> 从扑克牌里抽出任意5张牌，其中J, Q, K分表表示11, 12, 13，A表示1，大小王可以表示任意点数。求该5张牌是否可以构成顺子。

<!-- more -->

## 2. 思路：

> 思路：
> 思路一：首先求出抽出的5张牌中，大小王的数量，然后将其它的牌按点数排序，计算它们之间的空隙，
如果空隙数小于王牌的数量，则表示可以构成顺子；否则，不能构成顺子。复杂度O(n)。
> 思路二：特殊牌做特殊处理，将大小王的点数当做0，求出最大点数和最小点数（非0），如果最大点数
和最小点数之间的差值小于5（抽出的牌的总数量），则表示可以构成顺子，否则不能；注意：为了方便
排重，因为牌的数量小，可以先排序，然后排重，最后比较。复杂度O(n)。

## 3. Java参考代码

    /**
     * 检查抽出的n张牌是否可以构成顺子
     * @param cards 表示抽出的牌的数组
     * @param length    抽出的牌的数量
     * @return  是否可以构成顺子
     */
    public boolean checkStraightPoker(int[] cards, int length) {
        // 桶排序
        int[] allCards = new int[ALL_CARDS_SIZE];
        for (int i = 0; i < length; i++) {
            allCards[cards[i]]++;
        }

        // 牌的排重
        for (int i = 1; i < ALL_CARDS_SIZE; i++) {
            if (allCards[i] > 1) {
                logger.error("identical cards exists.");
                return false;
            }
        }

        // 查找最小索引（非0）
        int firstIndex = 1;
        while (0 == allCards[firstIndex]) {
            firstIndex ++;
        }
        // 查找最大索引（非0）
        int lastIndex = ALL_CARDS_SIZE - 1;
        while (0 == allCards[lastIndex]) {
            lastIndex --;
        }
        // 构成顺子的条件
        if ((lastIndex - firstIndex + 1) <= length) {
            return true;
        }
        return false;
    }
