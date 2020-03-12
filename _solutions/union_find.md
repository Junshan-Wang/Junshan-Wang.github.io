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