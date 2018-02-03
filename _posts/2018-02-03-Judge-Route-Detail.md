---
layout:     post
title:      657. Judge Route Circle
subtitle:   LeetCode刷题笔记
date:       2018-02-03
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-657. Judge Route Circle 题目


# 题目 #

Initially, there is a Robot at position (0, 0). Given a sequence of its moves, judge if this robot makes a circle, which means it moves back to **the original place**.

The move sequence is represented by a string. And each move is represent by a character. The valid robot moves are R (Right), L (Left), U (Up) and D (down). The output should be true or false representing whether the robot makes a circle.

# 举例 #

	Example1:
	
	Input: "UD"
	Output: true

	Example2:

	Input: "LL"
	Output: false

# 问题思路 #

将问题转化为二维坐标移动问题：

- R(Right)向右移动 x++
- L(Left) 向左移动 x--
- U(Up)   向上移动 y++
- D(Down) 向下移动 y--

最后判断 x==0 && y == 0 即为有环。

## C语言实现 ##

	bool judgeCircle(char* moves) {
	    int i = 0;
	    int x = 0;
	    int y = 0;
	    while(moves[i] != '\0'){
	        if(moves[i] == 'R')
	            x++;
	        else if(moves[i] == 'L')
	            x--;
	            else if(moves[i] == 'U')
	                y++;
	            else
	                y--;
	        i++;
	    }
	    if(x==0 && y==0)
	        return true;
	    else
	        return false;
	}