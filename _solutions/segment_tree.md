---
title: "Segment Tree"
collection: solutions
permalink: /solutions/segment_tree/
author_profile: true
---

### NOTE

* 线段树，一种二叉搜索树，每个节点保存一个区间`[left,right]`，以及对应于这个区间的`value`（比如sum/max等）。
* 线段树的更新和查找的复杂度都是`O(logn)`。
* 线段树可以扩展到二维或者更高维度，比如在二维情况下每个节点保存一个区间，可以进一步划分成四个区域。
* 线段树可以用一个二维数组保存，`tree[N<<2]`，对于每一个节点`k`（下标从1开始），它的左右子节点分别是`2k`和`2k+1`。
* 线段树的插入、更新和查找都是采用递归的形式，即节点从上往下分裂，并且结果从下往上合并。
  
### 307. Range Sum Query - Mutable

Given an integer array nums, find the sum of the elements between indices i and j (i ≤ j), inclusive.

The update(i, val) function modifies nums by updating the element at index i to val.

Note:

The array is only modifiable by the update function.  
You may assume the number of calls to update and sumRange function is distributed evenly.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/range-sum-query-mutable

如果不做预处理，更新的复杂度为O(1)，求和的复杂度为O(n)。  
如果进行预处理，求出前缀和数组，更新的复杂度为O(n)，求和的复杂度为O(1)。  
采用线段树进行保存，更新和求和的复杂度都为O(logn)。

```
class NumArray {
public:
    vector<int> nums;
    vector<int> tree;
    NumArray(vector<int>& nums) {
        for (int num : nums) this -> nums.push_back(num);
        this -> tree = vector<int>(nums.size() << 2, 0);
        if (nums.size() > 0) build_tree(1, 0, nums.size() - 1);
    }

    void build_tree(int k, int left, int right) {
        if (left == right) {
            tree[k] = nums[left];
            return;
        }
        int middle = (left + right) / 2;
        build_tree((k << 1), left, middle);
        build_tree((k << 1) + 1, middle + 1, right);
        tree[k] = tree[(k << 1)] + tree[(k << 1) + 1];
    }

    void update_tree(int k, int i, int val, int left, int right) {
        if (left == right) {
            tree[k] = val;
            return;
        }
        int middle = (left + right) / 2;
        if (i <= middle) update_tree((k << 1), i, val, left, middle);
        else update_tree((k << 1) + 1, i, val, middle + 1, right);
        tree[k] = tree[(k << 1)] + tree[(k << 1) + 1];
    }

    int query(int k, int i, int j, int left, int right) {
        if (i <= left && j >= right) return tree[k];
        int middle = (left + right) / 2;
        int left_val = 0, right_val = 0;
        if (i <= middle) left_val = query((k << 1), i, j, left, middle);
        if (j > middle) right_val = query((k << 1) + 1, i, j, middle + 1, right);
        return left_val + right_val;
    }
    
    void update(int i, int val) {
        update_tree(1, i, val, 0, nums.size() - 1);
    }
    
    int sumRange(int i, int j) {
        return query(1, i, j, 0, nums.size() - 1);
    }
};

/**
 * Your NumArray object will be instantiated and called as such:
 * NumArray* obj = new NumArray(nums);
 * obj->update(i,val);
 * int param_2 = obj->sumRange(i,j);
 */
 ```

### 308. Range Sum Query 2D - Mutable

Given a 2D matrix matrix, find the sum of the elements inside the rectangle defined by its upper left corner (row1, col1) and lower right corner (row2, col2).

Note:

The matrix is only modifiable by the update function.  
You may assume the number of calls to update and sumRegion function is distributed evenly.  
You may assume that row1 ≤ row2 and col1 ≤ col2.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/range-sum-query-2d-mutable

复杂度分析和上题一样，不做预处理或者二维预处理的复杂度为O(n^2)，线段树的复杂度为O(logn^2)，只是这里应该用二维的线段树，太复杂了不写了。  
采用另一种折衷，使得更新和查询的复杂度都为O(n)。

```
class NumMatrix {
public:
    vector<vector<int>> a;
    vector<vector<int>> row_sum;
    NumMatrix(vector<vector<int>>& matrix) {
        int m = matrix.size();
        if (m == 0) return;
        int n = matrix[0].size();
        row_sum = vector<vector<int>>(m, vector<int>(n, 0));
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                row_sum[i][j] = matrix[i][j] + (j > 0 ? row_sum[i][j - 1] : 0);
            }
        }
        a = matrix;
    }
    
    void update(int row, int col, int val) {
        for (int j = col; j < row_sum[row].size(); ++j) {
            row_sum[row][j] += (val - a[row][col]);
        }
        a[row][col] = val;
    }
    
    int sumRegion(int row1, int col1, int row2, int col2) {
        int sum = 0;
        for (int i = row1; i <= row2; ++i) {
            sum += (row_sum[i][col2] - (col1 > 0 ? row_sum[i][col1 - 1] : 0));
        }
        return sum;
    }
};

/**
 * Your NumMatrix object will be instantiated and called as such:
 * NumMatrix* obj = new NumMatrix(matrix);
 * obj->update(row,col,val);
 * int param_2 = obj->sumRegion(row1,col1,row2,col2);
 */
 ```