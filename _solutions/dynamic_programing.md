---
title: "Dynamic Programing"
collection: solutions
permalink: /solutions/dynamic_programing/
author_profile: true
---

### 418. Sentence Screen Fitting

Given a rows x cols screen and a sentence represented by a list of non-empty words, find how many times the given sentence can be fitted on the screen.

Note:

* A word cannot be split into two lines.
* The order of words in the sentence must remain unchanged.
* Two consecutive words in a line must be separated by a single space.
* Total words in the sentence won't exceed 100.
* Length of each word is greater than 0 and won't exceed 10.
* 1 ≤ rows, cols ≤ 20,000.


来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/sentence-screen-fitting

我觉得不能算动态规划，实际上就是预处理，维护一个memory，保存的是在一行内以每个单词开头能够写下的单词数。然后再每行计算，复杂度是O(n)。

```
class Solution {
public:
    int wordsTyping(vector<string>& sentence, int rows, int cols) {
        vector<int> dp(sentence.size(), 0);
        int cnt = 0, p = 0;
        for (int i = 0; i < sentence.size(); ++i) {
            int j = 0, k = i;
            while (j + sentence[k].size() <= cols) {
                j = j + sentence[k].size() + 1;
                k = (k + 1) % sentence.size();
                dp[i]++;
            }
        }
        for (int i = 0; i < rows; ++i) {
            cnt += dp[p];
            p = (p + dp[p]) % sentence.size();
        }
        return cnt / sentence.size();
    }
};
```

### 361. Bomb Enemy

Given a 2D grid, each cell is either a wall 'W', an enemy 'E' or empty '0' (the number zero), return the maximum enemies you can kill using one bomb.  
The bomb kills all the enemies in the same row and column from the planted point until it hits the wall since the wall is too strong to be destroyed.  
Note: You can only put the bomb at an empty cell.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/bomb-enemy

暴力算法的复杂度是O(n^3)，可以将问题分解成四个子问题，即对于每一个格子，它上/下/左/右分别有多少个敌人，而每个子问题都可以用动态规划的思路解决，即当前格子的敌人数只和上一个格子有关：
* `dp[i] = dp[i - 1] + 1 if grid[i][j] == 'E'` 
* `dp[i] = dp[i - 1] if grid[i][j] == '0'`
* `dp[i] = 0 if grid[i][j] == 'W'`
  
因此在O(1)的时间复杂度完成计算，总的时间复杂度为O(n^2)。

```
class Solution {
public:
    int maxKilledEnemies(vector<vector<char>>& grid) {
        int m = grid.size(); 
        if (m == 0) return 0;
        int n = grid[0].size();
        int ans = 0;
        vector<vector<int>> up(m, vector<int>(n, 0)), down(m, vector<int>(n, 0));
        vector<vector<int>> up_left(m, vector<int>(n, 0)), down_right(m, vector<int>(n, 0));
        for (int i = 0; i < m; ++i) {
            int cnt = 0;
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 'W') {
                    cnt = 0;
                    up[i][j] = 0;
                }
                else if (grid[i][j] == 'E') {
                    cnt++;
                    up[i][j] = 1 + (i > 0 ? up[i - 1][j] : 0);
                }
                else {
                    up[i][j] = (i > 0 ? up[i - 1][j] : 0);
                }
                up_left[i][j] = up[i][j] + cnt;
            }
        }
        for (int i = m - 1; i >= 0; --i) {
            int cnt = 0;
            for (int j = n - 1; j >= 0; --j) {
                if (grid[i][j] == 'W') {
                    cnt = 0;
                    down[i][j] = 0;
                }
                else if (grid[i][j] == 'E') {
                    cnt++;
                    down[i][j] = 1 + (i < m - 1 ? down[i + 1][j] : 0);
                }
                else {
                    down[i][j] = (i < m - 1 ? down[i + 1][j] : 0);
                }
                down_right[i][j] = down[i][j] + cnt;
            }
        }
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j) 
                if (grid[i][j] == '0') {
                    ans = max(ans, up_left[i][j] + down_right[i][j]);
                }
        return ans;
    }
};
```

