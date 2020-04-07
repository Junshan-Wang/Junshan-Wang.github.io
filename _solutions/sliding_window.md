---
title: "Sliding Window"
collection: solutions
permalink: /solutions/sliding_window/
author_profile: true
---

## 滑动窗口的最大值/最小值数组
生成滑动窗口的最大值/最小值数组，可以直接作为题目，也可以间接作为预处理方法。
时间复杂度为$O(n)$。

构造一个双向队列，C++中使用数据结构`deque`。该队列保存的是在原数组中的索引；且该双向队列中的元素对应的值，在求最大值/最小值时分别是一个递减/递增的序列。

我们以求滑动窗口的最大值为例进行说明，依次遍历数组的每一个元素：

* 如果`deque`不为空，且队尾元素的值比当前元素的值小，则弹出队尾元素。重复该过程直到不满足条件，然后将当前元素的索引压入队尾。
* 如果队首和队尾的索引差值大于等于滑动窗口的大小，则说明此时队首应该不在滑动窗口内，所以弹出队首元素。
* 此时`deque`的队首对应的值就是当前滑动窗口的最大值。

### 239. Sliding Window Maximum

Given an array nums, there is a sliding window of size k which is moving from the very left of the array to the very right. You can only see the k numbers in the window. Each time the sliding window moves right by one position. Return the max sliding window.

Could you solve it in linear time?

最简单的直接计算是需要$O(nk)$的时间复杂度，当$k$和$n$一个数量级时，相当于$O(n^2)$的时间复杂度。

而用上述方法实现了$O(n)$的时间复杂度。

```
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;
    vector<int> res;
    for (int i = 0; i < nums.size(); ++i) {
        while (!dq.empty() && nums[dq.back()] <= nums[i])
            dq.pop_back();
        dq.push_back(i);
        if (i - dq.front() == k)
            dq.pop_front();
        if (i >= k - 1) {
            res.push_back(nums[dq.front()]);
        }
    }
    return res;    
}
```

### 683. K Empty Slots

You have `N` bulbs in a row numbered from `1` to `N`. Initially, all the bulbs are turned off. We turn on exactly one bulb everyday until all bulbs are on after `N` days.

You are given an array bulbs of length `N` where `bulbs[i] = x` means that on the `(i+1)th` day, we will turn on the bulb at position `x` where `i` is `0-indexed` and `x` is `1-indexed`.

Given an integer `K`, find out the minimum day number such that there exists two turned on bulbs that have exactly `K` bulbs between them that are all turned off.

If there isn't such day, return `-1`.


这一题可以有多种思路。

#### 暴力计数

最简单直接的办法，按照时间顺序依次考虑每一个花盆，分别往左和往右看是否满足条件。

* 时间复杂度：$O(n^2)$
* 但是很奇怪在实际运行中这种方法的用时竟然挺少的
  
```
int kEmptySlots(vector<int>& bulbs, int K) {
    int flag;
    vector<int> b(bulbs.size(), 0);
    for (int i = 0; i < bulbs.size(); ++i) {
        int pos = bulbs[i] - 1;
        b[pos] = 1;

        flag = 0;
        for (int j = 1; j <= K && pos - j >= 0; ++j) {
            if (b[pos - j] == 1) {
                flag = 1;
                break;
            }
        }
        if (!flag && pos > K && b[pos - K - 1])
            return i + 1;
        
        flag = 0;
        for (int j = 1; j <= K && pos + j < bulbs.size(); ++j) {
            if (b[pos + j] == 1) {
                flag = 1;
                break;
            }
        }
        if (!flag && pos + K + 1 < bulbs.size() && b[pos + K + 1])
            return i + 1;
    }
    return -1;
}
```

#### 插入到排序结构

官方给出的第一种简单方法，维护一个排序的数据结构，可以用C++中的`set`实现（红黑树）。按时间顺序依次考虑每一个花盆，检索到其需要插入的位置，判断它和它的邻居是否满足这一条件，如果满足则返回当前时间；不满足则将其插入到该数据结构中。

* 对于每一个元素，查找和插入都需要$O(\log n)$的复杂度，所以时间复杂度：$O(n \log n)$

```
class Solution {
public:
    int kEmptySlots(vector<int>& bulbs, int K) {
        set<int> s;
        for (int i = 0; i < bulbs.size(); ++i) {
            int bulb = bulbs[i];
            set<int>::iterator pos = s.insert(bulb).first;
            if (pos != s.begin() && bulb - (*(prev(pos))) - 1 == K) return i + 1;
            if (pos != s.end() && (*(next(pos))) - bulb - 1 == K) return i + 1;
            
        }
        return -1;
    }
};
```


#### 滑动窗口求最小值

注意到，任意连续的`K`个元素构成一个滑动窗口，该滑动窗口的最小时`min`，如果该滑动窗口的两个邻居都在`min`之前开花，则说明该滑动窗口满足条件，且事件发生的事件为两个邻居开花时间的最大值。

