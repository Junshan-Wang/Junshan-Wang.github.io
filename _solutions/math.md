---
title: "Math"
collection: solutions
permalink: /solutions/math/
author_profile: true
---

## 最小距离

最简单的一维情况：  
给定数组a，找到x，使得数组中每个数到x的距离最小，即`|a[0] - x| + |a[1] - x| + ... + |a[n-1] - x|`最小。  
*当x为a的中位数时使得距离最小。当a的大小为奇数时x为该中位数；当a的大小为偶数时x为两个中位数中间的任何一个数即可。*  
查找中位数可以先进行排序然后直接得到，时间复杂度为O(nlogn)。也可以通过快速选择算法（快排的变体，选择第k大数），时间复杂度为O(n)。

### 462. Minimum Moves to Equal Array Elements II

Given a non-empty integer array, find the minimum number of moves required to make all array elements equal, where a move is incrementing a selected element by 1 or decrementing a selected element by 1.

You may assume the array's length is at most 10,000.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/minimum-moves-to-equal-array-elements-ii

```
class Solution {
public:
    int minMoves2(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        int med = nums[nums.size() / 2], res = 0;
        for (int num : nums) res += abs(num - med);
        return res;
    }
};
```

### 296. Best Meeting Point

A group of two or more people wants to meet and minimize the total travel distance. You are given a 2D grid of values 0 or 1, where each 1 marks the home of someone in the group. The distance is calculated using Manhattan Distance, where distance $(p1, p2) = \|p2.x - p1.x\| + \|p2.y - p1.y\|$.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/best-meeting-point

二维情况下的曼哈顿距离，其实就是分成两个方向，分别求最小距离。

```
class Solution {
public:
    int minTotalDistance(vector<vector<int>>& grid) {
        vector<int> xs, ys;
        for (int i = 0; i < grid.size(); ++i) {
            for (int j = 0; j < grid[0].size(); ++j) {
                if (grid[i][j] == 1) {
                    xs.push_back(i);
                    ys.push_back(j);
                }
            }
        }
        sort(xs.begin(), xs.end());
        sort(ys.begin(), ys.end());
        int med_x = xs[xs.size() / 2], med_y = ys[ys.size() / 2];
        int res = 0;
        for (int i = 0; i < xs.size(); ++i) {
            res += abs(xs[i] - med_x);
            res += abs(ys[i] - med_y);
        }
        return res;
    }
};
```

---

## 函数性质

### 360. Sort Transformed Array

Given a sorted array of integers nums and integer values a, b and c. Apply a quadratic function of the form f(x) = ax2 + bx + c to each element x in the array.

The returned array must be in sorted order.

Expected time complexity: O(n)

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/sort-transformed-array

根据抛物线的性质，即结果先增后减（先减后增），可以实现在O(n)的复杂度内进行排序。

```
class Solution {
public:
    vector<int> sortTransformedArray(vector<int>& nums, int a, int b, int c) {
        vector<int> res1, res2;
        int min_idx = 0, neg = (a < 0);
        for (int i : nums) res1.push_back(a * i * i + b * i + c);
        if (neg) for (int i = 0; i < res1.size(); ++i) res1[i] = - res1[i];

        for (int i = 1; i < res1.size(); ++i) {
            if (res1[i] < res1[i - 1]) min_idx = i;
        }

        int p1 = 0, p2 = 1;
        for (int i = 0; i < res1.size(); ++i) {
            if (min_idx - p1 < 0) res2.push_back(res1[min_idx + (p2++)]);
            else if (min_idx + p2 >= res1.size()) res2.push_back(res1[min_idx - (p1++)]);
            else if (res1[min_idx - p1] <= res1[min_idx + p2]) res2.push_back(res1[min_idx - (p1++)]);
            else res2.push_back(res1[min_idx + (p2++)]);
        }
        
        if (neg) {
            for (int i = 0; i < res2.size() / 2; ++i) {
                int tmp = res2[i];
                res2[i] = res2[res2.size() - 1 - i];
                res2[res2.size() - 1 - i] = tmp;
            }
            for (int i = 0; i < res2.size(); ++i) res2[i] = - res2[i];
        }
        return res2;
    }
};
```

