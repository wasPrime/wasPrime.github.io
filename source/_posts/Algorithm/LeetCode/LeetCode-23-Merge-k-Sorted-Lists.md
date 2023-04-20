---
title: LeetCode 23 - Merge k Sorted Lists
date: 2023-04-14 22:03:20
categories:
- [algorithm, leetcode, linked_list]
- [algorithm, leetcode, priority_queue]
- [algorithm, leetcode, heap]
tags:
- algorithm
- leetcode
- linked_list
- priority_queue
- heap
- hard
---

{% note info %}
Difficulty: {% label danger @hard %}
{% endnote %}

## Problem Description

### English (Merge k Sorted Lists)

You are given an array of `k` linked-lists `lists`, each linked-list is sorted in ascending order.

*Merge all the linked-lists into one sorted linked-list and return it.*

**Example 1:**

```log
Input: lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]
Explanation: The linked-lists are:
[
  1->4->5,
  1->3->4,
  2->6
]
merging them into one sorted list:
1->1->2->3->4->4->5->6
```

**Example 2:**

```log
Input: lists = []
Output: []
```

**Example 3:**

```log
Input: lists = [[]]
Output: []
```

**Constraints:**

- `k == lists.length`
- `0 <= k <= 104`
- `0 <= lists[i].length <= 500`
- `-104 <= lists[i][j] <= 104`
- `lists[i]` is sorted in **ascending order**.
- The sum of `lists[i].length` will not exceed $10^4$.

### Chinese (合并 K 个升序链表)

给你一个链表数组，每个链表都已经按升序排列。

请你将所有链表合并到一个升序链表中，返回合并后的链表。

**示例 1：**

```log
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：链表数组如下：
[
  1->4->5,
  1->3->4,
  2->6
]
将它们合并到一个有序链表中得到。
1->1->2->3->4->4->5->6
```

**示例 2：**

```log
输入：lists = []
输出：[]
```

**示例 3：**

```log
输入：lists = [[]]
输出：[]
```

**提示：**

- `k == lists.length`
- `0 <= k <= 10^4`
- `0 <= lists[i].length <= 500`
- `-10^4 <= lists[i][j] <= 10^4`
- `lists[i]` 按 **升序** 排列
- `lists[i].length` 的总和不超过 $10^4$

## Solution

### C++

```C++
/**
 * Definition for singly-linked list.
 */
struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<ListNode*, vector<ListNode*>, cmp> pq;
        for (ListNode* list : lists) {
            if (list != nullptr) {
                pq.push(list);
            }
        }

        ListNode* dummy = new ListNode;
        ListNode* last = dummy;
        while (!pq.empty()) {
            ListNode* cur = pq.top();
            pq.pop();
            last->next = cur;
            last = cur;
            if (cur->next != nullptr) {
                pq.push(cur->next);
            }
        }

        ListNode* res = dummy->next;
        delete dummy;
        return res;
    }

private:
    struct cmp {
        bool operator()(ListNode* a, ListNode* b) { return a->val < b->val; }
    };
};
```

### Python

{% note warning %}
In LeetCode's code template, we are unable to change the declaration of `ListNode` which means we can't add a `__lt__` method inside  `ListNode`. Fortunately, there is a workaround with `ListNode.__lt__ = ...` dynamically.
{% endnote %}

```Python
# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        import queue

        ListNode.__lt__ = lambda self, other: self.val < other.val
        pq = queue.PriorityQueue()
        for list in lists:
            if list is not None:
                pq.put(list)

        dummy = ListNode()
        last = dummy
        while not pq.empty():
            cur = pq.get()
            last.next = cur
            last = cur

            if cur.next is not None:
                pq.put(cur.next)

        return dummy.next
```
