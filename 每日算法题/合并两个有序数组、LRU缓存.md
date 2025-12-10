题目一：【合并两个有序数组】[https://leetcode.cn/problems/merge-sorted-array/solutions/666608/he-bing-liang-ge-you-xu-shu-zu-by-leetco-rrb0/](https://leetcode.cn/problems/merge-sorted-array/solutions/666608/he-bing-liang-ge-you-xu-shu-zu-by-leetco-rrb0)

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int nums[m + n + 1], k = 0;
        int p1 = 0, p2 = 0;
        while(p1 < m && p2 < n) {
            if(nums1[p1] < nums2[p2])
                nums[k++] = nums1[p1++];
            else nums[k++] = nums2[p2++];
        }
        while(p1 < m) nums[k++] = nums1[p1++];
        while(p2 < n) nums[k++] = nums2[p2++];
        for(int i = 0; i < m + n; i++) nums1[i] = nums[i];
    }
};
```


题目二：【LRU缓存】[https://leetcode.cn/problems/lru-cache-lcci/description/](https://leetcode.cn/problems/lru-cache-lcci/description/)

```cpp
class LRUCache {
    int capacity_;
    list<pair<int, int>> LRU_list_;  // 头部是最近使用
    unordered_map<int, list<pair<int, int>>::iterator> key_list_map_;  // 哈希表：key -> 链表迭代器

public:
    LRUCache(int capacity) : capacity_(capacity) {}
    
    int get(int key) {
        // 1. 检查key是否存在
        auto it = key_list_map_.find(key);
        if (it == key_list_map_.end()) {
            return -1;
        }
        // 2. 存在则移到链表头部（先删后插）
        LRU_list_.splice(LRU_list_.begin(), LRU_list_, it->second);
        // 3. 返回对应的值
        return it->second->second;
    }
    
    void put(int key, int value) {
        // 情况1：key已存在，更新值并移到头部
        auto it = key_list_map_.find(key);
        if (it != key_list_map_.end()) {
            // 更新value
            it->second->second = value;
            // 移到链表头部
            LRU_list_.splice(LRU_list_.begin(), LRU_list_, it->second);
            return;
        }
        
        // 情况2：key不存在，检查容量
        if (LRU_list_.size() == capacity_) {
            // 容量满，删除尾部元素（最久未使用）
            auto last_pair = LRU_list_.back();
            int last_key = last_pair.first;
            // 哈希表中删除该key
            key_list_map_.erase(last_key);
            // 链表中删除尾部
            LRU_list_.pop_back();
        }
        
        // 插入新元素到链表头部
        LRU_list_.emplace_front(key, value);
        // 哈希表中记录迭代器
        key_list_map_[key] = LRU_list_.begin();
    }
};
```



