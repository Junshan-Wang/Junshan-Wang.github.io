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

### 25. Reverse Nodes in k-Group

Given a linked list, reverse the nodes of a linked list k at a time and return its modified list.

k is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of k then left-out nodes in the end should remain as it is.

Example:

Given this linked list: 1->2->3->4->5

For k = 2, you should return: 2->1->4->3->5

For k = 3, you should return: 3->2->1->4->5

Note:

Only constant extra memory is allowed.  
You may not alter the values in the list's nodes, only nodes itself may be changed.


来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/reverse-nodes-in-k-group

其实想法比较简单，只需要搞清楚每一段的链表是如何反转的即可。

```
class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        int s = 0;
        ListNode *p = head;
        ListNode *new_head = new ListNode(0);
        new_head -> next = head;
        ListNode *begin = new_head, *end = new_head, *q;
        while (p != NULL) {
            p = p -> next;
            s++;
        }
        for (int i = 0; i < s / k; ++i) {
            begin = end;
            p = begin;
            q = begin -> next;
            for (int j = 0; j < k; ++j) {
                ListNode *tmp = q -> next;
                q -> next = p;
                p = q;
                q = tmp;
            }
            end = begin -> next;
            begin -> next -> next = q;
            begin -> next = p;
        }
        return new_head -> next;
    }
};
```