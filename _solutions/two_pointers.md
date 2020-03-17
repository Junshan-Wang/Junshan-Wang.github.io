---
title: "Two Pointers"
collection: solutions
permalink: /solutions/two_pointers/
author_profile: true
---

### 777. Swap Adjacent in LR String

In a string composed of 'L', 'R', and 'X' characters, like "RXXLRXRXL", a move consists of either replacing one occurrence of "XL" with "LX", or replacing one occurrence of "RX" with "XR". Given the starting string start and the ending string end, return True if and only if there exists a sequence of moves to transform one string to the other.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/swap-adjacent-in-lr-string

题目有些歧义。

其实这道题的意思是，`L`只能向左移动，`R`只能向右移动。且注意到`start`和`end`中的`L`和`R`不能交错。

所以用两个指针`p1`和`p2`，分别指向`start`和`end`，遍历每一个`L`或`R`，要求：

* 两个指针指向的字母必须相同
* 如果当前指向的是`L`，则`p1>=p2`
* 如果当前指向的是`R`，则`p1<=p1`

如果遍历完整个字符串都满足上述要求，则返回true，否则返回false。