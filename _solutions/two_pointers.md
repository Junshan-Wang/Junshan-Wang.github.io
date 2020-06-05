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

### 75. Sort Colors

Given an array with n objects colored red, white or blue, sort them in-place so that objects of the same color are adjacent, with the colors in the order red, white and blue.

Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.

Note: You are not suppose to use the library's sort function for this problem.

Follow up:

A rather straight forward solution is a two-pass algorithm using counting sort.
First, iterate the array counting number of 0's, 1's, and 2's, then overwrite array with total number of 0's, then 1's and followed by 2's.
Could you come up with a one-pass algorithm using only constant space?

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/sort-colors

需要一次遍历完成排序，考虑快排partition的思路，同时注意到我们只需要把0都放在开头，把2都放在结尾即可，所以可以用两个指针分别从开始和结束记录0和2的位置，此外再用一个指针从头到尾去遍历每个数进行交互。

```
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int l = 0, r = nums.size() - 1, cur = 0;
        while (cur <= r) {
            if (nums[cur] == 0) swap(nums[cur++], nums[l++]);
            else if (nums[cur] == 2) swap(nums[cur], nums[r--]);
            else cur++;
        }
    }
};
```

### 28. Implement strStr()

Implement strStr().

Return the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/implement-strstr

最简单的方法是遍历依次比较，复杂度是$O(mn)$，可以通过双指针/滑动窗口的方法，将字符串用数字表示，时间复杂度为$O(m+n)$。  
注意到，当字符串很长时，会溢出，所以需要整除一个整数来实现。

```
class Solution {
public:
    int strStr(string haystack, string needle) {
        if (needle.size() == 0) return 0;
        long long target = 0, source = 0, begin = 1;
        for (int i = 0; i < needle.size(); ++i) {
            target = (target * 26 + needle[i] - 'a') % INT_MAX;
            source = (source * 26 + haystack[i] - 'a') % INT_MAX;
            begin = (begin * 26) % INT_MAX;
        }
        for (int i = needle.size(); i < haystack.size(); ++i) {
            if (source == target) return i - needle.size();
            else source = (source * 26 - (haystack[i - needle.size()] - 'a') * begin + haystack[i] - 'a') % INT_MAX;
        }
        if (source == target) return haystack.size() - needle.size();
        else return -1;
    }
};
```