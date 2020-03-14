---
title: "Stack / Heap / Queue"
collection: solutions
permalink: /solutions/stack_heap_queue/
author_profile: true
---

## Queue

队列，先进先出。

### 353. Design Snake Game
Design a Snake game that is played on a device with screen size = width x height. Play the game online if you are not familiar with the game.

The snake is initially positioned at the top left corner (0,0) with length = 1 unit.

You are given a list of food's positions in row-column order. When a snake eats the food, its length and the game's score both increase by 1.

Each food appears one by one on the screen. For example, the second food will not appear until the first food was eaten by the snake.

When a food does appear on the screen, it is guaranteed that it will not appear on a block occupied by the snake.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/design-snake-game

这题的关键是如何保存当前蛇的状态。首先，贪吃蛇的状态可以很好的用`queue`来保存，蛇的头部和尾部分别对应队列的头部和尾部，根据蛇是否吃到食物来决定是否需要弹出队列尾。其次，为了判断当前位置是否合法，需要一个数据结构保存当前的非法位置，我采用二维数组的方式保存屏幕上每个格子的状态。


```
class SnakeGame {
public:
    /** Initialize your data structure here.
        @param width - screen width
        @param height - screen height 
        @param food - A list of food positions
        E.g food = [[1,1], [1,0]] means the first food is positioned at [1,1], the second is at [1,0]. */
    int width;
    int height;
    vector<vector<int>> food;
    queue<vector<int>> body;
    vector<vector<int>> screen;
    int score;
    SnakeGame(int width, int height, vector<vector<int>>& food) {
        this -> width = width;
        this -> height = height;
        this -> food = food;
        this -> body.push(vector<int>{0,0});
        this -> screen = vector<vector<int>>(height, vector<int>(width, 0));
        this -> screen[0][0] = 1;
        this -> score = 0;
    }
    
    /** Moves the snake.
        @param direction - 'U' = Up, 'L' = Left, 'R' = Right, 'D' = Down 
        @return The game's score after the move. Return -1 if game over. 
        Game over when snake crosses the screen boundary or bites its body. */
    int move(string direction) {
        vector<int> pos = body.back();
        if (direction == "U") pos[0]--;
        else if (direction == "D") pos[0]++;
        else if (direction == "L") pos[1]--;
        else pos[1]++;

        if (pos[0] < 0 || pos[0] >= height || pos[1] < 0 || pos[1] >= width) return -1;
        else if (score < food.size() && pos[0] == food[score][0] && pos[1] == food[score][1]) score++;
        else {screen[body.front()[0]][body.front()[1]] = 0; body.pop();}

        if (screen[pos[0]][pos[1]] == 1) return -1;
        body.push(pos);
        screen[pos[0]][pos[1]] = 1;
        return score;
    }
};
```

### 362. Design Hit Counter

Design a hit counter which counts the number of hits received in the past 5 minutes.

Each function accepts a timestamp parameter (in seconds granularity) and you may assume that calls are being made to the system in chronological order (ie, the timestamp is monotonically increasing). You may assume that the earliest timestamp starts at 1.

It is possible that several hits arrive roughly at the same time.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/design-hit-counter

和时间有关的题目，满足先进先出条件。

```
class HitCounter {
public:
    queue<int> qu;
    /** Initialize your data structure here. */
    HitCounter() {

    }
    
    /** Record a hit.
        @param timestamp - The current timestamp (in seconds granularity). */
    void hit(int timestamp) {
        qu.push(timestamp);
        while (!qu.empty() && timestamp - qu.front() >= 300) qu.pop();
    }
    
    /** Return the number of hits in the past 5 minutes.
        @param timestamp - The current timestamp (in seconds granularity). */
    int getHits(int timestamp) {
        while (!qu.empty() && timestamp - qu.front() >= 300) qu.pop();
        return qu.size();
    }
};

/**
 * Your HitCounter object will be instantiated and called as such:
 * HitCounter* obj = new HitCounter();
 * obj->hit(timestamp);
 * int param_2 = obj->getHits(timestamp);
 */
 ```


