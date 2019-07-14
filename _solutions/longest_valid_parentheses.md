---
title: "32. Longest Valid Parentheses"
collection: solutions
permalink: /solutions/longest_valid_parentheses/
author_profile: true
---


Given a string containing just the characters '(' and ')', find the length of the longest valid (well-formed) parentheses substring.

```
Example 1:

Input: "(()"
Output: 2
Explanation: The longest valid parentheses substring is "()"
```

```
Example 2:

Input: ")()())"
Output: 4
Explanation: The longest valid parentheses substring is "()()"
```

---

### 解法 1

我的解法比较繁琐，大概思路就是用两个栈来做。首先想到的是判断括号是否匹配题目的思路，即其中一个栈用来保存还未匹配上的'('或者')'，如果当前遍历到的括号能够和栈顶的括号匹配上则将栈顶弹出；否则将当前括号压栈。但是这种做法不能解决这题，考虑到可以将字符串认为由很多个满足括号匹配的子串组成，而每个子串之间则是无法匹配的'('或者')'，于是用另一个栈保存匹配上的字符串的长度，如果当前遍历到的括号能匹配（根据第一个栈的情况）则将栈顶的0弹出，如果此时栈顶大于0，则继续弹出并将栈顶的数+2再压栈，否则直接将2压栈；否则将0压栈。最后将第二个栈的数依次弹出，找到最大的非0的数。

```
class Solution {
public:
    int longestValidParentheses(string s) {
        stack<char> pars;
        stack<int> lens;
        for (auto c : s) {
            if (c == '(') {
                pars.push(c);
                lens.push(0);
            }
            else {
                if (pars.empty() || pars.top() == ')') {
                    pars.push(c);
                    lens.push(0);
                }
                else {
                    pars.pop();
                    int l = 2;
                    while (lens.top() > 0) {
                        l += lens.top();
                        lens.pop();
                    }
                    lens.pop();
                    lens.push(l);
                }
            }
        }
        int l = 0, max_len = 0;
        while (!lens.empty()) {
            if (lens.top() > 0) {
                l += lens.top();
            }
            else if (lens.top() == 0 && l > max_len) {
                max_len = l;
                l = 0;
            }
            else {
                l = 0;
            }
            lens.pop();
        }
        if (l > max_len)
            max_len = l;
        return max_len;
    }
};
```

其实栈的思路可以直接用一个栈解决，该栈保存了上一个无法匹配的括号的位置，如何区分这里的无法匹配的括号是'('还是')'呢？可以发现，当一个无法匹配的右括号出现时，它前面所有的左括号都一定已经被匹配了，那么它以后也不会再被匹配，也就是说我们只需保存最后一个未被匹配的右括号的位置即可，且该位置会被保存在栈底。在某一时刻，如果栈的元素数量等于1，则说明只有未被匹配的右括号，而没有未被匹配的左括号；如果大于1，则说明有未被匹配的左括号，且位置会被优先弹出。我们只需要判断栈顶的是左括号还是右括号，然后计算相应的长度即可。

### 解法 2

这道题还可以用动态规划的方法来做，`dp[i]`保存以第i个字符结尾的最长匹配字符串，更新的公式为
* 如果`s[i]='('`，则`dp[i] = 0`
* 如果`s[i]=')'`且`s[i-1]='('`，则`dp[i]=dp[i-2]+2`
* 如果`s[i]=')'`且`s[i-1]=')'`且`s[i-dp[i-1]-1]=')'`，则`dp[i]=dp[i-1]+dp[i-dp[i-1]-2]+2`

### 解法 3

上述两种方法都实现了`O(n)`的时间复杂度，但是空间复杂度也都为`O(n)`，有一种方法能将空间复杂度降为`O(1)`。

从左到右遍历字符串，用`left`和`right`分别表示左括号和右括号的个数。当`left=right`时，当前位置以前的字符串能够完成匹配，且长度为`2*right`；当`left<right`时，会出现一个无法再被匹配的右括号，它也将字符串分成互不影响的前后两个子串，此时将`left`和`right`都置0。

但是这么做会忽略掉`left`一直大于`right`的情况，即有无法匹配的左括号，于是我们需要再从右到左遍历字符串，类似的重复上述做法，最终能得到最长的匹配字符串长度。