---
title: "#41. First Missing Positive"
collection: solutions
permalink: /solutions/first_missing_positive/
author_profile: true
---

Given an unsorted integer array, find the smallest missing positive integer.

Example 1:
```
Input: [1,2,0]
Output: 3
```
Example 2:
```
Input: [3,4,-1,1]
Output: 2
```
Example 3:
```
Input: [7,8,9,11,12]
Output: 1
```

Note:

Your algorithm should run in O(n) time and uses constant extra space.

from LeetCode

---

这一题的解法非常巧，用到(key, value)的逆映射，同时由于不能使用额外的空间，所以用到负号进行标记。

如果不对空间复杂度进行限制，则可以用一个辅助数组`B`（初始化为0），数组的每个元素表示对应下标的数是否出现过，辅助数组的大小为原数组的最大值，只需要遍历原数组`A`，遍历到`a[i]`时，将B数组对应下标为`a[i]`的数`b[a[i]]`置为1。最后只要再遍历一次`B`数组，输出第一个为0的数的下标即可。

当限制空间复杂度时，我们首先注意到，第一个不存在的数一定小于等于`len(A)+1`（即`[1,len(A)]`有不存在的数，或者`[1,len(A)]`都存在则为`len(A)+1`）。当遍历到`a[i]`时，`a[abs(a[i])]=-a[abs(a[i])]`，可以实现既更新了第`i`个元素的出现情况，同时保留了第`a[i]`个元素的值。

```
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] <= 0 || nums[i] > nums.size())
                nums[i] = nums.size() + 1;
        }
        for (int i = 0; i < nums.size(); ++i) {
            int pos = abs(nums[i]) - 1;
            if (pos >= 0 && pos < nums.size() && nums[pos] > 0)
                nums[pos] = - nums[pos];
        }
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] > 0)
                return i + 1;
        }
        return nums.size() + 1;
    }
};
```