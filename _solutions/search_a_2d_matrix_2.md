---
title: "Sort Colors"
collection: solutions
permalink: /solutions/sortcolors/
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

Given target = `20`, return `false.



来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/search-a-2d-matrix-ii
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

---
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