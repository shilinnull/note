题目一：【验证回文串II】https://leetcode.cn/problems/RQku0D/description/        字节跳动-2024-开发              

```CPP
class Solution {
public:
    bool validPalindrome(string s) {
        int left = 0, right = s.size() - 1;
        while (left < right) {
            if (s[left] != s[right]) {
                // 尝试删除左边字符 或 删除右边字符
                return isPalindrome(s, left + 1, right) || 
                       isPalindrome(s, left, right - 1);
            }
            left++;
            right--;
        }
        return true;
    }
    bool isPalindrome(string& s, int left, int right) {
        while(left < right) {
            if(s[left] != s[right]) return false;
            left++, right--; 
        }
        return true;
    }
};
```



题目二：【x的平方根】        https://leetcode.cn/problems/jJ0w9p/description/        百度-2024-开发

```CPP
class Solution {
public:
    int mySqrt(int x) {
        if(x <= 1) return x;
        int left = 1, right = x;
        while(left < right) {
            long long mid = left + (right - left + 1) / 2;
            if(mid * mid <= x) left = mid;
            else right = mid - 1;
        }
        return left;
    }
};
```