* 首先计算每个花盆的开花时间，复杂度为$O(n)$
* 计算每个滑动窗口的最小开花时间，即计算滑动窗口的最小值数组，复杂度为$O(n)$
* 然后判断每个滑动窗口及其邻居是否满足条件，并找出满足条件的滑动窗口中的最小值，复杂度为$O(n)$
* 时间复杂度为$O(n)$
  
```
int kEmptySlots(vector<int>& bulbs, int K) {
    vector<int> days(bulbs.size(), 0);
    vector<int> mins(bulbs.size(), 100000);
    deque<int> dq;
    int min_day = 100000;

    for (int i = 0; i < bulbs.size(); ++i)
        days[bulbs[i] - 1] = i;

    
    for (int i = 0; i < days.size(); ++i) {
        int cur = days[i];
        while (!dq.empty() && days[dq.back()] > cur)
            dq.pop_back();
        dq.push_back(i);
        if (i - dq.front() + 1 > K)
            dq.pop_front();
        if (!dq.empty())
            mins[i] = days[dq.front()];
    }

    for (int i = K; i < days.size() - 1; ++i) {
        if (days[i - K] < mins[i] && days[i + 1] < mins[i] && max(days[i - K], days[i + 1]) < min_day)
            min_day = max(days[i - K], days[i + 1]);
    }
    return min_day == 100000 ? -1 : min_day + 1;
}
```


---

## 给定条件的子字符串

### 340. Longest Substring with At Most K Distinct Characters

滑动窗口的应用（左右指针），同时需要用一个额外的数据结构统计每个字符的出现情况。

#### 哈希表（`unordered_map<character, count>`）
`right++`：如果出现了新字符，则`cnt++`，如果当前出现的次数`cnt`小于等于`k`，则重复上述过程；`left++`：如果某个字符的出现次数等于`0`，则`cnt++`，如果当前出现的次数`cnt`等于`k`，则重复上述过程。

```
int lengthOfLongestSubstringKDistinct(string s, int k) {
    unordered_map<char, int> cha2cnt;
    int left = 0, right = -1, cnt = 0, max_len = 0;
    if (s.size() == 0 || k == 0) return 0;
    while (right < (int)s.size()) {
        while (cnt <= k && (++right) < s.size()) {
            if (cha2cnt.find(s[right]) == cha2cnt.end() || cha2cnt[s[right]] == 0) {
                cnt++;
                cha2cnt[s[right]] = 0;
            }
            cha2cnt[s[right]]++;
        }
        if (right - left > max_len)
            max_len = right - left;
        while (cnt > k && left <= right) {
            if (cha2cnt[s[left]] == 1) {
                cnt--;
            }
            cha2cnt[s[left]]--;
            left++;
        }
    }
    return max_len;
}
```


#### 哈希表（`unordered_map<character, right_most_position>`）


#### 有序字典

### 76. Minimum Window Substring

Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).

题目没有说清楚，实际上是要求满足出现的字符及对应次数。

显然可以用两个指针遍历S，用三个hash表`unordered_map`分别保存T中出现的字符（提前处理）、S中出现的字符、S中出现的已经满足T中数量的字符。因此在每次修改指针时，需要判断S中满足条件的字符数和T中的字符数是否相等，同时更新S中出现的字符，以及可能更新S中满足条件的字符。

```
class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char, int> t2c, s2c, s2match;
        for (int i = 0; i < t.size(); ++i) {
            if (t2c.find(t[i]) == t2c.end()) t2c[t[i]] = 0;
            t2c[t[i]] += 1;
        }
        
        int i = 0, j = 0, min_len = s.size() + 1, min_i = -1;
        while (j < s.size()) {
            while (j < s.size() && s2match.size() < t2c.size()) {
                if (t2c.find(s[j]) != t2c.end()) {
                    if (s2c.find(s[j]) == s2c.end()) s2c[s[j]] = 0;
                    s2c[s[j]] += 1;
                    if (s2c[s[j]] >= t2c[s[j]]) s2match[s[j]] = 1;
                }
                j++;
            }
            while (i <= j && s2match.size() == t2c.size()) {
                if (j - i < min_len) {
                    min_len = j - i;
                    min_i = i;
                }
                if (t2c.find(s[i]) != t2c.end()) {
                    s2c[s[i]] -= 1;
                    if (s2c[s[i]] < t2c[s[i]]) s2match.erase(s[i]);
                }
                i++;
            }
        }
        if (min_i == -1) return "";
        else return s.substr(min_i, min_len);
    }

    bool satisfy(unordered_map<char, int>& t2c, unordered_map<char, int>& s2c) {
        for (unordered_map<char, int>::iterator it = t2c.begin(); it != t2c.end(); it++) {
            char key = it -> first;
            int cnt = it -> second;
            if (s2c.find(key) == s2c.end()) return false;
            else if (s2c[key] < cnt) return false;
        }
        return true;
    }
};
```
