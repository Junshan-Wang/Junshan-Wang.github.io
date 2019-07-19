---
title: "#56. Merge Intervals"
collection: solutions
permalink: /solutions/merge_interval/
author_profile: true
---

Given a collection of intervals, merge all overlapping intervals.

Example 1:
```
Input: [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].
```

Example 2:
```
Input: [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considered overlapping.
```

From LeetCode

---

最暴力的方法是对每一个区间，都遍历一次其他区间观测是否能够合并，一共需要`O(n^2)`的时间复杂度。应该可以用树的方法保存每一段，然后进行更新，但是比较麻烦。

用贪心算法可以将时间复杂度降到`O(nlogn)`。先将区间按照起始位置进行升序排序，注意到如果第$i$个区间的起始位置小于等于第$i-1$个区间的末尾位置，则说明它们可以合并；同时如果第$i$个区间不能与第$i-1$个区间合并，由于$end[i-1]<start[i]<start[i+k]$，$i$以后的区间都无法和第$i-1$个区间合并，即考虑第$i$个区间时只需要比较它和前一个已经合并好的区间进行比较即可。所以我们只需要在排序后遍历从小到大遍历每一个区间，合并相邻两个区间，或者创建一个新的区间。

```
bool vectorCompare(const vector<int>& a, const vector<int>& b) {
    return a[0] < b[0];
}

class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        vector<vector<int>> res(0);
        sort(intervals.begin(), intervals.end(), vectorCompare);
        for (auto& interval : intervals) {
            if (res.empty() || res[res.size() - 1][1] < interval[0]) 
                res.push_back(interval);
            else if (res[res.size() - 1][1] < interval[1])
                res[res.size() - 1][1] = interval[1];
        }
        return res;
    }
};
```