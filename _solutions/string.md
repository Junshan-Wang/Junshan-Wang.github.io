---
title: "String"
collection: solutions
permalink: /solutions/string/
author_profile: true
---

### 271. Encode and Decode Strings

Design an algorithm to encode a list of strings to a string. The encoded string is then sent over the network and is decoded back to the original list of strings.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/encode-and-decode-strings

这一题的关键其实是，用什么分隔字符串。由于出现的字符串有可能是任何一个ascii码字符，因此不能简单的用空格等特殊字符进行分隔。

* 可以用ascii码以外的特殊字符进行分隔，但是这样感觉意义不大，实际应用中有可能出现任何字符。
* 用多个特殊字符的组合进行分隔，其实这是参考了转义的方式。当然如果要完全严谨还是很复杂的，但是这道题目中用这种方法可以通过。
* 一个技巧：用表示字符串长度的数字作为字符进行分隔，但是考虑到不同长度占的位数不同，我们可以在每个字符串前留出一个固定长度的字符数，表示数字，比如留出四个字符，每个char占8bits，一共32bits，即一个整数类型。

*注意类型转换，如果占满了char的所有位数，则此时转换为int会变成负数，因为char认为是有符号的，会进行符号扩展。直接将char转换成unsigned int也会变成负数，因为char->int->unsigned int，所以还是进行符号扩展。需要将char转换为unsigned char。*  
*注意`to_string`函数和数组到字符的强制转换是不一样的。*

```
class Codec {
public:
    string int2string(int len) {
        string ans(4, 0);
        for (int i = 3; i >= 0; --i) {
            ans[i] = len % 256;
            len = len / 256;
        }
        return ans;
    }
    int string2int(string str) {
        int len = 0;
        for (int i = 0; i < 4; ++i) {
            len = len * 256 + (unsigned char)str[i];
        }
        return len;
    }

    // Encodes a list of strings to a single string.
    string encode(vector<string>& strs) {
        string ans;
        for (string str : strs) {
            string len = int2string(str.size());
            ans += len + str;
        }
        return ans;
    }

    // Decodes a single string to a list of strings.
    vector<string> decode(string s) {
        vector<string> ans;
        int i = 0;
        while (i < s.size()) {
            int len = string2int(s.substr(i, 4));
            ans.push_back(s.substr(i + 4, len));
            i = i + 4 + len;
        }
        return ans;
    }
};
```

### 158. Read N Characters Given Read4 II - Call multiple times

Given a file and assume that you can only read the file using a given method read4, implement a method read to read n characters. Your method read may be called multiple times.

Method read4:

The API read4 reads 4 consecutive characters from the file, then writes those characters into the buffer array buf.

The return value is the number of actual characters read.

Note that read4() has its own file pointer, much like FILE *fp in C.

来源：力扣（LeetCode） 
链接：https://leetcode-cn.com/problems/read-n-characters-given-read4-ii-call-multiple-times

考虑到当读取的字符串的大小n不一定能被4整除，因此需要额外维护一个临时内存tmp。保存三个成员变量：一个大小为4的char数组作为tmp，一个整数表示tmp的实际大小，另一个整数表示当前读取到tmp的位置。每次先将字符从文件读取到tmp中，更新两个状态，然后再从tmp中读取到实际的buf中。

```
// Forward declaration of the read4 API.
int read4(char *buf);

class Solution {
public:
    char tmp[4];
    int tmp_p = 4;
    int tmp_s = 4;
    /**
     * @param buf Destination buffer
     * @param n   Number of characters to read
     * @return    The number of actual characters read
     */
    int read(char *buf, int n) {
        int cnt = 0;
        while (cnt < n) {
            if (tmp_p == tmp_s) {
                tmp_s = read4(tmp);
                tmp_p = 0;
                if (tmp_s == 0) break;
            }
            while (tmp_p < tmp_s && cnt < n) {
                buf[cnt++] = tmp[tmp_p++];
            } 
        }
        return cnt;
    }
};
```

### 616. Add Bold Tag in String

Given a string s and a list of strings dict, you need to add a closed pair of bold tag <b> and </b> to wrap the substrings in s that exist in dict. If two such substrings overlap, you need to wrap them together by only one pair of closed bold tag. Also, if two substrings wrapped by bold tags are consecutive, you need to combine them.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/add-bold-tag-in-string

这道题挺简单的，首先需要每一个字典中的字符串都需要在目标字符串中进行查找，无法更优化，时间复杂度可以认为达到O(n^3)。然后问题是如何加入tag，只需要引入一个mask辅助数组，将每次匹配成功的对应的所有字符都设置为1，当字典中的所有字符串都匹配完后，只需要在对应的mask为01交替的位置插入tag即可。

```
class Solution {
public:
    string addBoldTag(string s, vector<string>& dict) {
        int p = -1;
        string ans;
        vector<int> mask(s.size(), 0);
        for (string w : dict) {
            while ((p = s.find(w, p + 1)) != string::npos) {
                for (int i = p; i < p + w.size(); ++i) 
                    mask[i] = 1;
            }
        }
        for (int i = 0; i < s.size(); ++i) {
            if (mask[i] == 1 && (i == 0 || mask[i - 1] == 0))
                ans += "<b>";
            ans += s[i];
            if (mask[i] == 1 && (i == s.size() - 1 || mask[i + 1] == 0))
                ans += "</b>";
        }
        return ans;
    }
};
```

### 1088. Confusing Number II

We can rotate digits by 180 degrees to form new digits. When 0, 1, 6, 8, 9 are rotated 180 degrees, they become 0, 1, 9, 8, 6 respectively. When 2, 3, 4, 5 and 7 are rotated 180 degrees, they become invalid.

A confusing number is a number that when rotated 180 degrees becomes a different number with each digit valid.(Note that the rotated number can be greater than the original number.)

Given a positive integer N, return the number of confusing numbers between 1 and N inclusive.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/confusing-number-ii

由于该题中N可能很大，所以无法每个数都遍历，采取另一种策略。合法的数字只有0，1，6，8，9，所以直接递归，每一位逐渐增加，直到当前数大于N时停止递归。同时对于访问到的每一个数，判断其旋转后和当前是否相同（只需要一个位数不同即不相同，枚举即可），且注意到一个数只会被访问到一次，所以如果当前数合法，则总数直接加1即可。


```
class Solution {
public:
    int N;
    int cnt;
    int confusingNumberII(int N) {
        this -> N = N;
        this -> cnt = 0;
        count(0);
        return cnt;
    }

    void count(long long i) {
        if (i > N) return;
        if (diff(i)) {cnt += 1;}
        if (i > 0) count(i * 10 + 0);
        count(i * 10 + 1);
        count(i * 10 + 6);
        count(i * 10 + 8);
        count(i * 10 + 9);
    }

    bool diff(int i) {
        string s = to_string(i);
        int n = s.size();
        for (int j = 0; j < n / 2; ++j) {
            if (s[j] == s[n - 1 - j] && s[j] != '6' && s[j] != '9') continue;
            else if (s[j] == '6' && s[n - 1 - j] == '9') continue;
            else if (s[j] == '9' && s[n - 1 - j] == '6') continue;
            else return true;
        }
        if (n % 2 == 1 && (s[n / 2] == '6' || s[n / 2] == '9')) return true;
        else return false;
    }
};
```