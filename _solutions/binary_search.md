---
title: "Binary Search"
collection: solutions
permalink: /solutions/binary_search/
author_profile: true
---

二分查找的问题，往往关键是定义需要查找的结构是什么，且结构必须是排序的，然后查找过程是简单的，即通过三个指针`left`、`right`和`middle`迭代直至收敛。

C++中有三个二分查找的函数：`lower_bound(vec.begin(), vec.end(), val)`返回大于或等于`val`的第一个元素的位置，`upper_bound`返回大于`val`的第一个元素位置，和`binary_search`返回是否存在指定元素。

---

## 有序数组二分查找

该类型的题目往往是给定了有序递增/递减数组（比如时间等），然后往往要进行很多次查找，所以可以用二分查找进行加速。

### 528. Random Pick with Weight

Given an array w of positive integers, where w[i] describes the weight of index i, write a function pickIndex which randomly picks an index in proportion to its weight.

Note:

1 <= w.length <= 10000
1 <= w[i] <= 10^5
pickIndex will be called at most 10000 times.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/random-pick-with-weight


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

### 981. Time Based Key-Value Store
Create a timebased key-value store class TimeMap, that supports two operations.

1. set(string key, string value, int timestamp). Stores the key and value, along with the given timestamp.
2. get(string key, int timestamp)

Returns a value such that set(key, value, timestamp_prev) was called previously, with timestamp_prev <= timestamp.
If there are multiple such values, it returns the one with the largest timestamp_prev.
If there are no values, it returns the empty string ("").

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/time-based-key-value-store

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

### 911. Online Election
In an election, the i-th vote was cast for persons[i] at time times[i].

Now, we would like to implement the following query function: TopVotedCandidate.q(int t) will return the number of the person that was leading the election at time t.  

Votes cast at time t will count towards our query.  In the case of a tie, the most recent vote (among tied candidates) wins.

Note:  
1 <= persons.length = times.length <= 5000  
0 <= persons[i] <= persons.length  
times is a strictly increasing array with all elements in [0, 10^9].  
TopVotedCandidate.q is called at most 10000 times per test case.  
TopVotedCandidate.q(int t) is always called with t >= times[0].


来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/online-election

类似上述解法，需要注意预处理的时候，每次改变状态时将当前新的候选人和时间加入数组中。进行二分查找。

```
class TopVotedCandidate {
public:
    vector<int> ts;
    vector<int> ps;
    TopVotedCandidate(vector<int>& persons, vector<int>& times) {
        unordered_map<int, int> cnts;
        unordered_map<int, int> recs;
        int max_idx = -1;
        for (int i = 0; i < persons.size(); ++i) {
            cnts[persons[i]] += 1;
            recs[persons[i]] = times[i];
            
            if (max_idx == -1 || cnts[persons[i]] > cnts[max_idx] || (cnts[persons[i]] == cnts[max_idx] && recs[persons[i]] > recs[max_idx])) {
                max_idx = persons[i];
                ts.push_back(times[i]);
                ps.push_back(max_idx);
                // cout << times[i] << " " << max_idx << endl;
            }
        }
    }
    
    int q(int t) {
        int idx = upper_bound(ts.begin(), ts.end(), t) - ts.begin();
        return ps[idx - 1];
    }
};
```

### 1146. Snapshot Array

Implement a SnapshotArray that supports the following interface:

SnapshotArray(int length) initializes an array-like data structure with the given length.  Initially, each element equals 0.  
void set(index, val) sets the element at the given index to be equal to val.  
int snap() takes a snapshot of the array and returns the snap_id: the total number of times we called snap() minus 1.  
int get(index, snap_id) returns the value at the given index, at the time we took the snapshot with the given snap_id

来源：力扣（LeetCode）   
链接：https://leetcode-cn.com/problems/snapshot-array

