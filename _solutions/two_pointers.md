---
title: "Two Pointers"
collection: solutions
permalink: /solutions/two_pointers/
author_profile: true
---

### 777. Swap Adjacent in LR String

In a string composed of 'L', 'R', and 'X' characters, like "RXXLRXRXL", a move consists of either replacing one occurrence of "XL" with "LX", or replacing one occurrence of "RX" with "XR". Given the starting string start and the ending string end, return True if and only if there exists a sequence of moves to transform one string to the other.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/swap-adjacent-in-lr-string

题目有些歧义。

其实这道题的意思是，`L`只能向左移动，`R`只能向右移动。且注意到`start`和`end`中的`L`和`R`不能交错。

所以用两个指针`p1`和`p2`，分别指向`start`和`end`，遍历每一个`L`或`R`，要求：

* 两个指针指向的字母必须相同
* 如果当前指向的是`L`，则`p1>=p2`
* 如果当前指向的是`R`，则`p1<=p1`

如果遍历完整个字符串都满足上述要求，则返回true，否则返回false。


### 15. 3Sum

Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.

Note:

The solution set must not contain duplicate triplets.

2sum问题的扩展，可以简单的枚举第一个数，然后对于剩下两个数解2sum问题即可。

可以先进行排序，然后用左右指针的方法求解，排除了一些不可行的情况，复杂度降为O(n^2)。
```
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int left, right, n = nums.size();
        vector<vector<int>> res(0);
        sort(nums.begin(), nums.end());
        for (int i = 0; i < n; ++i) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            left = i + 1;
            right = n - 1;
            while (left < right) {
                while (left < right && nums[left] + nums[right] > - nums[i]) right--;
                if (left < right && nums[left] + nums[right] == - nums[i]) {
                    while (left < right && nums[right - 1] == nums[right]) right--;
                    res.push_back(vector<int>{nums[i], nums[left], nums[right]});
                    right--;
                }
                while (left < right && nums[left] + nums[right] < - nums[i]) left++;
                if (left < right && nums[left] + nums[right] == - nums[i]) {
                    while (left < right && nums[left + 1] == nums[left]) left++;
                    res.push_back(vector<int>{nums[i], nums[left], nums[right]});
                    left++;
                }
            }
        }
        return res;
    }
};
```

也可以用hash表来处理，不过考虑到不能出现重复的结果，所以还需要一些额外的考虑，所以这里简单的通过提前排序来简化问题。

```
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int left, right, n = nums.size();
        vector<vector<int>> res(0);
        unordered_map<int, int> num2cnt;
        sort(nums.begin(), nums.end());
        for (int i = 0; i < n; ++i) {
            if (num2cnt.find(nums[i]) == num2cnt.end()) num2cnt[nums[i]] = 0;
            num2cnt[nums[i]] = i;
        }
        for (int i = 0; i < n; ++i) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            for (int j = i + 1; j < n; ++j) {
                if (j > i + 1 && nums[j] == nums[j - 1]) continue;
                int val = 0 - nums[i] - nums[j];
                if (num2cnt.find(val) != num2cnt.end() && num2cnt[val] > j) 
                    res.push_back(vector<int>{nums[i], nums[j], val});
            }
        }
        return res;
    }
};
```