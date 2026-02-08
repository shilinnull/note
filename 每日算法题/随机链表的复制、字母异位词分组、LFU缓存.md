题目一：【随机链表的复制】https://leetcode.cn/problems/copy-list-with-random-pointer/?envType=problem-list-v2&envId=hash-table

使用在每个节点的后面插入一个节点，然后再进行链接

```CPP
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if(!head) return head;
        // 1. 创建新的节点链接到每个节点的后面
        Node* cur = head;
        while(cur) {
            Node* t = new Node(cur->val);
            t->next = cur->next;
            cur->next = t;
            cur = t->next;
        }
        // 2. 连接random指针
        cur = head;
        while (cur) {
            if (cur->random) {
                cur->next->random = cur->random->next;
            }
            cur = cur->next->next;
        }
        // 3. 断开链接
        cur = head;
        Node* ret = head->next;
        Node* copyCur = ret;
        while (copyCur) {
            // 恢复原链表的next指针
            cur->next = copyCur->next;
            cur = cur->next;
            
            // 设置新链表的next指针
            if (cur) {
                copyCur->next = cur->next;
                copyCur = copyCur->next;
            } else {
                copyCur->next = nullptr;
                copyCur = nullptr;
            }
        }
        return ret;
    }
};
```

使用hash表进行记录

```CPP
class Solution {
public:
    Node* copyRandomList(Node* head) {
        unordered_map<Node*, Node*> map;
        
        // 1.创建所有新节点，并建立原节点到新节点的映射
        Node* cur = head;
        while (cur) 
        {
            Node* node = new Node(cur->val);
            map[cur] = node;
            cur = cur->next;
        }
        
        // 2. 设置新节点的next和random指针
        cur = head;
        while (cur) 
        {
            // 设置新节点的next指针：指向原节点next对应的新节点
            map[cur]->next = map[cur->next];
            // 设置新节点的random指针：指向原节点random对应的新节点
            map[cur]->random = map[cur->random];
            cur = cur->next;
        }
        
        return map[head];
    }
};
```



题目二：【字母异位词分组】https://leetcode.cn/problems/group-anagrams/description/?envType=problem-list-v2&envId=hash-table

```CPP
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> hash;
        for(auto &s : strs) {
            string t = s;
            sort(t.begin(), t.end());
            hash[t].push_back(s);   // 将排序后相同的放到一组
        }

        vector<vector<string>> ret;
        for(auto &v : hash) {
            ret.push_back(v.second);
        }
        return ret;
    }
};
```



题目三：【LFU缓存】https://leetcode.cn/problems/lfu-cache/solutions/186348/lfuhuan-cun-by-leetcode-solution/?envType=problem-list-v2&envId=hash-table

````CPP
struct Node {
    int key, val, freq;
    Node* prev;
    Node* next;
    Node(int k, int v) : key(k), val(v), freq(1), prev(nullptr), next(nullptr) {}
};

class LFUCache {
private:
    int capacity;
    int minFreq;
    unordered_map<int, Node*> keyToNode;
    unordered_map<int, list<Node*>> freqToList;
    
    void updateFreq(Node* node) {
        // 从原频率链表中移除
        int freq = node->freq;
        freqToList[freq].remove(node);
        
        // 如果该频率链表为空且是最小频率，更新minFreq
        if (freq == minFreq && freqToList[freq].empty()) {
            minFreq++;
        }
        
        // 增加频率
        node->freq++;
        freq = node->freq;
        
        // 添加到新频率链表的头部
        freqToList[freq].push_front(node);
    }
    
public:
    LFUCache(int capacity) {
        this->capacity = capacity;
        this->minFreq = 0;
    }
    
    int get(int key) {
        if (keyToNode.find(key) == keyToNode.end()) {
            return -1;
        }
        
        Node* node = keyToNode[key];
        updateFreq(node);       // 更新频率
        return node->val;
    }
    
    void put(int key, int value) {
        if (capacity == 0) return;
        
        if (keyToNode.find(key) != keyToNode.end()) {
            // 键已存在，更新值和频率
            Node* node = keyToNode[key];
            node->val = value;
            updateFreq(node);
        } else {
            // 键不存在，需要插入
            if (keyToNode.size() == capacity) {
                // 容量已满，需要删除最不经常使用的节点
                Node* nodeToRemove = freqToList[minFreq].back();
                freqToList[minFreq].pop_back();
                keyToNode.erase(nodeToRemove->key);
                delete nodeToRemove;
            }
            
            // 创建新节点
            Node* newNode = new Node(key, value);
            keyToNode[key] = newNode;
            
            // 新节点的频率为1，添加到频率1的链表头部
            minFreq = 1;
            freqToList[1].push_front(newNode);
        }
    }
    
    ~LFUCache() {
        // 清理内存
        for (auto& pair : keyToNode) {
            delete pair.second;
        }
    }
};
````



