---
title: "Depth First Search (DFS) / Breadth First Search"
collection: solutions
permalink: /solutions/dfs_bfs/
author_profile: true
---

### 753. Cracking the Safe
There is a box protected by a password. The password is a sequence of n digits where each digit can be one of the first k digits 0, 1, ..., k-1.

While entering a password, the last n digits entered will automatically be matched against the correct password.

For example, assuming the correct password is "345", if you type "012345", the box will open because the correct password matches the suffix of the entered password.

Return any password of minimum length that is guaranteed to open the box at some point of entering it.


这道题目还是比较难的，首先要考虑到将问题抽象为图的遍历，构造对应的图结构，然后用寻找欧拉回路的算法（Hierholzer算法）。

构造图，每个节点表示n-1位密码，有$k^{n-1}$个节点，每个节点有k条出边和入边，每条边表示一位密码，即一个节点和一条出边正好表示n位密码，图中的所有边表示所有可能的n位密码，即有$k^{n-1}*k$条边。

欧拉回路：一条能够不重不漏地经过图上的每一条边的路径，且路径的起点和终点相同。Hierholzer算法：深度遍历，注意到是采用后序遍历的方法，从而保证序列是连续的。

不过我的解法有点复杂，实际上可以通过维护一个`set`来记录哪些边被访问过。

```
class Solution {
public:
    unordered_map<string, unordered_set<string>> adjs;
    string ans;
    string crackSafe(int n, int k) {
        if (n == 1) {
            for (int j = 0; j < k; ++j)
                ans += to_string(j);
            return ans;
        }
        else {
            vector<string> nodes = {""};
            gen_adjs(0, n, k, nodes);
            dfs(nodes[0]);
            return nodes[0].substr(0, nodes[0].size() - 1) + ans;
        }
    }

    void gen_adjs(int cur, int n, int k, vector<string>& nodes) {
        int size = nodes.size();
        for (int i = 0; i < size; ++i) {
            string node = nodes[i];
            nodes[i] = node + "0";
            for (int j = 1; j < k; ++j) {
                nodes.push_back(node + to_string(j));
            }
        }
        if (cur < n - 2)
            gen_adjs(cur + 1, n, k, nodes);
        else {
            for (string node : nodes) {
                adjs[node] = unordered_set<string>();
                string child = node.substr(1);
                for (int j = 0; j < k; ++j) {
                    adjs[node].insert(child + to_string(j));
                }
            }
        }
    }

    void dfs(string node) {
        for (unordered_set<string>::iterator it = adjs[node].begin(); it != adjs[node].end(); it = adjs[node].begin()) {
            string child = *it;
            adjs[node].erase(it);
            dfs(child);
        }
        ans = node.substr(node.size() - 1) + ans;;
    }

};
```

---

### 489. Robot Room Cleaner

Given a robot cleaner in a room modeled as a grid.

Each cell in the grid can be empty or blocked.

The robot cleaner with 4 given APIs can move forward, turn left or turn right. Each turn it made is 90 degrees.

When it tries to move into a blocked cell, its bumper sensor detects the obstacle and it stays on the current cell.

Design an algorithm to clean the entire room using only the 4 given APIs shown below.

```
interface Robot {
  // returns true if next cell is open and robot moves into the cell.
  // returns false if next cell is obstacle and robot stays on the current cell.
  boolean move();

  // Robot will stay on the same cell after calling turnLeft/turnRight.
  // Each turn will be 90 degrees.
  void turnLeft();
  void turnRight();

  // Clean the current cell.
  void clean();
}
```

这道题其实挺简单的，问题在输入是完全不知道，只能通过move的返回结果来判断，而它封装好的函数不知道具体实现是什么，导致一开始疯狂报错。就比如`robot`这个类不仅是包含了当前机器人的位置，还包含了地面的清洁情况，导致复制一个新`robot`结果会有问题。

其实关键就是维护一个全局数组/字典`vis`保存每一个区域的访问情况。

```
class Solution {
public:
    unordered_map<int, int> vis;
    vector<pair<int, int>> dir = {pair(-1, 0), pair(0, 1), pair(1, 0), pair(0, -1)};
    
    void cleanRoom(Robot& robot) {
        search(robot, 0, 0, 0);
    }

    void search(Robot& robot, int x, int y, int d) {
        robot.clean();
        for (int i = 0; i < 4; ++i) {
            int d_ = (d + i) % 4;
            int x_ = x + dir[d_].first;
            int y_ = y + dir[d_].second;
            if (vis.find(x_ * 20000 + y_) != vis.end());
            else if (robot.move()) {
                vis[x_ * 20000 + y_] = 1;
                search(robot, x_, y_, d_);
                reset(robot);
            }
            robot.turnRight();
        }
    }
    void reset(Robot& robot) {
        robot.turnLeft();
        robot.turnLeft();
        robot.move();
        robot.turnLeft();
        robot.turnLeft();
    }
};
```

---
### 947. Most Stones Removed with Same Row or Column

On a 2D plane, we place stones at some integer coordinate points.  Each coordinate point may have at most one stone.

Now, a move consists of removing a stone that shares a column or row with another stone on the grid.

What is the largest possible number of moves we can make?

