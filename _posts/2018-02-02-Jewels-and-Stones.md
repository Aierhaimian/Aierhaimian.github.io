---
layout:     post
title:      771. Jewels and Stones
subtitle:   LeetCode刷题笔记
date:       2018-02-02
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-561.Array Partition I 题目

# 题目

You're given strings **J** representing the types of stones that are jewels, and **S **representing the stones you have.  Each character in **S** is a type of stone you have.  You want to know how many of the stones you have are also jewels.

The letters in **J** are guaranteed distinct, and all characters in **J** and **S** are letters. Letters are case sensitive, so **"a"** is considered a different type of stone from **"A"**.

假设字符串**J**表示的是宝石的类型，**S**表示你拥有的石头。 **S**中的每个字符都是你所拥有的一种石头。 你想知道你有多少石头是宝石。

**J**中的字母保证不同，**J**和**S**中的所有字符都是字母。 字母是区分大小写的，所以“a”被认为是与“A”不同类型的石头。

#举例

	Example1:
	Input: J = "aA", S = "aAAbbbb"
	Output: 3

	Example2:
	Input: J = "z", S = "ZZ"
	Output: 0

#说明

1. **S**和**J**由字母组成，长度最多为50；
2. **J**中的字符是不同的。

#问题思路1

将**J**中字符元素与**S**中的字符元素一一比对，字符相同者对应的宝石数量加1。
时间复杂度：O(S*J)
空间复杂度：O(1)
#Python实现1

	class Solution:
    def numJewelsInStones(self, J, S):
        """
        :type J: str
        :type S: str
        :rtype: int
        """
        jewels = 0
        for i in range(len(J)):
            for j in range(len(S)):
                if J[i] == S[j]:
                    jewels = jewels + 1

        return jewels

#python实现2

	class Solution:
    def numJewelsInStones(self, J, S):
        """
        :type J: str
        :type S: str
        :rtype: int
        """
        jewels = 0
        stone = []
        for i in range(len(J)):
            stone.append(hash(J[i]))
        for i in range(len(S)):
                jewels = jewels + stone.count(hash(S[i]))
        return jewels


#问题思路2

通过将字符转换为ASCII码的方法，先将**J**中的每个字符当做key存起来，再遍历**S**并检查是否存在这些key。
时间复杂度：O(S+J)
空间复杂度：O(J)

#Python实现

	class Solution:
    def numJewelsInStones(self, J, S):
        """
        :type J: str
        :type S: str
        :rtype: int
        """
        jewels = 0
        stone = [None]*128
        for i in range(len(J)):
            stone[ord(J[i])] = 1
        for i in range(len(S)):
            if stone[ord(S[i])] == 1:
                jewels = jewels + 1
        return jewels