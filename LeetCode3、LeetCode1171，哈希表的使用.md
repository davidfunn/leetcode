# LeetCode3、LeetCode1171，哈希表的使用

## 题目描述：

* LeetCode3：给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

  ```
  输入: "abcabcbb"
  输出: 3 
  解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
  ```

  

* LeetCode1171：给你一个链表的头节点 head，请你编写代码，反复删去链表中由**总和**值为 0 的连续节点组成的序列，直到不存在这样的序列为止。删除完毕后，请你返回最终结果链表的头节点。

  ```
  输入：head = [1,2,-3,3,1]
  输出：[3,1]
  提示：答案 [1,2,1] 也是正确的。
  ```

## 题解

**LeetCode3：**使用双指针法。快指针fast_ptr遍历整个字符串，当遇到重复的字符后，慢指针slow_ptr放到重复字符的下一个位置，然后计算并更新长度。

* 关键点：怎么快速知道当前字符发生了重复？

  使用一个**哈希表**，每次快指针遍历的时候将字符的位置放入到**哈希表**。

以下是代码，一些细节在注释中说明，这里再总结下：

1、这种方法真的就只要判断字符出现过就可以了吗？

​	答案是否！==还必须判断这个字符是否在当前区间！==

​	举个例子，**t**m[**m**zux**t**]，若当前慢指针slow_ptr在第二个m处，fast_ptr在最后一个t处，它也需要去计算长度，而不是把慢指针放到t的下一个位置，虽然t已经出现过了(第一个)，**但是这个t并不在快慢指针的区间**。

2、存放字符的哈希表存放的是**当前位置**还是**当前位置+1的位置。**这两种方法都是可以的，但是实现不同。

* 存放**当前位置+1**的位置，代码如下。这种方法的巧妙之处是可以把哈希表直接初始化为0，并且避免了第一个字符(位置0)的位置造成的bug。因为位置0加了1，所以加入到哈希表时有了标识。

```c++
int lengthOfLongestSubstring(string s) {
         int ch_map[256] = {0};
             int fast_ptr = 0, slow_ptr = 0, max_len = 0;   
         for (int i = fast_ptr; i < s.size(); ++i)
         {
             if (ch_map[s[i]] == 0 || ch_map[s[i]] < slow_ptr)
             {//如果从没出现过，或者它之前出现过现在不在这个区间里
                 max_len = max( max_len, i - slow_ptr + 1 );
             }
             else slow_ptr = ch_map[s[i]];//否则慢指针移到重复字母后面
             ch_map[s[i]] = i + 1;//记录字符在string中的位置+1，+1是为了得到下一字符的位置，同时避免第一个字符为0         
         }
         return max_len;
     }
```

* 存放当前位置，代码如下。因为第一个位置为0，所以哈希表不能初始化为0。这时候我们必须把哈希表初始化为负数(-1或者其他)。同时每次慢指针还是要移动到重复字母的后面，那么必须要**手动+1**。

```c++
int lengthOfLongestSubstring(string s) {
    int ch_map[256];
    memset(ch_map, -1, 256 * sizeof(int));//将哈希表初始化为全-1
    int fast_ptr = 0, slow_ptr = 0, max_len = 0;   
    for (int i = fast_ptr; i < s.size(); ++i)
    {
        if (ch_map[s[i]] == -1 || ch_map[s[i]] < slow_ptr)
        {//如果从没出现过，或者它之前出现过现在不在这个区间里
            max_len = max( max_len, i - slow_ptr + 1);
        }
        else slow_ptr = ch_map[s[i]] + 1;//否则慢指针移到重复字母后面，这里要手动加一
        ch_map[s[i]] = i;//记录字符在string中的位置
    }
    return max_len;
}
```

---

**LeetCode1171：**使用一个前缀和数组来标识是否发生了重复。所谓前缀和，就是将整个数组从前往后加，比如[1, 2,(3, -3,) 4]，前缀和数组为[1, 3,(6, 3,) 7]。现在发现两项均为3，所以，我们把第一个3到第二个3之间(左开右闭)删除即可，即删除3和-3。删除操作为指针操作，让第一个3的next指向第二个3的next即可。

* 找到重复的元素用**哈希表**，可以节省了不少时间。具体做法是将每次计算出来的前缀和以及**当前的节点**放到哈希表中。
* 因为头结点也可能被消除，所以我们使用一个**哑结点**来指向头结点，方便处理。
* 消除了节点之后，必须把哈希表中对应的前缀和及节点**删除**，防止后续出现bug。

代码如下，一些细节在注释中说明：

```c++
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};
ListNode* removeZeroSumSublists(ListNode* head) {
    //保存前缀和
    unordered_map<int, ListNode*> prefixSum;
    //因为头结点也可能被消掉，所以这里加一个哑结点作为头结点
    ListNode *dummy = new ListNode(0), *p = dummy;
    dummy->next = head;
    prefixSum[0] = p;
    int cur = 0, tempCur = 0;
    while (p = p->next) {
        cur += p->val;
        if (prefixSum.find(cur) != prefixSum.end()) {//当找到相同的Sum
            ListNode *temp = prefixSum[cur]->next;
            prefixSum[cur]->next = p->next;//next指向next
            tempCur = cur;
            //还需要从map中删除消除区间的前缀和
            while (temp != p) {
                tempCur += temp->val;
                prefixSum.erase(tempCur);
                temp = temp->next;
            } 
        } else {
             prefixSum[cur] = p;
            }
    }
    return dummy->next;
}
```

## 总结：
哈希表的使用带来性能上的提升，同时利用**key->value**的映射，在许多细节上方便了处理。