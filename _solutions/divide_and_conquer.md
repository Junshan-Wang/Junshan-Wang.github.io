---
title: "Divide and Conquer"
collection: solutions
permalink: /solutions/divide_and_conquer/
author_profile: true
---

### 215. Kth Largest Element in an Array

Find the kth largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element.

简单的排序后取中位数复杂度是O(nlogn)，可以有更简单的思想，采用类似快速排序的方法，不过不需要进行完整的快排，只需要在对应的左/右区间进行查找即可，时间复杂度为O(n)。

```
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        int i = 0, j = nums.size() - 1;
        k--;
        while (i <= j) {
            int pos = partition(nums, i, j);
            if (pos == k) return nums[pos];
            else if (k < pos) j = pos - 1;
            else i = pos + 1;
        }
        return nums[i];
    }

    int partition(vector<int>& nums, int i, int j) {
        while (i < j) {
            while (j > i && nums[j] <= nums[i]) j--;
            swap(nums[j], nums[i]);
            while (i < j && nums[i] >= nums[j]) i++;
            swap(nums[i], nums[j]);
        }
        return i;
    }
};
```

### 4. Median of Two Sorted Arrays

There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

You may assume nums1 and nums2 cannot be both empty.

直接合并两个数组并得到中位数的复杂度是O(m+n)，可以进一步优化。

很容易联想到采用二分查找的思路，只是这里是两个数组。我们可以对长度较小的数组采用二分查找方法，这样能保证另一个数组不会溢出，并且可以知道如果第一个数组当前指向i，则第二个数组应该指向(m+n+1)/2-i，这样能保证左右部分各一半。这题的边界条件比较复杂！

```
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int m = nums1.size(), n = nums2.size();
        if (m > n) return findMedianSortedArrays(nums2, nums1);
        int left = 0, right = m, i, j;
        while (left <= right) {
            i = (left + right) / 2;
            j = (m + n + 1) / 2 - i;

            if (i < right && nums1[i] < nums2[j - 1]) left = i + 1;
            else if (i > left && nums2[j] < nums1[i - 1]) right = i - 1;
            else {
                int max_l = 0, min_r = 0;
                if (i == 0) max_l = nums2[j - 1];
                else if (j == 0) max_l = nums1[i - 1];
                else max_l = max(nums1[i - 1], nums2[j - 1]);
                if ((m + n) % 2 == 1) return max_l;
                
                if (i == m) min_r = nums2[j];
                else if (j == n) min_r = nums1[i];
                else min_r = min(nums1[i], nums2[j]);

                
                return (double)(max_l + min_r) / 2.0;
            }
        }
        return 0;
    }
};
```

### 395. Longest Substring with At Least K Repeating Characters

Find the length of the longest substring T of a given string (consists of lowercase letters only) such that every character in T appears no less than k times.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/longest-substring-with-at-least-k-repeating-characters

暴力遍历每一种子串的方法复杂度是`O(n^2)`，简单的滑动窗口无法解（似乎可以改进，分别考虑子串内不同字符数量为0～26时的情况）。  
这里才用分治的方法，对于每一个子串，遍历一次统计每个字符出现的次数，然后找到其出现次数小于`k`的字符出现的位置，并进行`m`路分治。复杂度是`O(nlogn)`。

```
class Solution {
public:
    int longestSubstring(string s, int k) {  
        vector<int> cha2cnt(26, 0);
        int max_len = 0, min_cnt_idx = -1, last_split = -1;
        for (int i = 0; i < s.size(); ++i) cha2cnt[s[i] - 'a'] += 1;
        for (int i = 0; i < 26; ++i) if (cha2cnt[i] > 0 && (min_cnt_idx == -1 || cha2cnt[i] < cha2cnt[min_cnt_idx])) min_cnt_idx = i;
        if (min_cnt_idx == -1 || cha2cnt[min_cnt_idx] >= k) return s.size();
        for (int i = 0; i < s.size(); ++i) {
            if (s[i] - 'a' == min_cnt_idx) {
                if (i - last_split - 1 > 0 && i - last_split - 1 > max_len) {
                    string t = s.substr(last_split + 1, i - last_split - 1);
                    max_len = max(max_len, longestSubstring(t, k));
                }
                last_split = i;
            } 
        }
        string t = s.substr(last_split + 1, s.size() - last_split - 1);
        max_len = max(max_len, longestSubstring(t, k));
        return max_len;
    }
};
```