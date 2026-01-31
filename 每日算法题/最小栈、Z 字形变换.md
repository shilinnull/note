题目一：【最小栈】https://leetcode.cn/problems/min-stack/description/ "字节-2024-开发 Meta-2024-开发"       

思路：每次入栈2个元素，一个是入栈的元素本身，一个是当前栈元素的最小值

```CPP
class MinStack {
public:
    stack<int> s;
    int minNum = INT_MAX;
    MinStack() {
    }
    
    void push(int val) {
        if(s.empty()) {
            s.push(val);
            s.push(val);
        } else {
            int tmp = s.top();  s.push(val);
            if(tmp < val) s.push(tmp);
            else s.push(val);
        }
    }
    
    void pop() {
        s.pop();
        s.pop();
    }
    
    int top() {
        int tmp = s.top(); s.pop();
        int ret = s.top(); s.push(tmp);
        return ret;
    }
    
    int getMin() {
        return s.top();
    }
};
```



题目二：【Z 字形变换】https://leetcode.cn/problems/zigzag-conversion/description/ "字节-2024-开发 微软-2024-开发 亚马逊-2024-开发 中兴-2024-开发"

```CPP
class Solution {
public:
    string convert(string s, int numRows) {
        if (numRows < 2)
            return s;
        vector<string> t(numRows);
        int i = 0, flag = -1;
        for(auto &c : s) {
            t[i].push_back(c);
            if(i == 0 || i == numRows - 1)  // 折返
                flag = -flag;
            i += flag;
        }
        string ret;
        for(auto &s : t) {
            ret += s;
        }
        return ret;
    }
};
```

