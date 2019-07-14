---
title: "#240. Search a 2D Matrix II"
collection: solutions
permalink: /solutions/search_a_2d_matrix_2/
author_profile: true
---


Write an efficient algorithm that searches for a value in an m x n matrix. This matrix has the following properties:

Integers in each row are sorted in ascending from left to right.
Integers in each column are sorted in ascending from top to bottom.
Example:

Consider the following matrix:

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

Given target = `5`, return `true.`

Given target = `20`, return `false`.

from Leetcode

---
在一个排好序的一维数组中查找是否存在某个数，用二分查找的时间复杂度是`O(logn)`，即一个节点数为`n`的二叉树，查找的复杂度为从根节点到一个叶子结点的路径长度，即树的高度`logn`。

扩展到二维数组上时，在每一步将数组划分成左上、左下、右上、右下四个部分，对于每个部分，如果target小于左上角的数或者大于右上角的数，则返回false，否则继续划分。但是此时复杂度是`O(mn)`？因为此时是一个节点数为`mn`的4叉树，且查找的复杂度不再是一条路径，在每个非叶子结点处最多会有三个子节点继续搜索。

所以这里用的应该是*动态规划*的思想，从右上角开始遍历，当遍历到`matrix[i][j]`时，我们已知`target`不在`matrix[0:i][0:n]`和`matrix[0:m][j+1:n]`中，即只有可能在当前位置的左下角`matrix[i:m][0:j+1]`。此时
* 如果`matrix[i][j]>target`，则`target`不可能在`matrix[i:m][j]`，所以继续搜索`matrix[i][j-1]`
* 如果`matrix[i][j]<target`，则`target`不可能在`matrix[i][0:j+1]`，所以继续搜索`matrix[i+1][j]`

搜索直到`i>=m`或者`j<0`，从右上走到左下，时间复杂度为`O(m+n)`。

```
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int size1 = matrix.size();
        if (size1 == 0)
            return false;
        int size2 = matrix[0].size();
        int i = 0;
        int j = size2 - 1;
        while (i < size1 && j >= 0) {
            if (target == matrix[i][j])
                return true;
            else if (target < matrix[i][j])
                j--;
            else 
                i++;
        }
        return false;
    }
};
```