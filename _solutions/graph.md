---
title: "Graph"
collection: solutions
permalink: /solutions/graph/
author_profile: true
---

## Topogolical Sort

拓扑排序，满足一系列先后关系的节点进行排序。还可以查找图中的环。

### 269. Alien Dictionary

There is a new alien language which uses the latin alphabet. However, the order among letters are unknown to you. You receive a list of non-empty words from the dictionary, where words are sorted lexicographically by the rules of this new language. Derive the order of letters in this language.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/alien-dictionary

没想到数算习题课上的题居然还是有用的。根据单词出现的先后关系，构造图，然后进行拓扑排序。注意到不合法的情况就是图中有环，即还没结束就找不到度数为0的节点/结束时数量少于n。

```
class Solution {
public:
    string alienOrder(vector<string>& words) {
        unordered_map<char, vector<char>> adj;
        unordered_map<char, int> deg;
        if (words.size() == 1) return words[0]; 
        for (int i = 1; i < words.size(); ++i) {
            for (int j = 0; j < words[i - 1].size(); ++j) {
                char x = words[i - 1][j];
                if (adj.find(x) == adj.end()) {
                    adj[x] = vector<char>(0);
                    deg[x] = 0;
                }
            }
            for (int j = 0; j < words[i].size(); ++j) {
                char x = words[i][j];
                if (adj.find(x) == adj.end()) {
                    adj[x] = vector<char>(0);
                    deg[x] = 0;
                }
            }
            for (int j = 0; j < words[i - 1].size() && j < words[i].size(); ++j) {
                char x = words[i - 1][j], y = words[i][j];
                if (words[i - 1][j] != words[i][j]) {
                    adj[x].push_back(y);
                    deg[y]++;
                    break;
                }
            }
        }

        int n = adj.size();
        char p;
        string res = "";
        for (int i = 0; i < n; ++i) {
            p = '0';
            for (unordered_map<char, int>::iterator j = deg.begin(); j != deg.end(); ++j) {
                if (j -> second == 0) {
                    p = j -> first;
                    res += p;
                    deg.erase(j -> first);
                    break;
                }
            }
            if (p == '0') return "";
            for (char c : adj[p]) deg[c]--;
        }
        return res;
    }
};
```