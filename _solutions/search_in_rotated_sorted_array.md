---
title: "#33. Search in Rotated Sorted Array"
collection: solutions
permalink: /solutions/search_in_rotated_sorted_array/
author_profile: true
---


Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.

(i.e., `[0,1,2,4,5,6,7]` might become `[4,5,6,7,0,1,2]`).

You are given a target value to search. If found in the array return its index, otherwise return -1.

You may assume no duplicate exists in the array.

Your algorithm's runtime complexity must be in the order of `O(log n)`.

---

### 解法 1
我的解法是直接进行二分查找，不过会分情况讨论，即判断是会落在旋转的前半段还是后半段，判断的根据是前半段的数都比后半段的数大，但是这种方法其实较为繁琐。

```
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int pivot = 1;
        int left = 0, right = nums.size() - 1;
        int middle;
        while (left <= right) {
            middle = (left + right) / 2;
            //cout << left << " " << right << " " << middle << endl;
            if (nums[middle] == target) 
                return middle;
            else if (nums[middle] < target) {
                if (middle < nums.size() - 1 && nums[middle] > nums[middle + 1])
                    return -1;
                if (nums[middle] > nums[right])
                    left = middle + 1;
                else if (nums[right] >= target)
                    left = middle + 1;
                else
                    right = middle - 1;
            }
            else if (nums[middle] > target) {
                if (middle > 0 && nums[middle - 1] > nums[middle])
                    return -1;
                if (nums[middle] < nums[left])
                    right = middle - 1;
                else if (nums[left] <= target)
                    right = middle - 1;
                else
                    left = middle + 1;
            }
        }
        return -1;
    }
};
```

### 解法 2
网上看到还有第二种解法，过程更加清晰，分成两步：先对旋转后的数组进行二分查找，找到旋转点，建立旋转前和旋转后的数组下标的对应关系，即可以认为得到了旋转前的原数组；然后直接在原数组上进行普通的二分查找。