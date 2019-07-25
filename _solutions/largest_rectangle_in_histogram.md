---
title: "#84. Largest Rectangle in Histogram"
collection: solutions
permalink: /solutions/largest_rectangle_in_histogram/
author_profile: true
---

Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.

Example:
```
Input: [2,1,5,6,2,3]
Output: 10
```

from LeetCode

---

### Solution 1

这道题最暴力的方法是对柱状图中的每一个方块，计算包含它的最大面积是多少，只需要分别找到它左边和右边高度比它小的方第一个方块。但是这种方法的时间复杂度是`O(n^2)`。

用动态规划的方法可以降低时间复杂度（不确定是否能到`O(n)`）。当前遍历到第$i$个方块时，我们向左寻找第一个比它小的第一个方块，不需要逐个遍历，而是用数组`left_idx[j]`保存比$j$小的第一个方块，用到已经计算好的`left_idx[j]`来加速更新`left_idx[i]`。

```
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        vector<int> left_idx(heights.size(), 0);
        vector<int> right_idx(heights.size(), 0);
        int max_area = 0;
        for (int i = 0; i < heights.size(); ++i) {
            int j = i - 1;
            while (j >= 0 && heights[j] >= heights[i])
                j = left_idx[j];
            left_idx[i] = j;
        }
        for (int i = heights.size() - 1; i >= 0; --i) {
            int j = i + 1;
            while (j <= heights.size() - 1 && heights[j] >= heights[i])
                j = right_idx[j];
            right_idx[i] = j;
        }
        for (int i = 0; i < heights.size(); ++i) 
            if (heights[i] * (right_idx[i] - left_idx[i] - 1) > max_area)
                max_area = heights[i] * (right_idx[i] - left_idx[i] - 1);
        return max_area;
    }
    
};
```

### Solution 2

我第一次做的时候用分治的思路，即找到当前高度最小的方块$i$，则包含它的矩阵面积为`(right - left) * heights[i]`，然后分别在(left,i-1)和(i+1,right)两个区域内找最大矩阵，然后比较三者大小即可。这样的时间复杂度为`O(nlogn)`。

```
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        return search(heights, 0, heights.size());
    }
    
    int search(vector<int>& heights, int left, int right) {
        if (left == right)
            return 0;
        int smallest_idx, smallest_height = ~(1 << 31);
        for (int i = left; i < right; ++i)
            if (heights[i] <= smallest_height) {
                smallest_idx = i;
                smallest_height= heights[i];
            }
        int largest_area = (right - left) * smallest_height;
        int left_area = search(heights, left, smallest_idx);
        int right_area = search(heights, smallest_idx + 1, right);
        if (left_area > largest_area)
            largest_area = left_area;
        if (right_area > largest_area)
            largest_area = right_area;
        return largest_area;
    }
};
```

### Solution 3

还有一种用栈的思路，用一个栈保存每个方块的索引，且从栈底到栈顶的方块高度是升序的。从左到右遍历每一个方块，如果当前方块的高度大于栈顶方块的高度时，则压栈；否则，表示以栈顶方块的高度形成的矩阵不能与当前方块组成矩阵，则计算以栈顶方块的高度形成的矩阵的面积大小，然后弹出栈顶，继续比较下一个栈顶和当前方块的高度大小。