---

### 280. Wiggle Sort

Given an unsorted array nums, reorder it in-place such that nums[0] <= nums[1] >= nums[2] <= nums[3]....

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/wiggle-sort

最直接的是先排序的做法，然后相邻两个数字交换即可，时间复杂度是O(nlogn)。

```
class Solution {
public:
    void wiggleSort(vector<int>& nums) {
        if (nums.size() == 0) return;
        sort(nums.begin(), nums.end());
        for (int i = 1; i < nums.size() - 1; i += 2) {
            int t = nums[i + 1];
            nums[i + 1] = nums[i];
            nums[i] = t;
        }
    }
};
```

然而实际上，可以不用排序。直接考虑相邻两个数字。在奇数位，如果`nums[i] < nums[i + 1]`，由于`nums[i - 1] < nums[i]`，所以直接交换两数即可，由传递性，可以保证`nums[i - 1] < nums[i + 1] > nums[i]`。在偶数位同理。

依次遍历数组元素，局部的修改不会受到之前结果的影响。

```
class Solution {
public:
    void wiggleSort(vector<int>& nums) {
        for (int i = 0; i < nums.size(); i += 2) {
            if (i + 1 < nums.size() && nums[i] > nums[i + 1]) swap(nums[i], nums[i + 1]);
            if (i + 2 < nums.size() && nums[i + 1] < nums[i + 2]) swap(nums[i + 1], nums[i + 2]);
        }
    }
};
```

### 41. First Missing Positive

Given an unsorted integer array, find the smallest missing positive integer.

Your algorithm should run in O(n) time and uses constant extra space.

由于不能用额外的空间，所以都是对访问过的元素作为下标对应的元素做标记，然后从头遍历，找到第一个无标记的位置

我的思路是将访问过的元素标记为下标，同时下一步访问它原始元素对应的下标。还可以元素和其元素对应下标的数进行交换。
```
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int t = 0;
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] > nums.size() || nums[i] <= 0 || nums[i] == i + 1) continue;
            else {
                int j = nums[i] - 1;
                while (j < nums.size() && j >= 0 && nums[j] != j + 1) {
                    t = nums[j] - 1;
                    nums[j] = j + 1;
                    j = t;
                    
                }
            }
        }
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] != i + 1) return i + 1;
        }
        return nums.size() + 1;
    }
};
```


还可以用负数标记，不过需要进行预处理，首先将所有负数标记为某个特定的值（1），那么首先还需要确定原数组是否存在1。  
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

### 剑指 Offer 43. 1～n整数中1出现的次数

输入一个整数 n ，求1～n这n个整数的十进制表示中1出现的次数。

例如，输入12，1～12这些整数中包含1 的数字有1、10、11和12，1一共出现了5次。



来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof  

考虑每一位出现1的次数，需要同时考虑比它高的位数和比它低的位数。分成三种情况考虑：
* 如果当前位数的数字`cur=0`，则出现1的次数是，它的高位数字`high`乘上低位的总次数
* 如果`cur=1`，则出现1的次数是，它的高位数字`high`乘上低位的总次数，再加上低位数字`low`
* 如果`cur>1`，则出现1的次数是，它的高位数字加1`high+1`乘上低位的总次数

```
class Solution {
public:
    int countDigitOne(int n) {
        int cnt = 0;
        for (int i = 0; i < 100; ++i) {
            long long mask = pow(10, i);
            long long low = n % mask;
            long long high = n / (mask * 10);
            long long cur = n / mask % 10;

            if (cur == 0 && high == 0) break;
            else if (cur == 0) cnt += high * mask;
            else if (cur == 1) cnt += high * mask + low + 1;
            else cnt += (high + 1) * mask;
        }
        return cnt;
    }
};
```