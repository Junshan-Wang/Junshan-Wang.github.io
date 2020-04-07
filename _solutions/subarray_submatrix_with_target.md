---
title: "Subarray / Submatrix with Target"
collection: solutions
permalink: /solutions/subarray_submatrix_with_target/
author_profile: true
---

和子数组、子矩阵以及给定目标值相关的题型。  
* 最简单的题型，返回子数组和为K的数量（560）/ 最大长度（325）：前缀和+哈希，时间复杂度O(n)
* 扩展到二维矩阵情况（1074）：在某一维上用sliding window的方法固定一个区间，转换为一维情况，注意需要提前对行之和进行前缀和预处理，时间复杂度O(m^2n)
* 扩展到子矩阵和不大于K的最大和：不能直接用哈希，而是采用有序结构存储+二分查找的方法，比如采用`set`结构，对应`insert`和`lower_bound`函数，时间复杂度为O(m^2nlogn)

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

## 363. Max Sum of Rectangle No Larger Than K

Given a non-empty 2D matrix matrix and an integer k, find the max sum of a rectangle in the matrix such that its sum is no larger than k.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/max-sum-of-rectangle-no-larger-than-k

首先采用上题解法，固定两列之间的区间，转换为一维情况。

不能继续采用hash方法，因为无法直接访问该值，所以考虑维护一个有序结构，该数据结构的插入和查找都是O(logn)的复杂度，所以用`set`。

```
class Solution {
public:
    int maxSumSubmatrix(vector<vector<int>>& matrix, int k) {
        int m = matrix.size(), n = matrix[0].size();
        int max_sum = - INT_MAX;
        vector<vector<int>> prefix(m + 1, vector<int>(n + 1, 0));
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                prefix[i][j] = prefix[i][j - 1] + matrix[i - 1][j - 1];
            }
        }
        for (int i = 1; i <= n; ++i) {
            for (int j = i; j <= n; ++j) {
                set<int> pres{0};
                int cur_pre = 0;
                for (int l = 1; l <= m; ++l) {
                    cur_pre = prefix[l][j] - prefix[l][i - 1] + cur_pre; 
                    set<int>::iterator it = pres.lower_bound(cur_pre - k);
                    if (it == pres.end());
                    else if (cur_pre - *it > max_sum) max_sum = cur_pre - *it;
                    pres.insert(cur_pre);
                }
            }
        }
        return max_sum;
    }
};
```
