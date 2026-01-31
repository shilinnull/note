题目一：【K个一组翻转链表】https://leetcode.cn/problems/reverse-nodes-in-k-group/description/

```CPP
class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        int n = 0;
        ListNode* cur = head;
        while(cur) {
            n++;
            cur = cur->next;
        }
        cur = head;
        n /= k;
        ListNode* newHead = new ListNode(-1);
        ListNode* newTail = newHead;
        while(n--) // n组
        {
            ListNode* tmp = cur;    // 先记录每组的第一个
            for(int i = 0; i < k; i++) {    // 反转k个
                ListNode* next = cur->next;
                cur->next = newTail->next;
                newTail->next = cur;
                cur = next;
            }
            newTail = tmp;  // 将上一次插入的最后一个作为下一个的头
        }
        newTail->next = cur;    // 最后连接剩下的
        return newHead->next;
    }
};
```



题目二：【字符串相乘】https://leetcode.cn/problems/multiply-strings/description/?envType=problem-list-v2&envId=string 

```CPP
class Solution {
public:
    string multiply(string num1, string num2) {
        reverse(num1.begin(), num1.end());
        reverse(num2.begin(), num2.end());
        int n = num1.size();
        int m = num2.size();
        // 无进位相乘
        vector<int> tmp(m + n - 1);
        for(int i = 0; i < n; i++)
            for(int j = 0; j < m; j++)
                tmp[i + j] += (num1[i] - '0') * (num2[j] - '0');

        // 处理进位
        string ret;
        int next = 0;
        for(int i = 0; i < tmp.size() || next; i++) {
            if(i < tmp.size()) next += tmp[i];
            ret += next % 10 + '0';
            next /= 10;
        }
        while(ret.size() > 1 && ret.back() == '0') ret.pop_back();
        reverse(ret.begin(), ret.end());
        return ret;
    }
};
```

