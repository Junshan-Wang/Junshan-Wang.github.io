---
title: "Greedy Algorithm"
collection: solutions
permalink: /solutions/greedy_algorithm/
author_profile: true
---

### 1007. Minimum Domino Rotations For Equal Row

In a row of dominoes, A[i] and B[i] represent the top and bottom halves of the i-th domino.  (A domino is a tile with two numbers from 1 to 6 - one on each half of the tile.)

We may rotate the i-th domino, so that A[i] and B[i] swap values.

Return the minimum number of rotations so that all the values in A are the same, or all the values in B are the same.

If it cannot be done, return -1.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/minimum-domino-rotations-for-equal-row

这种题思路比较简单，其实不需要枚举所有情况`O(2^n)`，只需要根据第一张牌的情况，就可以确定后面所有牌的操作，实际上只有四种可能，只需要`O(n)`的复杂度。有很多类似的题目，分析清楚后面的操作是由第一个操作决定的即可。

```
class Solution {
public:
    int minDominoRotations(vector<int>& A, vector<int>& B) {
        int cnts[4] = {0, 1, 0, 1};
        for (int i = 1; i < A.size(); ++i) {
            if (cnts[0] != INT_MAX) {
                if (A[i] == A[0]);
                else if (B[i] == A[0]) cnts[0]++;
                else cnts[0] = INT_MAX;
            }
            if (cnts[1] != INT_MAX) {
                if (B[i] == A[0]);
                else if (A[i] == A[0]) cnts[1]++;
                else cnts[1] = INT_MAX;
            }
            if (cnts[2] != INT_MAX) {
                if (B[i] == B[0]);
                else if (A[i] == B[0]) cnts[2]++;
                else cnts[2] = INT_MAX;
            }
            if (cnts[3] != INT_MAX) {
                if (A[i] == B[0]);
                else if (B[i] == B[0]) cnts[3]++;
                else cnts[3] = INT_MAX;
            }
            if (cnts[0] == INT_MAX && cnts[1] == INT_MAX && cnts[2] == INT_MAX && cnts[3] == INT_MAX) return -1;
        }
        return min(min(cnts[0], cnts[1]), min(cnts[2], cnts[3]));
    }
};
```

### 1055. Shortest Way to Form String

From any string, we can form a subsequence of that string by deleting some number of characters (possibly no deletions).

Given two strings source and target, return the minimum number of subsequences of source such that their concatenation equals target. If the task is impossible, return -1.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/shortest-way-to-form-string

贪心的关键是，注意到从`target[i]`出发，肯定是能一次匹配越多字符越好，显然如果某个字符不匹配完而留给下一次匹配，那么这种情况需要的次数肯定大于等于贪心需要的次数。同时当且仅当某个`target`中的字符在`source`中没找到时，匹配任务失败。

```
class Solution {
public:
    int shortestWay(string source, string target) {
        int cnt = 0;
        for (int i = 0; i < target.size(); ) {
            int k = 0;
            for (int j = 0; j < source.size() && i + k < target.size(); ++j) {
                if (source[j] == target[i + k]) k++;
            }
            if (k == 0) return -1;
            i = i + k;
            cnt++;
        }
        return cnt;
    }
};
```

### 358. Rearrange String k Distance Apart

Given a non-empty string s and an integer k, rearrange the string such that the same characters are at least distance k from each other.

All input strings are given in lowercase letters. If it is not possible to rearrange the string, return an empty string "".

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/rearrange-string-k-distance-apart

这道题显然需要对字符出现的次数进行统计和排序。  
一开始的想法是把出现最多的字符每隔k位安排，但是这样在某些情况下会出错，导致不满足题目要求。  
解法应该是每个k个字符一组安排，而这k个字符应该是当前出现次数最多的k个字符。用最大堆来实现，每次取最大k个，写入后将它们出现的次数减一后再压入堆中。这样能保证重新安排的字符串是满足条件的：每k个字符互不相同，而相邻k个字符由于顺序关系，也至少差k。同时这样能保证一定能找到：因为如果不是找最大的k个而是找另外k个，则剩下的字符串更集中于某几个字符，如果这样能找到解，则贪心算法一定也能找到解。此外，无解的条件是剩下的字符里找不到k个剩余次数大于0的字符了。


```
class Solution {
public:
    string rearrangeString(string s, int k) {
        vector<int> cha2cnt(26, 0);
        priority_queue<pair<int, int>, vector<pair<int, int>>, less<pair<int, int>>> cnt_cha;
        string ans(s);
        int l = 0;
        if (k == 0) return s;
        for (int i = 0; i < s.size(); ++i) cha2cnt[(int)(s[i] - 'a')]++;
        for (int i = 0; i < cha2cnt.size(); ++i) cnt_cha.push(make_pair(cha2cnt[i], i));
        for (int i = 0; i < s.size(); i += k) {
            vector<pair<int, int>> tmp;
            for (int j = 0; j < k && l < s.size(); ++j) {
                pair<int, int> t = cnt_cha.top();
                cnt_cha.pop();
                if (t.first == 0) return "";
                t.first--;
                tmp.push_back(t);
                ans[l++] = (char)(t.second + 'a');
            }
            for (int j = 0; j < tmp.size(); ++j) {
                cnt_cha.push(tmp[j]);
            }
        }
        return ans;
    }
};
```