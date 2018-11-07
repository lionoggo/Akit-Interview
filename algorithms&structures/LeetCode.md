## 比特与2比特字符

有两种特殊字符。第一种字符可以用一比特`0`来表示。第二种字符可以用两比特(`10` 或 `11`)来表示。

现给一个由若干比特组成的字符串。问最后一个字符是否必定为一个一比特字符。给定的字符串总是由0结束。

**示例 1:**

```
输入: 
bits = [1, 0, 0]
输出: True
解释: 
唯一的编码方式是一个两比特字符和一个一比特字符。所以最后一个字符是一比特字符。
```

**示例 2:**

```
输入: 
bits = [1, 1, 1, 0]
输出: False
解释: 
唯一的编码方式是两比特字符和两比特字符。所以最后一个字符不是一比特字符。
```

**注意:**

- `1 <= len(bits) <= 1000`.
- `bits[i]` 总是`0` 或 `1`.

解答思路:

由于10,11两个编码都是以1开头的,这意味着只要是以1开头的后面一个数必定是跟在这个1一起的字符编码.利用这一点：用一个指针从前向后走,遇到1就走两步,遇到0就走一步,看最后是不是走到n-1的位置,说明最后的0只能是单独存在的,否则走到n的位置就说明这个0是跟前面的1一起的.

```java
class Solution {
    public boolean isOneBitCharacter(int[] bits) {
        int i;
        for (i=0;i<bits.length-1;){
            if(bits[i]==0){
                i++;
            }else{
                i+=2;
            }
        }
  
        return (i==bits.length-1)?true:false;
        
    }
}
```

## 数组拆分


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

解答思路:排序数组后将相邻元素划分为一组，取其中较小的，实际上就是偶数位的元素。
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

## 买股票的最佳时机

给定一个数组，它的第 *i* 个元素是一支给定股票第 *i* 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票），设计一个算法来计算你所能获取的最大利润。

注意你不能在买入股票前卖出股票。