### 750. Number Of Corner Rectangles

Given a grid where each entry is only 0 or 1, find the number of corner rectangles.

A corner rectangle is 4 distinct 1s on the grid that form an axis-aligned rectangle. Note that only the corners need to have the value 1. Also, all four 1s used must be distinct.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/number-of-corner-rectangles

直接暴力搜索的时间复杂度为O(n^4)。

`dp[j][k]`保存从第j列到第k列，同时出现的1的行数，而以当前行的j和k作为下边界所能构成的长方形数量，刚好是当前行以上的j和k同时为1的行数。每一行遍历
* `count = count + dp[j][j]`
* `dp[j][k] = dp[j][k] + 1 if grid[i][j] == 1 and grid[i][k] ==1`

时间复杂度为O(n^3)。

这种题型的动态规划需要仔细设计递推方程，在这里，dp[i][j]不再是简单的表示第i行第j列的情况，而是表示第i列到第j列的情况。

```
class Solution {
public:
    int countCornerRectangles(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size(), ans = 0;
        vector<vector<int>> dp(n, vector<int>(n, 0));
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                for (int k = j + 1; k < n; ++k) {
                    if (grid[i][j] == 1 && grid[i][k] == 1) {
                        ans += dp[j][k];
                        dp[j][k] += 1;
                    }
                }
            }
        }
        return ans;
    }
};
```

### 1066. Campus Bikes II

On a campus represented as a 2D grid, there are N workers and M bikes, with N <= M. Each worker and bike is a 2D coordinate on this grid.

We assign one unique bike to each worker so that the sum of the Manhattan distances between each worker and their assigned bike is minimized.

The Manhattan distance between two points p1 and p2 is Manhattan(p1, p2) = |p1.x - p2.x| + |p1.y - p2.y|.

Return the minimum possible sum of Manhattan distances between each worker and their assigned bike.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/campus-bikes-ii

直接暴力搜索需要O(n!m!)的时间复杂度。但是考虑这个过程中有重复计算的情况，实际上需要枚举的情况有O(2^n * 2^m)，其实复杂度还是很高好吧。

用位压缩的方法减少空间开销。i在二进制下的第k位表示第k个worker是否有被考虑，i表示的是在二进制下为1的位数的worker已经被考虑的一种组合，dp[i][j]表示的是在i和j的worker和bike的组合下，需要的最小距离，递推方程为
* `dp[i][j] = min(dp[i][j], dis[k][l] + dp[i ^ (1 << k)][j ^ (1 << l)])`，注意k和l需要在i和j中对应位置存在

上述方程直接计算还是会超时，需要进行剪枝。最后结果是把worker全部考虑完后，任意一种bike组合的最小值。

```
class Solution {
public:
    int assignBikes(vector<vector<int>>& workers, vector<vector<int>>& bikes) {
        int n = workers.size(), m = bikes.size();
        int ans = 100000;
        vector<vector<int>> dp(1 << n, vector<int>(1 << m, 100000));
        vector<vector<int>> dis(n, vector<int>(m, 0));
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < m; ++j) {
                dis[i][j] = abs(workers[i][0] - bikes[j][0]) + abs(workers[i][1] - bikes[j][1]);
            }
        }
        for (int j = 0; j < dp[0].size(); ++j) {
            dp[0][j] = 0;
        }
        for (int i = 1; i < dp.size(); ++i) {
            for (int j = 1; j < dp[0].size(); ++j) {
                for (int k = 0; k < n; ++k) {
                    if (!(i & (1 << k))) continue;
                    for (int l = 0; l < m; ++l) {
                        if (!(j & (1 << l))) continue;
                        int d = dis[k][l] + dp[i ^ (1 << k)][j ^ (1 << l)];
                        dp[i][j] = min(dp[i][j], d);
                    }
                }
            }
        }
        for (int j = 0; j < dp[0].size(); ++j) {
            ans = min(ans, dp[dp.size() - 1][j]);
        }
        return ans;
    }
};
```

