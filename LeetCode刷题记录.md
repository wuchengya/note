# LeetCode刷题记录

可惜之前的200多道题没记录上

## [88. 合并两个有序数组 - 力扣（LeetCode）](https://leetcode.cn/problems/merge-sorted-array/submissions/655379468/?envType=study-plan-v2&envId=top-interview-150)

MyCode:

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        vector<int> res(m+n);
        int a = 0,b = 0;
        while(a!=m || b!=n)
        {
            if(a==m){
                res[a+b] = nums2[b];
                b++;
                continue;
            }
            if(b==n){
                res[a+b] = nums1[a];
                a++;
                continue;
            }
            if(nums1[a]>=nums2[b])
            {
                res[a+b] = nums2[b];
                b++;
            }
            else{
                res[a+b] = nums1[a];
                a++;
            }
        }
        nums1 = res;
    }
};
```

这是个时间复杂度O(m+n)，空间为O(m+n)的算法，已经很不错了，如何进一步优化空间呢？

逆向双指针

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int p1 = m-1,p2 = n-1,p3 = m+n-1;
        while(p1>=0||p2>=0)
        {
            if(p1==-1)
            {
                nums1[p3] = nums2[p2];
                p3--,p2--;
                continue;
            }
            if(p2==-1)
            {
                nums1[p3] = nums1[p1];
                p3--,p1--;
                continue;
            }
            if(nums1[p1]>=nums2[p2])
            {
                nums1[p3] = nums1[p1];
                p3--,p1--;
            }
            else{
                nums1[p3] = nums2[p2];
                p2--,p3--;
            }
        }
    }
};
```

感觉心态有点乱了，一直在想实习的事情，该怎么找到实习呢？

## [27. 移除元素 - 力扣（LeetCode）](https://leetcode.cn/problems/remove-element/description/?envType=study-plan-v2&envId=top-interview-150)

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int cur = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] != val) {
                nums[cur++] = nums[i];
            }
        }
        return cur;
    }
};
```

大佬的快慢指针，我怎么没想到呢？

## [206. 反转链表 - 力扣（LeetCode）](https://leetcode.cn/problems/reverse-linked-list/submissions/655680771/?envType=study-plan-v2&envId=top-100-liked)

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head==nullptr) return nullptr;
        ListNode* last = nullptr;
        ListNode* next = head->next;
        while(next)
        {
            head->next = last;
            last = head;
            head = next;
            next = next->next;
        }
        head->next = last;
        return head;
    }
};
```

类似于三个指针划过去，每次让中间的指针->next = 前面一个。然后集体向后平移，但是因为后面的指针是探路者，所以说必须要控制它不能为空，但是这样就会导致最后一个无法连上，所以循环外要单独处理一下。

 

## [234. 回文链表 - 力扣（LeetCode）](https://leetcode.cn/problems/palindrome-linked-list/submissions/655682001/?envType=study-plan-v2&envId=top-100-liked)

```cpp
class Solution {
public:

        ListNode* reverseList(ListNode* head) {
        if(head==nullptr) return nullptr;
        ListNode* last = nullptr;
        ListNode* next = head->next;
        while(next)
        {
            head->next = last;
            last = head;
            head = next;
            next = next->next;
        }
        head->next = last;
        return head;
    }
    ListNode* middleNode(ListNode* head){
        ListNode* fast = head,*slow = head;
        while(fast&&fast->next)
        {
            fast = fast->next->next;
            slow = slow->next;
        }
        return slow;
    }
    bool isPalindrome(ListNode* head) {
        ListNode* mid = middleNode(head);
        mid = reverseList(mid);
        while(mid&&head)
        {
            if(mid->val!=head->val) return false;
            mid = mid->next;
            head = head->next;
        }
        return true;
    }
};
```

找到中间的节点，然后反转一下，再从头比较。