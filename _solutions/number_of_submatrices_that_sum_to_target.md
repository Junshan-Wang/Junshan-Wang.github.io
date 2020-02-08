---
title: "Sliding Window"
collection: solutions
permalink: /solutions/number_of_submatrices_that_sum_to_target/
author_profile: true
---

## 560. Subarray Sum Equals K
Given an array of integers and an integer `k`, you need to find the total number of continuous subarrays whose sum equals to `k`.

直接暴力搜索，复杂度是`O(n^2)`。因此采用前缀+哈希的方法
* 根据前缀和`s[i]`可以在常数时间内求出某个区间内的和，即`s[i:j] = s[:j] - s[:i-1] = k`（注意，`s[i]`不需要提前求出，在计算过程中可以直接得到）
* `unordered_map<sum_of_prefix, cnt_of_sum>`，当遍历到第`i`个数时，首先计算当前的前缀和`s`，根据当前前缀`s[:i]`和目标`k`判断以第`i`个数结尾的和为`k`的区间数量，然后讲当前前缀和加入哈希表。
  
时间复杂度为`O(n)`。

```
int subarraySum(vector<int>& nums, int k) {
    int s = 0, ans = 0;
    unordered_map<int, int> sums;
    sums[0] = 1;
    for (int num : nums) {
        s += num;
        if (sums.find(s - k) != sums.end()) 
            ans += sums[s - k];
        sums[s] += 1;
    }
    return ans;
}
```

### 325. Maximum Size Subarray Sum Equals k
Given an array nums and a target value k, find the maximum length of a subarray that sums to k. If there isn't one, return 0 instead.

解法类似，时间复杂度为`O(n)`。

```
int maxSubArrayLen(vector<int>& nums, int k) {
    unordered_map<int, int> sums;
    int cur = 0, ans = 0;
    sums[0] = 0;
    for (int i = 0; i < nums.size(); ++i) {
        cur += nums[i];
        if (sums.find(cur - k) != sums.end())
            ans = (ans > i - sums[cur - k] + 1 ? ans : i - sums[cur - k] + 1);
        if (sums.find(cur) == sums.end())
            sums[cur] = i + 1;
    }
    return ans;
}
```


## 1074. Number of Submatrices That Sum to Target
Given a matrix, and a target, return the number of non-empty submatrices that sum to target.

A submatrix `x1, y1, x2, y2` is the set of all cells `matrix[x][y]` with `x1 <= x <= x2` and `y1 <= y <= y2`.

Two submatrices `(x1, y1, x2, y2)` and `(x1', y1', x2', y2')` are different if they have some coordinate that is different: for example, if `x1 != x1'`.

直接暴力搜索，复杂度是`O(n^4)`，题目经过简单变形后，可以转换为上题的解法。

固定两列之间的区间，区间内每一行的数相加（同样可以用前缀方法，提前计算前缀，每一行计算区间和的复杂度为`O(n)`），此时得到一个一维数组，找到这个一维数组和为`target`的区间数，即用上题思路复杂度为`O(n)`。该过程有`O(n^2)`次，总的复杂度为`O(n^3)`。

```
int numSubmatrixSumTarget(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size(), ans = 0;
    for (int k = 0; k < m; ++k)
        for (int i = 1; i < n; ++i) 
            matrix[k][i] = (i > 0 ? matrix[k][i - 1] + matrix[k][i] : matrix[k][i]);
    for (int i = 0; i < n; ++i) {
        for (int j = i; j < n; ++j) {
            unordered_map<int, int> cnts;
            cnts[0] = 1;
            int s = 0;
            for (int k = 0; k < m; ++k) {
                s += (i > 0 ? sums[k][j] - sums[k][i - 1] : sums[k][j]);
                ans += cnts[s - target];
                cnts[s] += 1;
            }
        }
    }
    return ans;
}
```