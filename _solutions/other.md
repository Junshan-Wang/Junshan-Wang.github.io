---
title: "Other"
collection: solutions
permalink: /solutions/other/
author_profile: true
---

### 73. Set Matrix Zeroes

Given a m x n matrix, if an element is 0, set its entire row and column to 0. Do it in-place.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/set-matrix-zeroes

很容易想到需要额外的空间$O(mn)$或者$O(m+n)$，如果希望不需要额外的空间，可以考虑用第一行和第一列作为标记，而第一行和第一列的标记则用两个常数标记即可。

```
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int r = 1, c = 1;
        for (int i = 0; i < matrix.size(); ++i) {
            if (matrix[i][0] == 0) c = 0;
        }
        for (int j = 0; j < matrix[0].size(); ++j) {
            if (matrix[0][j] == 0) r = 0;
        }
        for (int i = 1; i < matrix.size(); ++i) {
            for (int j = 1; j < matrix[0].size(); ++j) {
                if (matrix[i][j] == 0) {
                    matrix[0][j] = 0;
                    matrix[i][0] = 0;
                }
            }
        }
        for (int i = 1; i < matrix.size(); ++i) {
            if (matrix[i][0] == 0) {
                for (int j = 0; j < matrix[0].size(); ++j) matrix[i][j] = 0;
            }
        }
        for (int j = 1; j < matrix[0].size(); ++j) {
            if (matrix[0][j] == 0) {
                for (int i = 0; i < matrix.size(); ++i) matrix[i][j] = 0;
            }
        }
        if (c == 0) {
            for (int i = 0; i < matrix.size(); ++i) matrix[i][0] = 0;
        }
        if (r == 0) {
            for (int j = 0; j < matrix[0].size(); ++j) matrix[0][j] = 0;
        }
    }
};
```