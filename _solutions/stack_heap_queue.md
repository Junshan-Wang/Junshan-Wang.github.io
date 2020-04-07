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

### 5359. Maximum Performance of a Team
There are n engineers numbered from 1 to n and two arrays: speed and efficiency, where speed[i] and efficiency[i] represent the speed and efficiency for the i-th engineer respectively. Return the maximum performance of a team composed of at most k engineers, since the answer can be a huge number, return this modulo 10^9 + 7.

The performance of a team is the sum of their engineers' speeds multiplied by the minimum efficiency among their engineers. 



来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/maximum-performance-of-a-team


和上题类似的做法，贪心+优先队列。


```
class Solution {
public:
    int maxPerformance(int n, vector<int>& speed, vector<int>& efficiency, int k) {
        vector<pair<int, int>> eff_idx;
        priority_queue<int, vector<int>, greater<int>> spd;
        long long sum = 0, res = 0;
        
        for (int i = 0; i < n; ++i) eff_idx.push_back(make_pair(efficiency[i], i));
        sort(eff_idx.begin(), eff_idx.end(), greater<pair<int, int>>());
        
        for (int i = 0; i < n; ++i) {
            int eff = eff_idx[i].first, idx = eff_idx[i].second;
            
            if (spd.size() < k) {
                sum += speed[idx];
                spd.push(speed[idx]);
            }
            else if (spd.top() < speed[idx]) {
                sum -= spd.top();
                sum += speed[idx];
                spd.pop();
                spd.push(speed[idx]);
            }
            
            res = max(res, sum * eff);
        }
        return res % ((int)pow(10, 9) + 7);
    }
};
```

### 218. The Skyline Problem

A city's skyline is the outer contour of the silhouette formed by all the buildings in that city when viewed from a distance. Now suppose you are given the locations and height of all the buildings as shown on a cityscape photo (Figure A), write a program to output the skyline formed by these buildings collectively (Figure B).

The geometric information of each building is represented by a triplet of integers [Li, Ri, Hi], where Li and Ri are the x coordinates of the left and right edge of the ith building, respectively, and Hi is its height. It is guaranteed that 0 ≤ Li, Ri ≤ INT_MAX, 0 < Hi ≤ INT_MAX, and Ri - Li > 0. You may assume all buildings are perfect rectangles grounded on an absolutely flat surface at height 0.

For instance, the dimensions of all buildings in Figure A are recorded as: [ [2 9 10], [3 7 15], [5 12 12], [15 20 10], [19 24 8] ] .

The output is a list of "key points" (red dots in Figure B) in the format of [ [x1,y1], [x2, y2], [x3, y3], ... ] that uniquely defines a skyline. A key point is the left endpoint of a horizontal line segment. Note that the last key point, where the rightmost building ends, is merely used to mark the termination of the skyline, and always has zero height. Also, the ground in between any two adjacent buildings should be considered part of the skyline contour.

For instance, the skyline in Figure B should be represented as:[ [2 10], [3 15], [7 12], [12 0], [15 10], [20 8], [24, 0] ].

Notes:

The number of buildings in any input list is guaranteed to be in the range [0, 10000].  
The input list is already sorted in ascending order by the left x position Li.  
The output list must be sorted by the x position.  
There must be no consecutive horizontal lines of equal height in the output skyline. For instance, [...[2 3], [4 5], [7 5], [11 5], [12 7]...] is not acceptable; the three lines of height 5 should be merged into one in the final output as such: [...[2 3], [4 5], [12 7], ...]

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/the-skyline-problem

采用扫描线的方法，把每个建筑物的起始线和结束线按顺序排列好（注意x轴相同但y轴不同时的顺序）。依次扫描，如果当前线的高度大于已有线的最大高度（由于最大高度会一直更新，所以用最大堆维护，然而由于这里不仅要更新堆顶，而有可能删除其中任意元素，所以实际上用一个multiset来维护），则会产生关键点，此外还要更新最大高度的multiset。

```
class Solution {
public:
    vector<vector<int>> getSkyline(vector<vector<int>>& buildings) {
        vector<pair<int, int>> lines;
        multiset<int> heights;
        vector<vector<int>> res;
        for (int i = 1; i <= buildings.size(); ++i) {
            lines.push_back(make_pair(buildings[i - 1][0], - buildings[i - 1][2]));
            lines.push_back(make_pair(buildings[i - 1][1], buildings[i - 1][2]));
        }
        sort(lines.begin(), lines.end());
        heights.insert(0);

        for (int i = 0; i < lines.size(); ++i) {
            int x = lines[i].first, h = lines[i].second;
            if (h < 0) {
                h = - h;
                if (h > *heights.rbegin()) 
                    res.push_back(vector<int>{x, h});
                heights.insert(h);
            }
            else {
                heights.erase(heights.find(h));
                if (h > *heights.rbegin()) 
                    res.push_back(vector<int>{x, *heights.rbegin()});
            }
        }
        return res;
    }
};
```

### 23. Merge k Sorted Lists

Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.

最直观的想法是对每一个list维护一个当前指针，然后每次选择当前元素最小的一个list。但是这样的复杂度是O(kN)的，即每次都要遍历每个列表的指针选取最小值。维护一个最小堆，将复杂度降为O(logkN)。