### 568. Maximum Vacation Days

LeetCode wants to give one of its best employees the option to travel among N cities to collect algorithm problems. But all work and no play makes Jack a dull boy, you could take vacations in some particular cities and weeks. Your job is to schedule the traveling to maximize the number of vacation days you could take, but there are certain rules and restrictions you need to follow.

Rules and restrictions:

You can only travel among N cities, represented by indexes from 0 to N-1. Initially, you are in the city indexed 0 on Monday.  
The cities are connected by flights. The flights are represented as a N*N matrix (not necessary symmetrical), called flights representing the airline status from the city i to the city j. If there is no flight from the city i to the city j, flights[i][j] = 0; Otherwise, flights[i][j] = 1. Also, flights[i][i] = 0 for all i.  
You totally have K weeks (each week has 7 days) to travel. You can only take flights at most once per day and can only take flights on each week's Monday morning. Since flight time is so short, we don't consider the impact of flight time.  
For each city, you can only have restricted vacation days in different weeks, given an N*K matrix called days representing this relationship. For the value of days[i][j], it represents the maximum days you could take vacation in the city i in the week j.  
You're given the flights matrix and days matrix, and you need to output the maximum vacation days you could take during K weeks.


来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/maximum-vacation-days

`dp[k][i]`表示第k天时位于第i个城市，所能达到的最大值，需要注意判断哪些城市j在第k-1天能到达，且哪些城市j能飞往i，并考虑停留在同一个城市的情况，递推方程为
* `dp[k][i] = max(dp[k][i], dp[k - 1][j] + days[i][k - 1])`

时间复杂度为O(KN^2)。

```
class Solution {
public:
    int maxVacationDays(vector<vector<int>>& flights, vector<vector<int>>& days) {
        int N = flights.size(), K = days[0].size();
        int ans = 0;
        vector<vector<int>> dp(K + 1, vector<int>(N, -1));
        dp[0][0] = 0;
        for (int k = 1; k <= K; ++k) {
            for (int i = 0; i < N; ++i) {
                for (int j = 0; j < N; ++j) {
                    if ((flights[j][i] > 0 || i == j) && dp[k - 1][j] >= 0) {
                        dp[k][i] = max(dp[k][i], dp[k - 1][j] + days[i][k - 1]);
                    }
                }
            }
        }
        for (int i = 0; i < N; ++i)
            ans = max(ans, dp[K][i]);
        return ans;
    }
};
```

### 471. Encode String with Shortest Length

Given a non-empty string, encode the string such that its encoded length is the shortest.

The encoding rule is: k[encoded_string], where the encoded_string inside the square brackets is being repeated exactly k times.

Note:

k will be a positive integer and encoded string will not be empty or have extra space.  
You may assume that the input string contains only lowercase English letters. The string's length is at most 160.  
If an encoding process does not make the string shorter, then do not encode it. If there are several solutions, return any of them is fine.


来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/encode-string-with-shortest-length

这题比较复杂，`dp[i][i + j]`表示从第i个字符开始长度为j的子串的最短编码。
* 首先，如果子串可以分成独立的两部分，则`dp[i][i + j] = min(dp[i][i + k] + dp[i + k][i + j]), k < j`。
* 然后考虑如果整个子串可以统一编码，即存在t使得子串正好重复t一定次数，关键是如何判断是否有t，且如何找到。用到一个技巧，令T = s[i, i + j] + s[i, i + j]，调用find函数在T中查找第二次出现的s[i, i + j]，如果刚好是第j个位置，则说明s无重复子串t，否则如果位置`p<j`，则说明s有重复子串t，且t刚好就是s[i, i + p]，t在子串中出现的次数是j / p。

