---
title: "Back tracking"
collection: solutions
permalink: /solutions/back_tracking/
author_profile: true
---

递归 / 回溯算法

## MinMax

博弈 / 游戏问题中的方法。

### 294. Flip Game II

You are playing the following Flip Game with your friend: Given a string that contains only these two characters: + and -, you and your friend take turns to flip two consecutive "++" into "--". The game ends when a person can no longer make a move and therefore the other person will be the winner.

Write a function to determine if the starting player can guarantee a win.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/flip-game-ii

这道题动态规划无法直接解。需要用到递归方法。当前的情况下，如果存在一种翻转方法，使得翻转后的先手无法获胜，则当前情况一定可以获胜；否则如果所有翻转方法，都使得翻转后的先手一定能获胜，则当前情况无法获胜。

进一步，可以用记忆方法进行加速，用一个哈序表存储某一种字符串下的结果。

```
class Solution {
public:
    unordered_map<string, bool> mem;
    bool canWin(string s) {
        if (mem.find(s) != mem.end()) return mem[s];
        for (int i = 0; i < (int)s.size() - 1; ++i) {
            if (s[i] == '+' && s[i + 1] == '+') {
                s[i] = '-';
                s[i + 1] = '-';
                if (!canWin(s)) {
                    s[i] = '+';
                    s[i + 1] = '+';
                    mem[s] = true;
                    return true;
                }
                s[i] = '+';
                s[i + 1] = '+';
            }
        }
        mem[s] = false;
        return false;
    }
};
```

---

### 679. 24 Game

You have 4 cards each containing a number from 1 to 9. You need to judge whether they could operated through *, /, +, -, (, ) to get the value of 24.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/24-game 

这道题其实就是枚举，因为一共也就几千种情况。关键是如何枚举，实际上只需要枚举所有可能的数字排列，有24种情况。然后对于每种数字排列，枚举不同的计算顺序。对于每两个数的计算，有4种符号。而枚举的过程都是通过递归来完成。

```
class Solution {
public:
    bool judgePoint24(vector<int>& nums) {
        vector<double> ns(0);
        for (int i = 0; i < nums.size(); ++i) ns.push_back(nums[i]);
        vector<vector<double>> pers = gen_pers(ns, 0);
        for (int i = 0; i < pers.size(); ++i) {
            vector<double> res = judge(pers[i]);
            for (double r : res) {
                if (abs(r - 24.0) < 0.000001) return true;
            }
        }
        return false;
    }

    vector<double> judge(vector<double>& nums) {
        vector<double> res(0);
        if (nums.size() == 1) return nums;

        for (int i = 1; i < nums.size(); ++i) {
            vector<double> per1(0), per2(0), res1(0), res2(0);
            for (int j = 0; j < i; ++j) per1.push_back(nums[j]);
            for (int j = i; j < nums.size(); ++j) per2.push_back(nums[j]);
            res1 = judge(per1);
            res2 = judge(per2);
            for (double a : res1) for (double b : res2) {
                res.push_back(a + b);
                res.push_back(a - b);
                res.push_back(a * b);
                res.push_back(a / b);
            }
        }
        return res;
    }

    vector<vector<double>> gen_pers(vector<double>& nums, int n) {
        vector<vector<double>> pers(0);
        if (n == nums.size() - 1) pers.push_back(nums);
        for (int i = n; i < nums.size(); ++i) {
            swap(nums[i], nums[n]);
            vector<vector<double>> tmp = gen_pers(nums, n + 1);
            for (auto t : tmp) pers.push_back(t);
            swap(nums[i], nums[n]);
        }
        return pers;
    }
};
```