直接在每个snapshot都保存一遍整个数组，内存开销过大。所以采用保存数组改变量的方法，只在数组发生改变时进行修改。这样的问题是查找时需要找到指定snapshot之前的结果，由于时间满足有序数组，所以可以用二分查找进行加速。

```
class SnapshotArray {
public:
    unordered_map<int, int> idx2val;
    vector<int> a;
    vector<vector<int>> snap_a;
    vector<vector<int>> snap_t;
    int t;
    SnapshotArray(int length) {
        a = vector<int>(length, 0);
        snap_a = vector<vector<int>>(length, vector<int>(0));
        snap_t = vector<vector<int>>(length, vector<int>(0));
        t = 0;
    }
    
    void set(int index, int val) {
        idx2val[index] = val;
    }
    
    int snap() {
        for (unordered_map<int, int>::iterator it = idx2val.begin(); it != idx2val.end(); it++) {
            int i = it -> first, val = it -> second;
            if (snap_a[i].size() == 0 || val != snap_a[i][snap_a[i].size() - 1]) {
                snap_a[i].push_back(val);
                snap_t[i].push_back(t);
            }
        }
        return t++; 
    }
    
    int get(int index, int snap_id) {
        int pos = upper_bound(snap_t[index].begin(), snap_t[index].end(), snap_id) - snap_t[index].begin() - 1;
        return pos == -1 ? 0 : snap_a[index][pos];
    }
};

/**
 * Your SnapshotArray object will be instantiated and called as such:
 * SnapshotArray* obj = new SnapshotArray(length);
 * obj->set(index,val);
 * int param_2 = obj->snap();
 * int param_3 = obj->get(index,snap_id);
 */
 ```



---

## 二分查找最优解

这一类题，查找最大值的最小值/最小值的最大值。其实可以有两种思路，动态规划和二分查找，哪种解法更好可以根据具体的数值范围进行分析。

二分法查找的思路是，在范围内搜索解，通过一个`judge`函数判断当前解是否合法，然后根据其返回值二分修改区间范围。

### 410. Split Array Largest Sum
Given an array which consists of non-negative integers and an integer m, you can split the array into m non-empty continuous subarrays. Write an algorithm to minimize the largest sum among these m subarrays.

Note:
If n is the length of array, assume the following constraints are satisfied:

1 ≤ n ≤ 1000，1 ≤ m ≤ min(50, n)

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/split-array-largest-sum

用二分查找+贪心的解法，找最大值的最小值，搜索下界为0，上界为所有元素求和。搜索时，依次遍历每一个元素，如果大于当前阈值，则划分一个新的子数组，直到所有元素遍历完或者已经有m大于阈值的子数组了。如果当前解满足条件，则解在当前解左边，移动右指针，否则移动左指针。时间复杂度是`O(n log sums[n-1])`。

```
class Solution {
public:
    int splitArray(vector<int>& nums, int m) {
        long long left = 0, right = 0, middle;
        for (int num : nums) right += (long long)num;
        while (left < right) {
            middle = (left + right) / 2;
            if (judge(nums, m, middle)) right = middle;
            else left = middle + 1;
        }
        return left;
    }
    bool judge(vector<int>& nums, int m, long long max_sum) {
        long long cur = 0;
        for (int num : nums) {
            cur += (long long)num;
            if (cur > max_sum) {
                cur = (long long)num;
                m--;
                if (cur > max_sum) return false;
                if (cur > 0 && m <= 0) return false;
            }
        }
        return true;
    }
};
```

用动态规划+前缀数组的解法，可以写出递归方程：`dp[i][j]=min(max(dp[i-1][k], sums[j]-sums[k]), k=0,...j-1)`，其中`sums[i]=nums[0]+...+nums[i]`，初始化为`dp[0][j]=sums[j]`。观察到`dp[i]`只和`dp[i-1]`有关，所以可以只用一维数组，每一轮遍历需要从后往前。时间复杂度为`O(n^2m)`，空间复杂度为`O(n)`。

