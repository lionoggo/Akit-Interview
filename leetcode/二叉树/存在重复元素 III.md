# 220. [Contains Duplicate III](https://leetcode-cn.com/problems/contains-duplicate-iii/description/)

给定一个整数数组，判断数组中是否有两个不同的索引 *i* 和 *j*，使得 **nums [i]** 和 **nums [j]** 的差的绝对值最大为 *t*，并且 *i* 和 *j* 之间的差的绝对值最大为 *ķ*。

**示例 1:**

```
输入: nums = [1,2,3,1], k = 3, t = 0
输出: true
```

**示例 2:**

```
输入: nums = [1,0,1,1], k = 1, t = 2
输出: true
```

**示例 3:**

```
输入: nums = [1,5,9,1,5,9], k = 2, t = 3
输出: false
```

## 解答思路

根据题意可得出以下条件,在一定范围内K,寻找满足该条件的数`-t <= x-nums[i] <=t`.不难想到此时可以使用滑动窗口来描述范围K,在该滑动窗口内查找满足上述条件的nums[i]和nums[j].

第一种思路,我们可以红黑树来表示滑动窗口,当添加元素超过K后,删除掉最早加入的元素即可.

```java
class Solution {
    public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
        if(k <= 0 || t < 0){
            return false;
        }
       
        TreeSet<Long> set=new TreeSet();
        for (int i = 0;i< nums.length;i++){
            // ceiling()方法返回set中大于等于给定元素的最小元素
            Long x =set.ceiling((long)nums[i]-t);
            if(x!=null&&x <= t+(long)nums[i]){
                return true;
            }
            // 限定红黑树中最多存放K个元素,超过则删除之前的.比如限定存放5个,当遍历到第6个
            // 时,删除第1个(即6-5),当遍历到第7个时,删除第2个(即7-5)
            if(i>= k){
                set.remove((long)nums[i-k]);
            }
            set.add((long)(nums[i]));
        }
        return false;
    }
}
```

