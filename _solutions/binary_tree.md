---
title: "Binary Tree"
collection: solutions
permalink: /solutions/binary_tree/
author_profile: true
---

## Binary Search Tree

二叉搜索树的性质：
* 每个节点上的值，比它左子树上的任意值都大，比它右子树上的任意值都小
* 对搜索树进行中序遍历，得到递增序列

### 1382. Balance a Binary Search Tree

Given a binary search tree, return a balanced binary search tree with the same node values.

A binary search tree is balanced if and only if the depth of the two subtrees of every node never differ by more than 1.

If there is more than one answer, return any of them.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/balance-a-binary-search-tree

根据二叉搜索树的性质，进行中序遍历，得到有序递增序列。然后根据要求构造二叉平衡树即可，即在每个节点处进行分裂时，左右子树的节点数各占一半。

```
class Solution {
public:
    vector<int> nodes;
    TreeNode* balanceBST(TreeNode* root) {
        decode(root);
        TreeNode* new_root = encode(0, nodes.size() - 1);
        return new_root;
    }
    
    void decode(TreeNode* root) {
        if (root == NULL) return;
        decode(root -> left);
        nodes.push_back(root -> val);
        decode(root -> right);
    }
    
    TreeNode* encode(int i, int j) {
        if (i > j) return NULL;
        int mid = (i + j) / 2;
        TreeNode* root = new TreeNode(nodes[mid]);
        root -> left = encode(i, mid - 1);
        root -> right = encode(mid + 1, j);
        return root;
    }
};
```

---

## DFS

树的遍历，前序、中序、后序遍历。

### 549. Binary Tree Longest Consecutive Sequence II

Given a binary tree, you need to find the length of Longest Consecutive Path in Binary Tree.

Especially, this path can be either increasing or decreasing. For example, [1,2,3,4] and [4,3,2,1] are both considered valid, but the path [1,2,4,3] is not valid. On the other hand, the path can be in the child-Parent-child order, where not necessarily be parent-child order.

来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/binary-tree-longest-consecutive-sequence-ii

考虑到有可能两种顺序，并且有可能从子节点到父节点，因此情况较为复杂。

采用后序遍历的方法，即先访问每个节点的两个子节点，关键是需要返回两个值：增序的长度和降序的长度。然后考虑两种情况可以更新最大长度：没有拐点和以当前节点作为拐点。

```
class Solution {
public:
    int max_len = 0;
    int longestConsecutive(TreeNode* root) {
        search(root);
        return max_len;
    }

    vector<int> search(TreeNode* root) {
        if (root == NULL) return vector<int>(2, 0);
        vector<int> left_len = search(root -> left);
        vector<int> right_len = search(root -> right);

        vector<int> len(2, 1);
        if (root -> left != NULL) {
            if (root -> left -> val == root -> val + 1) len[0] = max(len[0], left_len[0] + 1);
            else if (root -> left -> val + 1 == root -> val) len[1] = max(len[1], left_len[1] + 1);
        }
        if (root -> right != NULL) {
            if (root -> right -> val == root -> val + 1) len[0] = max(len[0], right_len[0] + 1);
            else if (root -> right -> val + 1 == root -> val) len[1] = max(len[1], right_len[1] + 1);
        }
        max_len = max(max_len, len[0]);
        max_len = max(max_len, len[1]);
        
        if (root -> left != NULL && root -> right != NULL) {
            if (root -> left -> val == root -> val + 1 && root -> right -> val + 1 == root -> val) {
                max_len = max(max_len, left_len[0] + right_len[1] + 1);
            }
            else if (root -> left -> val + 1 == root -> val && root -> right -> val == root -> val + 1) {
                max_len = max(max_len, left_len[1] + right_len[0] + 1);
            }
        }
        return len;
    }
};
```