```
class Solution {
public:
    string encode(string s) {
        int n = s.size();
        vector<vector<string>> dp(n, vector<string>(n + 1, ""));
        for (int j = 1; j <= n; ++j) {
            for (int i = 0; i + j <= n; ++i) {
                dp[i][i + j] = s.substr(i, j);
                if (j < 5) continue;
                string tmp = (dp[i][i + j] + dp[i][i + j]).substr(0, j);
                int p = (dp[i][i + j] + dp[i][i + j]).find(tmp, 1);
                if (p < j) 
                    dp[i][i + j] = to_string(j / p) + "[" + dp[i][i + p] + "]";
                for (int k = i + 1; k < i + j; ++k)
                    if (dp[i][k].size() + dp[k][i + j].size() < dp[i][i + j].size())
                        dp[i][i + j] = dp[i][k] + dp[k][i + j];
            }
        }
        return dp[0][n];
    }
};
```

### 403. Frog Jump

A frog is crossing a river. The river is divided into x units and at each unit there may or may not exist a stone. The frog can jump on a stone, but it must not jump into the water.

Given a list of stones' positions (in units) in sorted ascending order, determine if the frog is able to cross the river by landing on the last stone. Initially, the frog is on the first stone and assume the first jump must be 1 unit.

If the frog's last jump was k units, then its next jump must be either k - 1, k, or k + 1 units. Note that the frog can only jump in the forward direction.

Note:  
The number of stones is ≥ 2 and is < 1,100.  
Each stone's position will be a non-negative integer < 231.  
The first stone's position is always 0.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/frog-jump

直接暴力搜索的复杂度是O(3^n)。

考虑到🐸只能往前跳，并且石头数量较少，且每个时刻能够跳的距离k最多为石头数量，因此可以用动态规划求解。`dp[i][k]`表示是否能在最后一跳以k的距离到达i位置。

* `dp[i][k] = dp[i - k][k - 1] | dp[i - k][k] | dp[i - k][k + 1]`。



```
class Solution {
public:
    bool canCross(vector<int>& stones) {
        unordered_map<int, vector<bool>> dp;
        dp[0] = vector<bool>{true};
        for (int i : stones) {
            if (i == 0) continue;
            int cap = dp.size() + 1;
            dp[i] = vector<bool>(cap, false);
            for (int k = 1; k < cap && k <= i / 2 + 1; ++k) {
                if (dp.find(i - k) != dp.end()) {
                    if (k < dp[i - k].size()) dp[i][k] = dp[i][k] | dp[i - k][k];
                    if (k - 1 < dp[i - k].size()) dp[i][k] = dp[i][k] | dp[i - k][k - 1];
                    if (k + 1 < dp[i - k].size()) dp[i][k] = dp[i][k] | dp[i - k][k + 1];
                }
            }
        }
        for (int k = 1; k < dp[*stones.rbegin()].size(); ++k) {
            if (dp[*stones.rbegin()][k]) return true;
        }
        return false;
    }
};
```

### 139. Word Break

Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, determine if s can be segmented into a space-separated sequence of one or more dictionary words.

Note:

The same word in the dictionary may be reused multiple times in the segmentation.  
You may assume the dictionary does not contain duplicate words.


暴力搜索的复杂度是指数级别的，在当前字符处，只需要考虑以当前字符作为结尾构成一个单词，是否能够划分，复杂度是O(n^2)

* `dp[i] = max(dp[i - size(word[j])] && s[i, size(word[j])] == word[j], j=1, ..., n)`
  
```
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        vector<bool> dp(s.size(), false);
        for (int i = 0; i < s.size(); ++i) {
            for (string word : wordDict) {
                if (word.size() > i + 1) continue;
                bool v = (i + 1 == word.size() || dp[i - word.size()]) && (word == s.substr(i + 1 - word.size(), word.size()));
                dp[i] = dp[i] || v;
                if (dp[i]) break;
            }
        }
        return dp[s.size() - 1];
    }
};
```

### 42. Trapping Rain Water 

Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