```
class Solution {
public:
    int splitArray(vector<int>& nums, int m) {
        int n = nums.size();
        vector<long long> sums(n, 0);
        vector<long long> dp(n, 0);
        for (int i = 0; i < n; ++i) {
            sums[i] = nums[i] + (i > 0 ? sums[i - 1] : 0);
            dp[i] = sums[i];
        }
        for (int i = 1; i < m; ++i) {
            for (int j = n - 1; j >= 0; --j) {
                for (int k = 0; k < j; ++k) {
                    long long max_s = max(dp[k], sums[j] - sums[k]);
                    if (max_s < dp[j]) dp[j] = max_s;
                }
            }
        }
        return dp[n - 1];
    }
};
```

分析一下在这一题中两种解的时间开销，二分查找约为`n log sums[n-1] = 10^3 log 2^32 = 10^5`，动态规划约为`n^2m = 10^7`，所以二分查找更快。

### 1231. Divide Chocolate
You have one chocolate bar that consists of some chunks. Each chunk has its own sweetness given by the array sweetness.

You want to share the chocolate with your K friends so you start cutting the chocolate bar into K+1 pieces using K cuts, each piece consists of some consecutive chunks.

Being generous, you will eat the piece with the minimum total sweetness and give the other pieces to your friends.

Find the maximum total sweetness of the piece you can get by cutting the chocolate bar optimally.

Constraints:

0 <= K < sweetness.length <= 10^4，1 <= sweetness[i] <= 10^5

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/divide-chocolate

和上一题类似，不过最优解相反，求的是最小值的最大值。用二分法做的时间开销约为`n log sums[n-1] = 10^4 log 10^4*10^5 = 10^5`，而用动态规划的时间开销约为`n^2K=10^13`，所以这一题用动态规划会超时，只能用二分法求解。

```
class Solution {
public:
    int maximizeSweetness(vector<int>& sweetness, int K) {
        int left = 0, right = accumulate(sweetness.begin(), sweetness.end(), 0), middle;
        while (left < right) {
            middle = (left + right + 1) / 2;
            if (judge(sweetness, K, middle)) left = middle;
            else right = middle - 1;
        }
        return left;
    }
    bool judge(vector<int>& sweetness, int K, int min_sweet) {
        int cur = 0;
        for (int sweet : sweetness) {
            cur += sweet;
            if (cur >= min_sweet) {
                cur = 0;
                if (K <= 0) return true;
                else K--;
            }
        }
        return false;
    }
};
```

### 774. Minimize Max Distance to Gas Station
On a horizontal number line, we have gas stations at positions stations[0], stations[1], ..., stations[N-1], where N = stations.length.

Now, we add K more gas stations so that D, the maximum distance between adjacent gas stations, is minimized.

Return the smallest possible value of D.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/minimize-max-distance-to-gas-station

查找最小的可能距离`D`。对于每一距离`d`，我们可以检查它是否满足条件，即是否能保证新建`K`个加油站后彼此之间距离小于`d`。如果满足条件，则将`d`缩小，从而寻找更小的可能`d`；如果不满足条件，则放大`d`，直到找到可能的`d`。而这个放大缩小的过程可以用二分查找的思路。由于此时查找的不再是整数，而是浮点数，所以停止的条件是左右指针之间的距离小于给定误差范围。时间复杂度为`O(N log station[N-1])=10^4`。

其实这一题还可以用贪心法（`O(NK)=10^9`或用堆加速后`O(K logN)=10^6`），或者动态规划（`O(NK^2)=10^15`）的方法去做，不过会超时。

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
            D = (left + right) / 2;
            if (judge(stations, K, D)) right = D;
            else left = D;
        }
        return D;
    }

    bool judge(vector<int>& stations, int K, double D) {
        for (int i = 1; i < stations.size(); ++i) {
            K = K - ceil((stations[i] - stations[i - 1]) / D) + 1;
            if (K < 0) return false;
        }
        return true;
    }
};
```

