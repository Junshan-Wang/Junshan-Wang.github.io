---
title: "Binary Search"
collection: solutions
permalink: /solutions/binary_search/
author_profile: true
---

二分查找的问题，往往关键是定义需要查找的结构是什么，且结构必须是排序的，然后查找过程是简单的，即通过三个指针`left`、`right`和`middle`迭代直至收敛。

C++中有三个二分查找的函数：`lower_bound(vec.begin(), vec.end(), val)`返回大于或等于`val`的第一个元素的位置，`upper_bound`返回大于`val`的第一个元素位置，和`binary_search`返回是否存在指定元素。

---
### 528. Random Pick with Weight

Given an array w of positive integers, where w[i] describes the weight of index i, write a function pickIndex which randomly picks an index in proportion to its weight.

Note:

1 <= w.length <= 10000
1 <= w[i] <= 10^5
pickIndex will be called at most 10000 times.

根据题意，保存一个数组，是到当前元素的权重之和，则该数组的元素是递增的。每次`pick`的时候，调用随机函数生成一个随机数，查找该数处于那个位置即可，查找过程用二分查找，可以直接调用`lower_bound`函数即可（注意不是`upper_bound`）。

```
class Solution {
public:
    vector<int> s;
    int max_s;
    Solution(vector<int>& w) {
        for (int i = 0; i < w.size(); ++i)
            s.push_back(w[i] + (i > 0 ?  s[i - 1] : 0));
        max_s = s.back();
    }
    
    int pickIndex() {
        int r = rand() % max_s + 1;
        return lower_bound(s.begin(), s.end(), r) - s.begin();
    }
};
```

---
### 774. Minimize Max Distance to Gas Station
On a horizontal number line, we have gas stations at positions stations[0], stations[1], ..., stations[N-1], where N = stations.length.

Now, we add K more gas stations so that D, the maximum distance between adjacent gas stations, is minimized.

Return the smallest possible value of D.

这题的解法和上题不同，不是根据`station`进行查找，而是查找答案即最小的可能距离`D`。对于每一距离`d`，我们可以检查它是否满足条件，即是否能保证新建`K`个加油站后彼此之间距离小于`d`。如果满足条件，则将`d`缩小，从而寻找更小的可能`d`；如果不满足条件，则放大`d`，直到找到可能的`d`。而这个放大缩小的过程可以用二分查找的思路。由于此时查找的不再是整数，而是浮点数，所以停止的条件是左右指针之间的距离小于给定误差范围。

```
class Solution {
public:
    double minmaxGasDist(vector<int>& stations, int K) {
        int max_dist = 0;
        for (int i = 1; i < stations.size(); ++i) {
            if (stations[i] - stations[i - 1] > max_dist) {
                max_dist = stations[i] - stations[i - 1];
            }
        }
        double left = 0, right = max_dist, D;
        while (right - left > 10e-7) {
            // cout << left << " " << right << endl;
            D = (left + right) / 2;
            if (judge(stations, K, D)) right = D;
            else left = D;
        }
        return D;
    }

    bool judge(vector<int>& stations, int K, double D) {
        for (int i = 1; i < stations.size(); ++i) {
            K = K - ceil((stations[i] - stations[i - 1]) / D) + 1;
            // cout << i << " " << K << endl;
            if (K < 0) return false;
        }
        return true;
    }
};
```

---
### 981. Time Based Key-Value Store
Create a timebased key-value store class TimeMap, that supports two operations.

1. set(string key, string value, int timestamp)

Stores the key and value, along with the given timestamp.
2. get(string key, int timestamp)

Returns a value such that set(key, value, timestamp_prev) was called previously, with timestamp_prev <= timestamp.
If there are multiple such values, it returns the one with the largest timestamp_prev.
If there are no values, it returns the empty string ("").

对每一个`key`保存每一个`timestamp`，由于题目中给定了时间片是递增的，所以`vector`直接插入即可，而查找时直接进行二分查找。

```
class TimeMap {
public:
    unordered_map<string, vector<string>> key2vals;
    unordered_map<string, vector<int>> key2times;
    TimeMap() {    
    }
    
    void set(string key, string value, int timestamp) {
        key2vals[key].push_back(value);
        key2times[key].push_back(timestamp);
    }
    
    string get(string key, int timestamp) {
        if (key2times.find(key) == key2times.end() || key2times[key][0] > timestamp) return "";
        int left = 0, right = key2times[key].size() - 1, middle;
        while (left < right) {
            middle = (left + right + 1) / 2;
            if (key2times[key][middle] == timestamp) return key2vals[key][middle];
            else if (key2times[key][middle] < timestamp) left = middle;
            else right = middle - 1;
        }
        return key2vals[key][left];
    }
};
```