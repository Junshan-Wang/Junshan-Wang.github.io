---
title: "#45. Jump Game II"
collection: solutions
permalink: /solutions/jump_game_2/
author_profile: true
---

Given an array of non-negative integers, you are initially positioned at the first index of the array.

Each element in the array represents your maximum jump length at that position.

Determine if you are able to reach the last index.

Example 1:
```
Input: [2,3,1,1,4]
Output: true
```


Example 2:
```
Input: [3,2,1,0,4]
Output: false
```

From LeetCode

---

这是Jump Game的第一个版本，比较简单。

我们知道，从初始位置开始的前k个位置都能跳到，而k个以后则都不能跳到。用一个变量保存当前能够跳到的最远距离，遍历数组更新最远距离，最后只需要判断最远距离是否大于`num.size()-1`即可。

---

Given an array of non-negative integers, you are initially positioned at the first index of the array.

Each element in the array represents your maximum jump length at that position.

Your goal is to reach the last index in the minimum number of jumps.

Example:

```
Input: [2,3,1,1,4]
Output: 2
```

Note:

You can assume that you can always reach the last index.

From LeetCode

---

这一题用的是动态规划/广度搜索的思路，需要达到`O(n)`的时间复杂度。我们知道，在保证能到达最后一个位置的前提下，到达每一个位置所需要的跳数是单调递增的，即可以认为是分段连续的。

因此用两个变量，一个保存当前步数能到达的最远距离，另一个保存下一步数能到达的最远距离。遍历数组，更新下一步数能到达的最远距离，而如果当前位置超过了当前步数所能到达的最远距离，则表示进入了下一跳的分段，则用下一段最远距离更新当前最远距离。

```
class Solution {
public:
    int jump(vector<int>& nums) {
        int current_max_pos = 0, last_max_pos = 0, jump = 0;
        for (int i = 0; i < nums.size(); ++i) {
            if (current_max_pos >= nums.size() - 1)
                break;
            if (i > current_max_pos) {
                current_max_pos = last_max_pos;
                jump++;
            }
            last_max_pos = max(last_max_pos, i + nums[i]);
        }
        return jump;
    }
};
```