对于每一个柱体，需要分别找到它左侧和右侧的最高点，两者的最小值减去当前柱体的高度为当前位置的储水量，但是简单的搜索需要O(n^2)的时间复杂度。  
采用动态规划的思想，进行预处理，用总共O(n)的复杂度分别计算每一个柱体左边和右边的最高点。  
进一步优化，实际上不需要额外的空间复杂度，只需要找到最高点位置，然后从左边到右遍历（直到到达最高点），用一个数保存当前最高点，则对于每一个位置而言它的储水量就是当前最高点（因为右侧比它高的位置是全局最高点）减去它的高度。

```
class Solution {
public:
    int trap(vector<int>& height) {
        int sum = 0, mid = 0, cur = 0;
        for (int i = 0; i < height.size(); ++i) if (height[i] > height[mid]) mid = i;
        for (int i = 0; i < mid; ++i) {
            cur = max(cur, height[i]); 
            sum += cur - height[i];
        }
        cur = 0;
        for (int i = height.size() - 1; i > mid; --i) {
            cur = max(cur, height[i]); 
            sum += cur - height[i];
        }
        return sum;
    }
};
```

这一题还可以用栈来做，不过不再是计算每一个位置的储水量，而是每一个高度的储水量。

```
class Solution {
public:
    int trap(vector<int>& height) {
        stack<pair<int, int>> st;
        int sum = 0;
        for (int i = 0; i < height.size(); ++i) {
            int down = 0;
            while (!st.empty() && st.top().second <= height[i]) {
                sum += (i - st.top().first - 1) * (st.top().second - down);
                down = st.top().second;
                st.pop();
            }
            if (!st.empty()) sum += (i - st.top().first - 1) * (height[i] - down);
            st.push(make_pair(i, height[i]));
        }
        return sum;
    }
};
```

### 312. Burst Balloons

Given n balloons, indexed from 0 to n-1. Each balloon is painted with a number on it represented by array nums. You are asked to burst all the balloons. If the you burst balloon i you will get nums[left] * nums[i] * nums[right] coins. Here left and right are adjacent indices of i. After the burst, the left and right then becomes adjacent.

Find the maximum coins you can collect by bursting the balloons wisely.

Note:

You may imagine nums[-1] = nums[n] = 1. They are not real therefore you can not burst them.  
0 ≤ n ≤ 500, 0 ≤ nums[i] ≤ 100

暴力求解的复杂度是指数级别的，很显然用动态规划方法。`dp[i][i + k]`表示在i和i+k的区间内（不包括两个端点，一开始没理清楚，发现要不包括两端点才能计算）能够计算出的最大值，需要遍历`(i,i+k)`内的每一个j，作为*最后计算的值*（这里一开始没理清楚是最先计算还是最后计算，但是发现最先计算会无法计算左右两个区间）。
* `dp[i][i + k] = max(dp[i][j] + dp[j][i + k] + nums[i] * nums[j] * nums[i + k], j = i + 1, ..., i + k - 1)`

```
class Solution {
public:
    int maxCoins(vector<int>& nums) {
        int n = nums.size();
        nums.insert(nums.begin(), 1);
        nums.insert(nums.end(), 1);
        vector<vector<int>> dp(n + 2, vector<int>(n + 2, 0));
        for (int k = 2; k <= n + 1; ++k) {
            for (int i = 0; i + k <= n + 1; ++i) {
                for (int j = i + 1; j < i + k; ++j) {
                    dp[i][i + k] = max(dp[i][i + k], dp[i][j] + dp[j][i + k] + nums[i] * nums[j] * nums[i + k]);
                }
            }
        }
        return dp[0][n + 1];
    }
};
```


### 240. Search a 2D Matrix II

Write an efficient algorithm that searches for a value in an m x n matrix. This matrix has the following properties:

Integers in each row are sorted in ascending from left to right.
Integers in each column are sorted in ascending from top to bottom.
Example:

Consider the following matrix:

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

Given target = `5`, return `true.`

Given target = `20`, return `false`.

