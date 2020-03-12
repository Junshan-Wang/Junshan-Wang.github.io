---
title: "Trie"
collection: solutions
permalink: /solutions/trie/
author_profile: true
---

### NOTE

* 字典树（前缀树），是一种k叉树，其中k是字符集的大小，每个节点表示一个字符，一条从根节点到中间节点的路径表示一个前缀，一条从根节点到叶子节点的路径表示一个字符。
* 字典树的更新和查询的复杂度都为`O(l)`，其中l为字符串的长度。
* 字典树可以用一个二维数组保存，`trie[i][j]`表示的是第i个节点对应的第j个字符指向的节点的索引。
* 字典树的插入和查询可以直接从上往下进行分裂。

### 425. Word Squares

Given a set of words (without duplicates), find all word squares you can build from them.

A sequence of words forms a valid word square if the kth row and column read the exact same string, where 0 ≤ k < max(numRows, numColumns).

For example, the word sequence ["ball","area","lead","lady"] forms a word square because each word reads the same both horizontally and vertically.

Note:

There are at least 1 and at most 1000 words.  
All words will have the exact same length.  
Word length is at least 1 and at most 5.  
Each word contains only lowercase English alphabet a-z.  

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/word-squares

采用递归的方法，每次考虑一个字符串，如果该字符串的前缀和已有字符串是满足转置条件的，则加入该字符串往下遍历。  
问题是如果每次都要遍历所有字符串，复杂度为O(nl)。  
注意到只需要字符串的前缀满足条件即可，所以采用trie进行保存，每次只取具有特定前缀的字符串即可，总的复杂度为O(l)。

```
class Solution {
public:
    vector<vector<string>> ans;
    vector<string> words;
    vector<vector<int>> trie;
    vector<vector<string>> strs;

    void insert(string s) {
        int p = 0;
        for (int i = 0; i < s.size(); ++i) {  
            int c = s[i] - 'a';
            if (trie[p][c] == 0) {
                trie[p][c] = trie.size();
                trie.push_back(vector<int>(26, 0));
                strs.push_back(vector<string>());
            }
            strs[p].push_back(s);
            p = trie[p][c];
        }
    } 

    int search(string prefix) {
        int p = 0;
        for (int i = 0; i < prefix.size(); ++i) {
            int c = prefix[i] - 'a';
            if (trie[p][c] == 0) return -1;
            p = trie[p][c];
        }
        return p;
    }

    vector<vector<string>> wordSquares(vector<string>& words) {
        this -> words = words;
        this -> trie.push_back(vector<int>(26, 0));
        this -> strs.push_back(vector<string>(0));
        for (string word : words) insert(word);
        vector<string> seq;
        recurse(seq);
        return ans;
    }

    void recurse(vector<string>& seq) {
        int n = seq.size();
        if (n == words[0].size()) {
            ans.push_back(vector<string>(seq));
            return;
        }
        string prefix = "";
        for (string s : seq) prefix += s[n]; 
        int p = search(prefix);
        if (p == -1) return;
        for (string word : strs[p]) {
            seq.push_back(word);
            recurse(seq);
            seq.pop_back();
        }
    }

};
```


### 642. Design Search Autocomplete System

Design a search autocomplete system for a search engine. Users may input a sentence (at least one word and end with a special character '#'). For each character they type except '#', you need to return the top 3 historical hot sentences that have prefix the same as the part of sentence already typed. Here are the specific rules:

The hot degree for a sentence is defined as the number of times a user typed the exactly same sentence before.  
The returned top 3 hot sentences should be sorted by hot degree (The first is the hottest one). If several sentences have the same degree of hot, you need to use ASCII-code order (smaller one appears first).  
If less than 3 hot sentences exist, then just return as many as you can.  
When the input is a special character, it means the sentence ends, and in this case, you need to return an empty list.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/design-search-autocomplete-system

因为要返回包含前缀的字符串，所以比较直观的可以采用trie，可以在O(l)的复杂度内动态更新，并查找到所有满足条件的字符串。

```
class AutocompleteSystem {
public:
    unordered_map<string, int> str2time;
    vector<vector<int>> trie;
    vector<unordered_set<string>> pres;
    string prefix;

    void insert(string s, int t) {
        str2time[s] += t;
        int p = 0;
        for (int i = 0; i < s.size(); ++i) {
            int c = (s[i] == ' ' ? 0 : s[i] - 'a' + 1);
            if (trie[p][c] == 0) {
                trie[p][c] = trie.size();
                trie.push_back(vector<int>(27, 0));
                pres.push_back(unordered_set<string>(0));
            }
            p = trie[p][c];
            pres[p].insert(s);
        }
    }

    int search(string s) {
        int p = 0;
        for (int i = 0; i < s.size(); ++i) {
            int c = (s[i] == ' ' ? 0 : s[i] - 'a' + 1);
            if (trie[p][c] == 0) return -1;
            p = trie[p][c];
        }
        return p;
    }


    AutocompleteSystem(vector<string>& sentences, vector<int>& times) {
        trie.push_back(vector<int>(27, 0));
        pres.push_back(unordered_set<string>(0));
        for (int i = 0; i < sentences.size(); ++i) insert(sentences[i], times[i]);
    }
    
    vector<string> input(char c) {
        vector<string> ans(0);
        if (c == '#') {
            insert(prefix, 1);
            prefix = "";
            return ans;
        }
        else {
            prefix += c;
            int p = search(prefix);
            if (p == -1) return ans;
            vector<pair<int, string>> t_s;
            for (string s : pres[p]) t_s.push_back(make_pair(1000 - str2time[s], s));
            sort(t_s.begin(), t_s.end());
            for (int k = 0; k < 3 && k < t_s.size(); ++k) ans.push_back(t_s[k].second);
            return ans;
        }
    }
};

/**
 * Your AutocompleteSystem object will be instantiated and called as such:
 * AutocompleteSystem* obj = new AutocompleteSystem(sentences, times);
 * vector<string> param_1 = obj->input(c);
 */
 ```