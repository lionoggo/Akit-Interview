# 561. [数组拆分 I](https://leetcode-cn.com/problems/array-partition-i/description/)

给定长度为 **2n** 的数组, 你的任务是将这些数分成 **n** 对, 例如 (a1, b1), (a2, b2), ..., (an, bn) ，使得从1 到 n 的 min(ai, bi) 总和最大。

**示例 1:**

```
输入: [1,4,3,2]

输出: 4
解释: n 等于 2, 最大总和为 4 = min(1, 2) + min(3, 4).
```

**提示:**

1. **n** 是正整数,范围在 [1, 10000].
2. 数组中的元素范围在 [-10000, 10000].

## 解答思路

排序数组后将相邻元素划分为一组，取其中较小的，实际上就是偶数位的元素。
不过本题只要求算法返回最小的和，可理解位2n个元素中取n个使得总和最小，可直接排序后输出前n个和即可。

```java
class Solution {
    public int arrayPairSum(int[] nums) {
        int sum=0;
        Arrays.sort(nums);
        for(int i=0;i<nums.length;i+=2){
            sum+=nums[i];
        }
        return sum;
    }
}
```

