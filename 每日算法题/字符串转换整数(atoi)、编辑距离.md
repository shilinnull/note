题目一：【字符串转换整数(atoi)】https://leetcode.cn/problems/string-to-integer-atoi/description/?envType=problem-list-v2&envId=string

```CPP
class Solution {
public:
    int myAtoi(string s) {
        int n = s.size();
        int ret = 0;
        int i = 0, flag = 0;
        
        // 1. 跳过前导空格
        while (i < n && s[i] == ' ') {
            i++;
        }
        
        if (i >= n) return 0;
        
        // 2. 处理符号
        int sign = 1;
        if (s[i] == '-') {
            sign = -1;
            i++;
        } else if (s[i] == '+') {
            i++;
        }

        // 3. 读取数字并处理溢出
        long long result = 0; // 使用 long long 来检测溢出
        
        while (i < n && s[i] >= '0' && s[i] <= '9') {
            result = result * 10 + (s[i] - '0');
            i++;
            
            // 检查溢出
            if (result * sign > INT_MAX) {
                return INT_MAX;
            }
            if (result * sign < INT_MIN) {
                return INT_MIN;
            }
        }
        
        return (int)(result * sign);
    }
};
```



题目二：【编辑距离】https://leetcode.cn/problems/edit-distance/description/?envType=problem-list-v2&envId=string

```CPP
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.size(), m = word2.size();
        vector<vector<int>> dp(n + 1, vector<int>(m + 1));
        // 处理其中一个字符串为空的情况
        for(int i = 0; i <= n; i++)
            dp[i][0] = i;
        for(int j = 0; j <= m; j++)
            dp[0][j] = j;

        for(int i = 1; i <= n; i++)
            for(int j = 1; j <= m; j++)
                if(word1[i - 1] == word2[j - 1])
                    dp[i][j] = dp[i - 1][j - 1];
                else 
                    //                  增              删              替换
                    dp[i][j] = 1 + min(dp[i - 1][j], min(dp[i][j - 1], dp[i - 1][j - 1]));
        return dp[n][m];
    }
};
```