from Leetcode

在一个排好序的一维数组中查找是否存在某个数，用二分查找的时间复杂度是`O(logn)`，即一个节点数为`n`的二叉树，查找的复杂度为从根节点到一个叶子结点的路径长度，即树的高度`logn`。

扩展到二维数组上时，在每一步将数组划分成左上、左下、右上、右下四个部分，对于每个部分，如果target小于左上角的数或者大于右上角的数，则返回false，否则继续划分。但是此时复杂度是`O(mn)`？因为此时是一个节点数为`mn`的4叉树，且查找的复杂度不再是一条路径，在每个非叶子结点处最多会有三个子节点继续搜索。

所以这里用的应该是*动态规划*的思想，从右上角开始遍历，当遍历到`matrix[i][j]`时，我们已知`target`不在`matrix[0:i][0:n]`和`matrix[0:m][j+1:n]`中，即只有可能在当前位置的左下角`matrix[i:m][0:j+1]`。此时
* 如果`matrix[i][j]>target`，则`target`不可能在`matrix[i:m][j]`，所以继续搜索`matrix[i][j-1]`
* 如果`matrix[i][j]<target`，则`target`不可能在`matrix[i][0:j+1]`，所以继续搜索`matrix[i+1][j]`

搜索直到`i>=m`或者`j<0`，从右上走到左下，时间复杂度为`O(m+n)`。

```
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int size1 = matrix.size();
        if (size1 == 0)
            return false;
        int size2 = matrix[0].size();
        int i = 0;
        int j = size2 - 1;
        while (i < size1 && j >= 0) {
            if (target == matrix[i][j])
                return true;
            else if (target < matrix[i][j])
                j--;
            else 
                i++;
        }
        return false;
    }
};
```

### 5391. Build Array Where You Can Find The Maximum Exactly K Comparisons

Given three integers n, m and k. Consider the following algorithm to find the maximum element of an array of positive integers:

You should build the array arr which has the following properties:

arr has exactly n integers.
1 <= arr[i] <= m where (0 <= i < n).
After applying the mentioned algorithm to arr, the value search_cost is equal to k.
Return the number of ways to build the array arr under the mentioned conditions. As the answer may grow large, the answer must be computed modulo 10^9 + 7.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/build-array-where-you-can-find-the-maximum-exactly-k-comparisons

```
class Solution {
public:
    int numOfArrays(int n, int m, int k) {
        vector<vector<vector<long long>>> dp(n + 1, vector<vector<long long>>(m + 1, vector<long long>(k + 1, 0)));
        if (k == 0 || k > m) return 0;
        for (int j = 1; j <= m; ++j) dp[1][j][1] = 1;
        for (int i = 2; i <= n; ++i) {
            for (int j = 1; j <= m; ++j) {
                for (int l = 1; l <= k; ++l) {
                    dp[i][j][l] = j * dp[i - 1][j][l] % 1000000007;
                    for (int t = 1; t < j; ++t) dp[i][j][l] = (dp[i][j][l] + dp[i - 1][t][l - 1]) % 1000000007;
                }
            }
        }
        long long res = 0;
        for (int j = 1; j <= m; ++j) res = (res + dp[n][j][k]) % 1000000007;
        return res;
    }
};
```

### 140. Word Break II

Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, add spaces in s to construct a sentence where each word is a valid dictionary word. Return all such possible sentences.

Note:

The same word in the dictionary may be reused multiple times in the segmentation.
You may assume the dictionary does not contain duplicate words.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/word-break-ii

