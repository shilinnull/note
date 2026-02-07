根据二叉树创建字符串https://leetcode.cn/problems/construct-string-from-binary-tree/description/?envType=problem-list-v2&envId=binary-tree        百度-2023-测开

如果 root 左右子节点都不存在，则返回 root

如果 root 左右子节点都存在，则返回 root(left)(right)

如果 root 只有左节点存在，则返回 root(left)

如果 root 只有右节点存在，则返回 root()(right)

```CPP
class Solution {
public:
    string tree2str(TreeNode* root) {
        if(!root->left && !root->right) {
            return to_string(root->val);
        } else if(root->left && root->right) {
            return to_string(root->val) + "(" + tree2str(root->left) + ")" + "(" + tree2str(root->right) + ")";
        } else if(root->left && !root->right) {
            return to_string(root->val) + "(" + tree2str(root->left) + ")";
        } else {
            return to_string(root->val) + "()" + "(" + tree2str(root->right) + ")";
        }

    }
};
```



买股票的最佳时机II        https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/?envType=problem-list-v2&envId=greedy&difficulty=MEDIUM                b站-2021-测开

```CPP
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        int ret = 0;
        for(int i = 1; i < n; i++) {
            if(prices[i] > prices[i - 1]) { // 只要有一天比前一天大就卖
                ret += prices[i] - prices[i - 1];
            }
        }
        return ret;
    }
};
```