---
## Priority Queue / Heap

优先队列 / 最大堆：排序结构，堆顶最大元素。

### 857. Minimum Cost to Hire K Workers

There are N workers.  The i-th worker has a quality[i] and a minimum wage expectation wage[i].

Now we want to hire exactly K workers to form a paid group.  When hiring a group of K workers, we must pay them according to the following rules:

Every worker in the paid group should be paid in the ratio of their quality compared to other workers in the paid group.
Every worker in the paid group must be paid at least their minimum wage expectation.
Return the least amount of money needed to form a paid group satisfying the above conditions.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/minimum-cost-to-hire-k-workers

这题浪费了很多时间。。

一开始考虑用动态规划的方法，至少需要O(NK) = O(N^2)的复杂度。

可以考虑这种思路，我们知道，由于要保持每个worker的工资和质量的比例一样，同时每个worker的工资有下界，因此每一种满足条件的方法，都有一个worker的工资和质量比例作为下界。简单的遍历每一种合法方案，即以每一个worker的比例作为下界，找到另外K-1个比例大于等于该下界且quality最小的worker，复杂度是O(N * N * log N)。这种方法复杂度过高，因为每次都需要重新排序，是否可以不重新排序呢。  
* 由于我们只是需要知道最小的K-1个worker，可以维护一个最大堆，堆的大小为K-1，保存当前最小的K-1个quality。
* 为了保证堆内的worker都是满足条件的，我们首先对worker按照工资和质量的比例进行升序排序。注意到，对于任意i，如果i满足条件，则对于任意j < i，j也满足条件。所以可以对于排序后的worker，顺序的进行遍历，将遍历到的worker的quality插入堆中（如果比堆顶元素小）。
* 计算当前堆内所有worker和当前worker的quality总和，乘上工资和质量的比例。
* 时间复杂度为O(N log N)。

```
class Solution {
public:
    double mincostToHireWorkers(vector<int>& quality, vector<int>& wage, int K) {
        int N = quality.size(), quality_sum = 0;
        double min_res = -1;
        vector<pair<double, int>> wq_idx;
        priority_queue<int> pq;

        for (int i = 0; i < N; ++i) {
            wq_idx.push_back(make_pair((double)wage[i] / (double)quality[i], i));
        }
        sort(wq_idx.begin(), wq_idx.end());

        for (int i = 0; i < N; ++i) {
            pq.push(quality[wq_idx[i].second]);
            quality_sum += quality[wq_idx[i].second];
            if (pq.size() == K) {
                if (min_res == -1 || min_res > (double)quality_sum * wq_idx[i].first) {
                    min_res = (double)quality_sum * wq_idx[i].first;
                }
                quality_sum -= pq.top();
                pq.pop();
            }
        }

        return min_res;
    }
};
```

---
## 栈

栈，先进后出。

### 975. Odd Even Jump

You are given an integer array A.  From some starting index, you can make a series of jumps.  The (1st, 3rd, 5th, ...) jumps in the series are called odd numbered jumps, and the (2nd, 4th, 6th, ...) jumps in the series are called even numbered jumps.

You may from index i jump forward to index j (with i < j) in the following way:

* During odd numbered jumps (ie. jumps 1, 3, 5, ...), you jump to the index j such that A[i] <= A[j] and A[j] is the smallest possible value.  If there are multiple such indexes j, you can only jump to the smallest such index j.
* During even numbered jumps (ie. jumps 2, 4, 6, ...), you jump to the index j such that A[i] >= A[j] and A[j] is the largest possible value.  If there are multiple such indexes j, you can only jump to the smallest such index j.
* (It may be the case that for some index i, there are no legal jumps.)
A starting index is good if, starting from that index, you can reach the end of the array (index A.length - 1) by jumping some number of times (possibly 0 or more than once.)