这道题的关键是，需要观察到，对于一个连通分量（位于同一行或者同一列的石头认为有边），每移走一块石头相当于删除一个节点及其对应的边，最终连通分量只剩下一个孤立点，最多的删除次数即为点的个数。（显然，但是没有比较严谨的解释）

则寻找连通分量有两个办法，深搜（`O(n^2)`）或者并查集（`O(nlogn)`），并查集复杂度更低。

DFS：
```
class Solution {
public:
    unordered_map<int, int> row;
    unordered_map<int, int> col;
    unordered_map<int, vector<int>> adj;
    unordered_map<int, int> vis;
    int removeStones(vector<vector<int>>& stones) {
        int cnt = 0;

        for (int i = 0; i < stones.size(); ++i) {
            vis[i] = 0;
            if (row.find(stones[i][0]) == row.end()) row[stones[i][0]] = i;
            else {
                adj[i].push_back(row[stones[i][0]]);
                adj[row[stones[i][0]]].push_back(i);
            }
            if (col.find(stones[i][1]) == col.end()) col[stones[i][1]] = i;
            else {
                adj[i].push_back(col[stones[i][1]]);
                adj[col[stones[i][1]]].push_back(i);
            }
        }
        for (unordered_map<int, int>::iterator it = vis.begin(); it != vis.end(); it++) {
            if (it -> second == 0) {
                dfs(it -> first);
                cnt++;
            }
        }
        return stones.size() - cnt;
    }

    void dfs(int cur) {
        vis[cur] = 1;
        for (int i : adj[cur]) {
            if (!vis[i]) dfs(i);
        }
    }
};
```

并查集：
```
class Solution {
public:
    unordered_map<int, int> row;
    unordered_map<int, int> col;
    unordered_map<int, int> par;
    unordered_set<int> vis;
    int removeStones(vector<vector<int>>& stones) {
        int cnt = 0;

        for (int i = 0; i < stones.size(); ++i) {
            par[i] = i;
            if (row.find(stones[i][0]) == row.end()) row[stones[i][0]] = i;
            else merge(i, row[stones[i][0]]);
            if (col.find(stones[i][1]) == col.end()) col[stones[i][1]] = i;
            else merge(i, col[stones[i][1]]);
        }
        for (unordered_map<int, int>::iterator it = par.begin(); it != par.end(); it++) {
            if (vis.find(find(it -> first)) == vis.end()) {
                cnt++;
                vis.insert(find(it -> first));
                // cout << it -> first << " " << it -> second << endl;
            }
        }
        return stones.size() - cnt;
    }

    int find(int x) {
        if (par[x] != x) par[x] = find(par[x]);
        return par[x];
    }

    void merge(int x, int y) {
        int px = find(x);
        int py = find(y);
        par[px] = py;
    }
};
```

### 417. Pacific Atlantic Water Flow

Given an m x n matrix of non-negative integers representing the height of each unit cell in a continent, the "Pacific ocean" touches the left and top edges of the matrix and the "Atlantic ocean" touches the right and bottom edges.

Water can only flow in four directions (up, down, left, or right) from a cell to another one with height equal or lower.

Find the list of grid coordinates where water can flow to both the Pacific and Atlantic ocean.

Note:

The order of returned grid coordinates does not matter.
Both m and n are less than 150.


来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/pacific-atlantic-water-flow  

这题无法用动归求解，因为每个格子的可达性可能受到其上下左右四个格子的影响。

用dfs实现。  
一开始考虑从每个格子出发，搜索它是否能到达海洋，但是这样似乎需要保存一个vis数组和一个avail数组，比较麻烦。  
考虑从四条海岸线出发，找到所有可达的格子，只需要一个数组进行标记即可。

```
class Solution {
public:
    vector<vector<int>> pacificAtlantic(vector<vector<int>>& matrix) {
        int m = matrix.size();
        if (m == 0) return vector<vector<int>>();
        int n = matrix[0].size();
        vector<vector<bool>> pac(m, vector<bool>(n, false)), atl(m, vector<bool>(n, false));
        vector<vector<int>> res(0);
        for (int i = 0; i < m; ++i) dfs(matrix, pac, i, 0);
        for (int j = 0; j < n; ++j) dfs(matrix, pac, 0, j);
        for (int i = 0; i < m; ++i) dfs(matrix, atl, i, n - 1);
        for (int j = 0; j < n; ++j) dfs(matrix, atl, m - 1, j);
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (pac[i][j] && atl[i][j]) res.push_back(vector<int>{i, j});
            }
        }
        return res;
    }

    void dfs(vector<vector<int>>& matrix, vector<vector<bool>>& mem, int i, int j) {
        mem[i][j] = 1;
        if (i > 0 && matrix[i][j] <= matrix[i - 1][j] && !mem[i - 1][j]) dfs(matrix, mem, i - 1, j);
        if (j > 0 && matrix[i][j] <= matrix[i][j - 1] && !mem[i][j - 1]) dfs(matrix, mem, i, j - 1);
        if (i < matrix.size() - 1 && matrix[i][j] <= matrix[i + 1][j] && !mem[i + 1][j]) dfs(matrix, mem, i + 1, j);
        if (j < matrix[0].size() - 1 && matrix[i][j] <= matrix[i][j + 1] && !mem[i][j + 1]) dfs(matrix, mem, i, j + 1);
    }
};
```