```
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
        vector<ListNode*> pointers;
        ListNode *head = new ListNode(0);
        ListNode *p = head;
        for (int i = 0; i < lists.size(); ++i) {
            if (lists[i] != NULL) {
                pq.push(make_pair(lists[i] -> val, pq.size()));
                pointers.push_back(lists[i]);
            }
        }
        while (!pq.empty()) {
            int i = pq.top().second;
            pq.pop();
            p -> next = pointers[i];
            p = p -> next;
            pointers[i] = pointers[i] -> next;
            if (pointers[i] != NULL) pq.push(make_pair(pointers[i] -> val, i));
        }
        return head -> next;
    }
};
```



---
## 栈

栈，先进后出。



## 单调栈和单调队列

* 单调栈：数组中针对每一个元素从它右边寻找第一个比它大的元素，总的复杂度是O(n)
* 单调队列：数组中找到每一个窗口内的最大值，总的复杂度是O(n)

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



### 84. Largest Rectangle in Histogram

Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.

这道题最暴力的方法是对柱状图中的每一个方块，计算包含它的最大面积是多少，只需要分别找到它左边和右边高度比它小的方第一个方块。但是这种方法的时间复杂度是`O(n^2)`。

用动态规划的方法可以降低时间复杂度（不确定是否能到`O(n)`）。当前遍历到第$i$个方块时，我们向左寻找第一个比它小的第一个方块，不需要逐个遍历，而是用数组`left_idx[j]`保存比$j$小的第一个方块，用到已经计算好的`left_idx[j]`来加速更新`left_idx[i]`。

```
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        vector<int> left_idx(heights.size(), 0);
        vector<int> right_idx(heights.size(), 0);
        int max_area = 0;
        for (int i = 0; i < heights.size(); ++i) {
            int j = i - 1;
            while (j >= 0 && heights[j] >= heights[i])
                j = left_idx[j];
            left_idx[i] = j;
        }
        for (int i = heights.size() - 1; i >= 0; --i) {
            int j = i + 1;
            while (j <= heights.size() - 1 && heights[j] >= heights[i])
                j = right_idx[j];
            right_idx[i] = j;
        }
        for (int i = 0; i < heights.size(); ++i) 
            if (heights[i] * (right_idx[i] - left_idx[i] - 1) > max_area)
                max_area = heights[i] * (right_idx[i] - left_idx[i] - 1);
        return max_area;
    }
    
};
```

我第一次做的时候用分治的思路，即找到当前高度最小的方块$i$，则包含它的矩阵面积为`(right - left) * heights[i]`，然后分别在(left,i-1)和(i+1,right)两个区域内找最大矩阵，然后比较三者大小即可。这样的时间复杂度为`O(nlogn)`。

```
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        return search(heights, 0, heights.size());
    }
    
    int search(vector<int>& heights, int left, int right) {
        if (left == right)
            return 0;
        int smallest_idx, smallest_height = ~(1 << 31);
        for (int i = left; i < right; ++i)
            if (heights[i] <= smallest_height) {
                smallest_idx = i;
                smallest_height= heights[i];
            }
        int largest_area = (right - left) * smallest_height;
        int left_area = search(heights, left, smallest_idx);
        int right_area = search(heights, smallest_idx + 1, right);
        if (left_area > largest_area)
            largest_area = left_area;
        if (right_area > largest_area)
            largest_area = right_area;
        return largest_area;
    }
};
```

还有一种用单调栈的思路，用一个栈保存每个方块的索引，且从栈底到栈顶的方块高度是升序的。从左到右遍历每一个方块，如果当前方块的高度大于栈顶方块的高度时，则压栈；否则，表示以栈顶方块的高度形成的矩阵不能与当前方块组成矩阵，则计算以栈顶方块的高度形成的矩阵的面积大小，然后弹出栈顶，继续比较下一个栈顶和当前方块的高度大小。

### 85. Maximal Rectangle

Given a 2D binary matrix filled with 0's and 1's, find the largest rectangle containing only 1's and return its area.

上一题在二维场景下的扩展，只需要依次遍历每一行，以每一行作为底边计算最大矩形面积，高度可以用动态规划的思想来得到，而计算最大矩形面积可以用单调栈进行预处理，得到每一列左边和右边第一个高度比它小的位置。

```
class Solution {
public:
    int maximalRectangle(vector<vector<char>>& matrix) {
        int m = matrix.size();
        if (m == 0) return 0;
        int n = matrix[0].size();
        int max_sum = 0;
        vector<int> heights(n, 0);
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (matrix[i][j] == '0') heights[j] = 0;
                else heights[j] += 1;
            }
            stack<int> st;
            vector<int> left(n, 0), right(n, 0);
            for (int j = 0; j < n; ++j) {
                while (!st.empty() && heights[st.top()] >= heights[j]) st.pop();
                if (st.empty()) left[j] = -1;
                else left[j] = st.top(); 
                st.push(j);
            }
            stack<int> st2;
            for (int j = n - 1; j >= 0; --j) {
                while (!st2.empty() && heights[st2.top()] >= heights[j]) st2.pop();
                if (st2.empty()) right[j] = n;
                else right[j] = st2.top(); 
                st2.push(j);
            }
            for (int j = 0; j < n; ++j) {
                max_sum = max(max_sum, heights[j] * (right[j] - left[j] - 1));
            }
        }
        return max_sum;
    }
};
```