**示例 1:**

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
```

**示例 2:**

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

解答方案:我们需要找出给定数组中两个数字之间的最大差值（即，最大利润）。此外，第二个数字（卖出价格）必须大于第一个数字（买入价格）。

形式上，对于每组 ii 和 jj（其中 j &gt; ij>i）我们需要找出 \max(prices[j] - prices[i])max(prices[j]−prices[i])

根据这种思路,第一种暴力法使我们最容易想到的:

```java
class Solution {
    public int maxProfit(int[] prices) {
        int maxProfit=0;
        for(int i=0;i<prices.length;i++){
            for(int j=i+1;j<prices.length;j++){
                if(prices[j]-prices[i]>maxProfit){
                    maxProfit=prices[j]-prices[i];
                }else{
                    continue;
                }
            }
        }
        return maxProfit;
    }
}
```

上述过程中由于使用了两次循环,导致时间复杂度较高,即O(n<sup>2</sup>).换个思路就是能不能在一次循环中找出最小值之后的最大值.因此需要我们添加一个变量用来存储最小值.

```java
class Solution {
    public int maxProfit(int[] prices) {
        int maxProfit=0;
        int minPrice=Integer.MAX_VALUE;
        for (int i=0;i<prices.length;i++){
            if(minPrice>prices[i]){
                minPrice=prices[i];
            }else if(prices[i]-minPrice>maxProfit){
                maxProfit=prices[i]-minPrice;
            }
        }
        
        return maxProfit;
    }
}
```

- 时间复杂度：O(n)。只需要遍历一次。

## 买卖股票的最佳时机 II

给定一个数组，它的第 *i* 个元素是一支给定股票第 *i* 天的价格。

设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。

**注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```
输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
```

**示例 2:**

```
输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
```

**示例 3:**

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

解答思路:

对于股票交易而言,在不考虑交易成本的情况下,在每个爬升期内进行交易都是赚钱,其最大利润是在爬升起点购入,最高点卖出,也就是需要找个每个爬生期内的最低点和最高点,两点之间的差值就是该周期内所获得最大利润,如下所示:

![image-20181107111901774](https://i.imgur.com/fz94Brx.png)

根据上述描述,代码如下:

```java
class Solution {
    public int maxProfit(int[] prices) {
        int i = 0;
        int valley = prices[0];
        int peak = prices[0];
        int maxprofit = 0;
        while (i < prices.length - 1) {
            // 查找爬升期的低点
            while (i < prices.length - 1 && prices[i] >= prices[i + 1])
                i++;
            valley = prices[i];
            // 查找爬升期的高点
            while (i < prices.length - 1 && prices[i] <= prices[i + 1])
                i++;
            peak = prices[i];
            // 计算爬升期内的利润
            maxprofit += peak - valley;
        }
        return maxprofit;
    }
}
```

在上面的思路中我们需要寻找每个爬升期内的最地点和最高点,并计算他们的的差值,但我们发现这两点的差值其实等价于该周期内每天连续收益的利润,即如下所示:

![image-20181107112330832](https://i.imgur.com/dCTLCae.png)

根据上述描述,代码描述如下:

```java
class Solution {
    public int maxProfit(int[] prices) {
        int maxProfit=0;
        for (int i=0;i<prices.length-1;i++){
            // 爬升期累积所有利润
            if(prices[i+1]>prices[i]){
                maxProfit+=(prices[i+1]-prices[i]);
            }
        }
        return maxProfit;
    }
}
```

- 时间复杂度：O(n).

## 种花问题

假设你有一个很长的花坛，一部分地块种植了花，另一部分却没有。可是，花卉不能种植在相邻的地块上，它们会争夺水源，两者都会死去。

给定一个花坛（表示为一个数组包含0和1，其中0表示没种植花，1表示种植了花），和一个数 **n** 。能否在不打破种植规则的情况下种入 **n** 朵花？能则返回True，不能则返回False。

**示例 1:**

```
输入: flowerbed = [1,0,0,0,1], n = 1
输出: True
```

**示例 2:**

```
输入: flowerbed = [1,0,0,0,1], n = 2
输出: False
```

**注意:**

1. 数组内已种好的花不会违反种植规则。
2. 输入的数组长度范围为 [1, 20000]。
3. **n** 是非负整数，且不会超过输入数组的大小。

解答思路:什么的位置可以种花呢?前提是当前位置是0,假设种完后,其左右位置还是0,那么说明这是个真正可以种花的位置,因此代码思路如下:

```java
class Solution {
    public boolean canPlaceFlowers(int[] flowerbed, int n) {
        int num=0;
        for(int i=0;i<flowerbed.length;i++){
            if(flowerbed[i]==0){
                int right=(i==flowerbed.length-1)?0:flowerbed[i+1];
                int left=(i==0)?0:flowerbed[i-1];
                if(right==0&&left==0){
                    num++;
                    flowerbed[i]=1;
                }        
            }
        if(num>=n){
            return true;
           }
        }
        return false;
    }
}
```



## 存在重复元素

给定一个整数数组，判断是否存在重复元素。

如果任何值在数组中出现至少两次，函数返回 true。如果数组中每个元素都不相同，则返回 false。

**示例 1:**

```
输入: [1,2,3,1]
输出: true
```

**示例 2:**

```
输入: [1,2,3,4]
输出: false
```

**示例 3:**

```
输入: [1,1,1,3,3,4,3,2,4,2]
输出: true
```

解答思路:一般有两种,排序后进行前后位置比较,如果有重复,那么前后位置比较时一定会遇到相等的情况;另一种则是利用java集合的特性,不允许添加重复元素.

先来看第一种:

```java
class Solution {
    public boolean containsDuplicate(int[] nums) {
       if(nums.length<2){
           return false;
       }
        Arrays.sort(nums);
        for(int i=0;i<nums.length-1;i++){
            if(nums[i]==nums[i+1]){
                return true;
            }
        }
        return false;
    }
}
```

再来看第二种:

```java
class Solution {
    public boolean containsDuplicate(int[] nums) {
        if(nums.length<2){
            return false;
        }
        Set<Integer> set=new HashSet(nums.length);
        for(int i=0;i<nums.length;i++){
            if(set.contains(nums[i])){
                return true;
            }
            set.add(nums[i]);
        }
        return false;
    }
}
```

## 存在重复元素 II

------



给定一个整数数组和一个整数 *k*，判断数组中是否存在两个不同的索引 *i* 和 *j*，使得 **nums [i] = nums [j]**，并且 *i* 和 *j* 的差的绝对值最大为 *k*。

**示例 1:**

```
输入: nums = [1,2,3,1], k = 3
输出: true
```

**示例 2:**

```
输入: nums = [1,0,1,1], k = 1
输出: true
```

**示例 3:**

```
输入: nums = [1,2,3,1,2,3], k = 2
输出: false
```

解答思路:和上题类似,区别是我们需要使用Map结构同时保存值和数组下标.

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        if(nums.length<2){
            return false;
        }
        Map<Integer,Integer> map=new HashMap();
        for(int i=0;i<nums.length;i++){
            if(map.containsKey(nums[i])){
                int value=map.get(nums[i]);
                if(Math.abs(i-value)<=k){
                    return true;
                }
            }
            map.put(nums[i],i);
        }
        return false;
    }
}
```

