---
title: "Graph"
collection: solutions
permalink: /solutions/linked_list/
author_profile: true
---

### 138. Copy List with Random Pointer

A linked list is given such that each node contains an additional random pointer which could point to any node in the list or null.

Return a deep copy of the list.

The Linked List is represented in the input/output as a list of n nodes. Each node is represented as a pair of [val, random_index] where:

val: an integer representing Node.val
random_index: the index of the node (range from 0 to n-1) where random pointer points to, or null if it does not point to any node.


来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/copy-list-with-random-pointer  

可以利用hash方法，设计旧节点指针到新节点指针的映射，实现O(n)的时间复杂度。  
可以进一步通过在旧链表中间隔插入新链表，完成next指针和random指针的复制，实现O(1)的额外空间复杂度。（略）

```
class Solution {
public:
    Node* copyRandomList(Node* head) {
        Node *p;
        unordered_map<Node*, Node*> old2new;
        p = head;
        while (p != NULL) {
            old2new[p] = new Node(p -> val);
            p = p -> next;
        }
        p = head;
        while (p != NULL) {
            old2new[p] -> next = old2new[p -> next];
            old2new[p] -> random = old2new[p -> random];
            p = p -> next; 
        }
        if (head == NULL) return NULL;
        else return old2new[head];
    }
};
```