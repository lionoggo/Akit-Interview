

# 122. [买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/description/)

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

## 解答思路

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