直接暴力搜索的方法复杂度是O(n!)，用trie树也会超时。
如果只是判断是否存在合法分割，则只需要O(n^2)的复杂度，但是因为要返回所有可能的集合，用记忆化搜索(也可以改为动态规划）的方法，避免重复计算，复杂度为O(n^3)。  
注意到一个特例无法通过，所以先用O(n^2)复杂度的方法判断是否存在合法分割。

```
class Solution {
public:
    vector<vector<string>> dp;
    unordered_set<string> wordSet;
    int max_len = 0;
    vector<string> wordBreak(string s, vector<string>& wordDict) {
        dp = vector<vector<string>>(s.size(), vector<string>(0));
        dp.push_back(vector<string>(1, ""));
        for (string word : wordDict) wordSet.insert(word);

        if (wordDict.size() == 0) return vector<string>(0);
        vector<int> reach(s.size(), 0);
        for(int i = 0; i < s.size(); ++i){
            for(int j = i; j >= 0; --j){
                if((j == 0 || reach[j - 1] == 1) && wordSet.find(s.substr(j, i - j + 1)) != wordSet.end()){
                    reach[i] = 1;
                    break;
                }
            }
        }
        if (reach[s.size() - 1] == 0) return vector<string>(0);

        dfs(s, 0);
        vector<string> ans(0);
        for (string t : dp[0]) ans.push_back(t.substr(0, t.size() - 1));
        return ans;
    }

    void dfs(string s, int i) {
        if (dp[i].size() > 0) return;
        for (int j = i + 1; j <= s.size(); ++j) {
            string word = s.substr(i, j - i);
            if (wordSet.find(word) != wordSet.end()) {
                dfs(s, j);
                for (string t : dp[j]) dp[i].push_back(word + " " + t);
            }
        }
    }
};
```


### 300. Longest Increasing Subsequence

Given an unsorted array of integers, find the length of longest increasing subsequence.

Example:

Input: [10,9,2,5,3,7,101,18]
Output: 4 
Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4. 

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/longest-increasing-subsequence

最简单的解法是动态规划，复杂度是O(n^2)。

```
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int ans = 0;
        vector<int> lens(nums.size(), 1);
        for (int i = 0; i < nums.size(); ++i) {
            for (int j = 0; j < i; ++j) if (nums[j] < nums[i]) lens[i] = max(lens[i], lens[j] + 1);
            ans = max(ans, lens[i]);
        }
        return ans;
    }
};
```

进一步优化，用动态规划+二分查找的方法。用`lens[j]`表示长度为`j`的子序列中最小的结尾元素，注意到`lens[j]`是单调递增的，反证法可得。从头开始遍历每一个元素，对于第`i`个元素，找到第一个比它小的`lens[j]`即可（二分），则说明以`nums[i]`结尾的子序列最大长度是`j+1`，同时相应地更新`lens[j+1]`的值。复杂度是O(n logn)。

```
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int max_len = 0;
        vector<int> lens(nums.size() + 1, INT_MAX);
        for (int i = 0; i < nums.size(); ++i) {
            int left = 0, right = max_len;
            while (left < right) {
                int mid = (left + right + 1) / 2;
                if (lens[mid] < nums[i]) left = mid;
                else right = mid - 1;
            }
            lens[left + 1] = min(lens[left + 1], nums[i]);
            max_len = max(max_len, left + 1);
        }
        return max_len;
    }
};
```


### 264. 丑数 II

编写一个程序，找出第 n 个丑数。

丑数就是质因数只包含 2, 3, 5 的正整数。

遍历每一个数，并判断它是否是丑数的方法会超时。
其实只需要考虑2、3、5的组合即可，从小到大的排列由2、3、5生成的数，关键是如何排序。这里用动态规划的方法，用一个数组保存丑数数列，该数列的下一个数由已知丑数分别✖️2、✖️3、✖️5决定。

```
class Solution {
public:
    int nthUglyNumber(int n) {
        vector<int> dp(n, 0);
        int a = 0, b = 0, c = 0;
        dp[0] = 1;
        for (int i = 1; i < n; ++i) {
            dp[i] = min(dp[a] * 2, min(dp[b] * 3, dp[c] * 5));
            if (dp[a] * 2 == dp[i]) a++;
            if (dp[b] * 3 == dp[i]) b++;
            if (dp[c] * 5 == dp[i]) c++;
        }
        return dp[n-1];
    }
};
```