Return the number of good starting indexes.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/odd-even-jump

先考虑最简单的思路，需要从每个起点出发进行跳转，而每次的跳转需要计算跳转目标，复杂度为O(n)，需要跳转O(n)次，需要从O(n)个起点出发，总的复杂度为O(n^3)。

然后考虑优化，即先计算到达每个点在奇数和偶数次时的跳转目标，直接进行预处理的复杂度为O(n^2)，总的复杂度也为O(n^2)。是否能进一步优化呢。

* 预处理的优化：计算每个点的跳转目标，也就是找到比第i个数大的最小的元素。首先对问题进行转换，对数组进行排序，得到排序后的索引，则问题转换为找到比第i个数大的第一个元素。此时可以用*单调栈*的方法，单调栈可以用O(n)的复杂度，找到每个元素右边第一个比它大的元素，单调栈从栈顶到栈底递减。从左到右遍历索引数组，如果当前元素大于栈顶元素，则可将栈顶元素弹出，同时记录下栈顶元素对应的右边第一个比它大的数是当前元素。
* 计算可达性的优化：原始的方法实际是一种递归，注意到计算第i个位置的可达性只和j > i有关，和j <= i无关。所以动态规划从后往前计算，递推方程为：dp_odd[i] = (odd_jump[i] != -1) & dp_even[odd_jump[i]]，dp_even[i] = (even_jump[i] != -1) & dp_odd[even_jump[i]]，且初始化为dp_odd[n - 1] = true, dp_even[n - 1] = true，其他为false。同时统计dp_odd为true的次数即可。

```
class Solution {
public:
    int oddEvenJumps(vector<int>& A) {
        vector<pair<int, int>> val_idx;
        for (int i = 0; i < A.size(); ++i) val_idx.push_back(make_pair(A[i], i));
        sort(val_idx.begin(), val_idx.end(), compare_odd);
        vector<int> odd_jump = jump(val_idx);
        sort(val_idx.begin(), val_idx.end(), compare_even);
        vector<int> even_jump = jump(val_idx);
        return dp(odd_jump, even_jump);
    }

    static bool compare_odd(pair<int, int> x, pair<int, int> y) {
        if (x.first == y.first) return x.second < y.second;
        return x.first < y.first;
    }
    static bool compare_even(pair<int, int> x, pair<int, int> y) {
        if (x.first == y.first) return x.second < y.second;
        return x.first > y.first;
    }

    vector<int> jump(vector<pair<int, int>>& val_idx) {
        vector<int> x;
        stack<int> st;
        vector<int> y(val_idx.size(), -1);
        for (int i = 0; i < val_idx.size(); ++i) x.push_back(val_idx[i].second);
        for (int i = 0; i < x.size(); ++i) {
            while (!st.empty() && st.top() < x[i]) {
                y[st.top()] = x[i];
                st.pop();
            }
            st.push(x[i]);
        }
        return y;
    }

    int dp(vector<int>& odd_jump, vector<int>& even_jump) {
        vector<bool> odd_vis(odd_jump.size(), true);
        vector<bool> even_vis(even_jump.size(), true);
        int cnt = 1;
        for (int i = odd_jump.size() - 2; i >= 0; --i) {
            if (odd_jump[i] == -1) odd_vis[i] = false;
            else odd_vis[i] = even_vis[odd_jump[i]];
            if (even_jump[i] == -1) even_vis[i] = false;
            else even_vis[i] = odd_vis[even_jump[i]];
            if (odd_vis[i]) cnt++;
        }
        return cnt;
    }
};
```

## 单调栈和单调队列

* 单调栈：数组中针对每一个元素从它右边寻找第一个比它大的元素，总的复杂度是O(n)
* 单调队列：数组中找到每一个窗口内的最大值，总的复杂度是O(n)