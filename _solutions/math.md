---
title: "Math"
collection: solutions
permalink: /solutions/math/
author_profile: true
---

## 最小距离

最简单的一维情况：  
给定数组a，找到x，使得数组中每个数到x的距离最小，即`|a[0] - x| + |a[1] - x| + ... + |a[n-1] - x|`最小。  
*当x为a的中位数时使得距离最小。当a的大小为奇数时x为该中位数；当a的大小为偶数时x为两个中位数中间的任何一个数即可。*  
查找中位数可以先进行排序然后直接得到，时间复杂度为O(nlogn)。也可以通过快速选择算法（快排的变体，选择第k大数），时间复杂度为O(n)。

### 462. Minimum Moves to Equal Array Elements II

Given a non-empty integer array, find the minimum number of moves required to make all array elements equal, where a move is incrementing a selected element by 1 or decrementing a selected element by 1.

You may assume the array's length is at most 10,000.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/minimum-moves-to-equal-array-elements-ii

```
class Solution {
public:
    int minMoves2(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        int med = nums[nums.size() / 2], res = 0;
        for (int num : nums) res += abs(num - med);
        return res;
    }
};
```

### 296. Best Meeting Point

A group of two or more people wants to meet and minimize the total travel distance. You are given a 2D grid of values 0 or 1, where each 1 marks the home of someone in the group. The distance is calculated using Manhattan Distance, where distance(p1, p2) = |p2.x - p1.x| + |p2.y - p1.y|.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/best-meeting-point

二维情况下的曼哈顿距离，其实就是分成两个方向，分别求最小距离。

```
class Solution {
public:
    int minTotalDistance(vector<vector<int>>& grid) {
        vector<int> xs, ys;
        for (int i = 0; i < grid.size(); ++i) {
            for (int j = 0; j < grid[0].size(); ++j) {
                if (grid[i][j] == 1) {
                    xs.push_back(i);
                    ys.push_back(j);
                }
            }
        }
        sort(xs.begin(), xs.end());
        sort(ys.begin(), ys.end());
        int med_x = xs[xs.size() / 2], med_y = ys[ys.size() / 2];
        int res = 0;
        for (int i = 0; i < xs.size(); ++i) {
            res += abs(xs[i] - med_x);
            res += abs(ys[i] - med_y);
        }
        return res;
    }
};
```
