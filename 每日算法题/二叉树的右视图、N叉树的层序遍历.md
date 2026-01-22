题目一：【二叉树的右视图】https://leetcode.cn/problems/binary-tree-right-side-view/description/        队列、二叉树        "腾讯-2023-开发  字节跳动-2024-开发"        

```CPP
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        if(!root) return {};
        queue<TreeNode*> q;
        vector<int> ret;
        q.push(root);
        while(!q.empty()) {
            int levelSize = q.size();
            while(levelSize--) {
                TreeNode* top = q.front();  q.pop();
                if(levelSize == 0) ret.push_back(top->val);
                if(top->left) q.push(top->left);
                if(top->right) q.push(top->right);
            }
        }
        return ret;
    }
};
```

题目二：【N叉树的层序遍历】https://leetcode.cn/problems/n-ary-tree-level-order-traversal/description/        队列、BFS        腾讯-2022-开发   

```CPP
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        if(!root) return {};
        vector<vector<int>> ret;
        queue<Node*> q; q.push(root);
        while(q.size()) {
            int levelSize = q.size();
            vector<int> tmp;
            while(levelSize--) {
                Node* top = q.front(); q.pop();
                tmp.push_back(top->val);
                if(top->children.size())    // 遍历孩子
                    for(auto& c : top->children) q.push(c);
            }
            ret.push_back(tmp);
        }
        return ret;
    }
};
```

