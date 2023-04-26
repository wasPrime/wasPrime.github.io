---
title: Traverse Binary Tree
date: 2023-04-28 23:29:45
categories:
- [algorithm, design]
tags:
- algorithm
- design
- binary_tree
- dfs
- stack
---

Binary tree traversal is a typical problem to implement. I suppose it's a required course for every programmer to master. I mean, not only recursive traversals, but also non-recursive traversals in preorder, inorder and postorder.

The question number on LeetCode are:

- `LeetCode 144 - Binary Tree Preorder Traversal`
- `LeetCode 94 - Binary Tree Inorder Traversal`
- `LeetCode 145 - Binary Tree Postorder Traversal`

## Declaration of Binary Tree

### C++

```C++
/**
 * Definition for a binary tree node.
 */
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;

    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};
```

### Python

```Python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## Recursive

### Preorder Traversal in Recursive

#### C++

```C++
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        if (root == nullptr) {
            return {};
        }

        vector<int> left_result = preorderTraversal(root->left), right_result = preorderTraversal(root->right);

        vector<int> result;
        result.reserve(1 + left_result.size() + right_result.size());

        result.push_back(root->val);
        copy(left_result.begin(), left_result.end(), back_inserter(result));
        copy(right_result.begin(), right_result.end(), back_inserter(result));

        return result;
    }
};
```

#### Python

```Python
class Solution:
    def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        return [root.val] + self.preorderTraversal(root.left) + self.preorderTraversal(root.right) if root else []
```

### Inorder Traversal in Recursive

#### C++

```C++
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        if (root == nullptr) {
            return {};
        }

        vector<int> left_result = inorderTraversal(root->left), right_result = inorderTraversal(root->right);

        vector<int> result;
        result.reserve(left_result.size() + 1 + right_result.size());

        copy(left_result.begin(), left_result.end(), back_inserter(result));
        result.push_back(root->val);
        copy(right_result.begin(), right_result.end(), back_inserter(result));

        return result;
    }
};
```

#### Python

```Python
class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        return self.inorderTraversal(root.left) + [root.val] + self.inorderTraversal(root.right) if root else []
```

### Postorder Traversal in Recursive

#### C++

```C++
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        if (root == nullptr) {
            return {};
        }

        vector<int> left_result = postorderTraversal(root->left), right_result = postorderTraversal(root->right);

        vector<int> result;
        result.reserve(left_result.size() + right_result.size() + 1);

        copy(left_result.begin(), left_result.end(), back_inserter(result));
        copy(right_result.begin(), right_result.end(), back_inserter(result));
        result.push_back(root->val);

        return result;
    }
};
```

#### Python

```Python
class Solution:
    def postorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        return self.postorderTraversal(root.left) + self.postorderTraversal(root.right) + [root.val] if root else []
```

## Non-Recursive

### Preorder Traversal in Non-Recursive

#### C++

```C++
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        stack<TreeNode*> node_stack;
        vector<int> result;

        TreeNode* cur = root;
        while (cur != nullptr || !node_stack.empty()) {
            if (cur == nullptr) {
                cur = node_stack.top();
                node_stack.pop();
            }

            result.push_back(cur->val);
            if (cur->right != nullptr) {
                node_stack.push(cur->right);
            }

            cur = cur->left;
        }

        return result;
    }
};
```

### Inorder Traversal in Non-Recursive

#### C++

```C++
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        stack<TreeNode*> node_stack;
        vector<int> result;

        TreeNode* cur = root;
        while (cur != nullptr || !node_stack.empty()) {
            if (cur == nullptr) {
                cur = node_stack.top();
                node_stack.pop();

                result.push_back(cur->val);
                cur = cur->right;
            } else {
                node_stack.push(cur);
                cur = cur->left;
            }
        }

        return result;
    }
};
```

### Postorder Traversal in Non-Recursive

#### C++

```C++

class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        stack<TreeNode*> node_stack;
        vector<int> result;

        TreeNode *cur = root, *right_visited = nullptr;
        while (cur != nullptr || !node_stack.empty()) {
            if (cur == nullptr) {
                cur = node_stack.top();

                if (cur->right == nullptr || cur->right == right_visited) {
                    result.push_back(cur->val);
                    node_stack.pop();
                    right_visited = cur;
                    cur = nullptr;
                } else {
                    cur = cur->right;
                }
            } else {
                node_stack.push(cur);
                cur = cur->left;
            }
        }

        return result;
    }
};
```

### Summary

We can summarize the template for binary tree traversal in non-recursive as below:

```C++
vector<int> postorderTraversal(TreeNode* root) {
    stack<TreeNode*> node_stack;
    vector<int> result;

    TreeNode* cur = root;
    while (cur != nullptr || !node_stack.empty()) {
        if (cur == nullptr) {
            ...
        } else {
            ...
        }
    }

    return result;
}
```
