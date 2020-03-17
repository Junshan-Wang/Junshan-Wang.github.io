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