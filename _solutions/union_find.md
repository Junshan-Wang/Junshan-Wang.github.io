---
title: "Union Find"
collection: solutions
permalink: /solutions/union_find/
author_profile: true
---

并查集，主要是两个函数`unoin`和`find`，并且可以分别在两个函数中加入优化：重量权衡分配和路径压缩。

### 305. Number of Islands II

A 2d grid map of m rows and n columns is initially filled with water. We may perform an addLand operation which turns the water at position (row, col) into a land. Given a list of positions to operate, count the number of islands after each addLand operation. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/number-of-islands-ii

如果只用普通的搜索方法，复杂度为O(mnL)，用并查集后复杂度为O(L)。

```
class Solution {
public:
    unordered_map<int, int> par;
    unordered_map<int, int> weight;
    unordered_set<int> root;
    vector<int> numIslands2(int m, int n, vector<vector<int>>& positions) { 
        vector<int> res;
        for (int k = 0; k < positions.size(); ++k) {
            int p = positions[k][0] * n + positions[k][1];
            if (par.find(p) == par.end()) {
                par[p] = p;
                weight[p] = 1;
                root.insert(p);
                if (p / n > 0 && par.find(p - n) != par.end()) merge(p, p - n);
                if (p / n < m - 1 && par.find(p + n) != par.end()) merge(p, p + n);
                if (p % n > 0 && par.find(p - 1) != par.end()) merge(p, p - 1);
                if (p % n < n - 1 && par.find(p + 1) != par.end()) merge(p, p + 1);
            }
            res.push_back(root.size());
        }
        return res;
    }

    void merge(int x, int y) {
        int px = find(x);
        int py = find(y);
        if (px != py) {
            if (weight[px] > weight[py]) {
                par[py] = px;
                weight[py] += weight[px];
                root.erase(py);
            }
            else {
                par[px] = py;
                weight[px] += weight[py];
                root.erase(px);
            }
        }
    }

    int find(int x) {
        if (par[x] != x) {
            par[x] = find(par[x]);
        }
        return par[x];
    }
};
```


### 399. Evaluate Division

Equations are given in the format A / B = k, where A and B are variables represented as strings, and k is a real number (floating point number). Given some queries, return the answers. If the answer does not exist, return -1.0.

Example:  
Given a / b = 2.0, b / c = 3.0.  
queries are: a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ? .  
return [6.0, 0.5, -1.0, 1.0, -1.0 ].

The input is: vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries , where equations.size() == values.size(), and the values are positive. This represents the equations. Return vector<double>.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/evaluate-division

这一道题的关键实际上是建图，即根据已知等式构建图，然后求图上两点的”距离“。注意到不连通的两点说明无法得到答案。可以用DFS / 并查集 / 最短路径的方法求解。这里用了Floyd算法。

```
class Solution {
public:
    vector<double> calcEquation(vector<vector<string>>& equations, vector<double>& values, vector<vector<string>>& queries) {
        unordered_map<string, int> string2idx;
        for (int i = 0; i < equations.size(); ++i) {
            string x = equations[i][0], y = equations[i][1];
            if (string2idx.find(x) == string2idx.end()) string2idx[x] = string2idx.size();
            if (string2idx.find(y) == string2idx.end()) string2idx[y] = string2idx.size();
        }
        
        int n = string2idx.size();
        vector<vector<double>> adj(n, vector<double>(n, -1));
        vector<vector<double>> dis(n, vector<double>(n, -1));
        for (int i = 0; i < n; ++i) {
            adj[i][i] = 1;
            dis[i][i] = 1;
        }
        for (int i = 0; i < equations.size(); ++i) {
            string x = equations[i][0], y = equations[i][1];
            adj[string2idx[x]][string2idx[y]] = values[i];
            adj[string2idx[y]][string2idx[x]] = 1.0 / values[i];
        }
        for (int k = 0; k < n; ++k) {
            for (int i = 0; i < n; ++i) {
                for (int j = 0; j < n; ++j) {
                    if (dis[i][j] == -1 && dis[i][k] != -1 && adj[k][j] != -1) {
                        dis[i][j] = dis[i][k] * adj[k][j];
                        dis[j][i] = 1.0 / dis[i][j];
                    }
                }
            }
        }

        vector<double> res;
        for (int i = 0; i < queries.size(); ++i) {
            string x = queries[i][0], y = queries[i][1];
            if (string2idx.find(x) == string2idx.end() || string2idx.find(y) == string2idx.end()) {
                res.push_back(-1.0);
            }
            else {
                res.push_back(dis[string2idx[x]][string2idx[y]]);
            }
        }
        return res;
    }
};
```