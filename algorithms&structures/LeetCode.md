# 字符串

## 转成小写字母

实现函数 ToLowerCase()，该函数接收一个字符串参数 str，并将该字符串中的大写字母转换成小写字母，之后返回新的字符串。

 

**示例 1：**

```
输入: "Hello"
输出: "hello"
```

**示例 2：**

```
输入: "here"
输出: "here"
```

**示例** **3：**

```
输入: "LOVELY"
输出: "lovely"
```

解答思路:字符串在java中统一用unicode表示( 即utf-16 LE) ，而小写字母a的值比大写字母的值大32.

```java
class Solution {
    public String toLowerCase(String str) {
        if(str == null){
            return null;
        }
        char[] charArr = str.toCharArray();
        for(int i=0;i<charArr.length;i++){
            if(charArr[i]>='A'&&charArr[i]<='Z'){
                charArr[i]=(char)(charArr[i]+32);
            }
        }
        return new String(charArr);
    }
}
```

## 反转字符串

编写一个函数，其作用是将输入的字符串反转过来。

**示例 1:**

```
输入: "hello"
输出: "olleh"
```

**示例 2:**

```
输入: "A man, a plan, a canal: Panama"
输出: "amanaP :lanac a ,nalp a ,nam A"
```

解答思路: 最简单的思路是遍历字符数组,从后往前输出,此外你要知道StringBuilder提供了反转操作.

```java
class Solution {
    public String reverseString(String s) {
        return new StringBuilder(s).reverse().toString();
    }
}
```



## 反转字符串中的单词 III

给定一个字符串，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

**示例 1:**

```
输入: "Let's take LeetCode contest"
输出: "s'teL ekat edoCteeL tsetnoc" 
```

**注意：**在字符串中，每个单词由单个空格分隔，并且字符串中不会有任何额外的空格。

解答思路:在反转单个字符串的基础上进行的扩展.分割成字符串数组后,依次反转即可.

```java
class Solution {
    public String reverseWords(String s) {
        String[] strArr = s.split(" ");
        StringBuilder builder = new StringBuilder();
        for(int i=0;i<strArr.length;i++){
            builder.append(reverse(strArr[i])).append(" "); 
        }
        return builder.toString().trim();
    }
    
    private StringBuilder reverse(String word){
        return new StringBuilder(word).reverse();
    }
}
```



## 检测大写字母

给定一个单词，你需要判断单词的大写使用是否正确。

我们定义，在以下情况时，单词的大写用法是正确的：

1. 全部字母都是大写，比如"USA"。
2. 单词中所有字母都不是大写，比如"leetcode"。
3. 如果单词不只含有一个字母，只有首字母大写， 比如 "Google"。

否则，我们定义这个单词没有正确使用大写字母。

**示例 1:**

```
输入: "USA"
输出: True
```

**示例 2:**

```
输入: "FlaG"
输出: False
```

**注意:** 输入是由大写和小写拉丁字母组成的非空单词。

解答思路:一开始能考虑到的是两种情况:第一个字符是小写字母,其余的字母只能是小写才正确;第一个字符是大写字母的情况下,剩余字母全部是小写或者大写,加入一个计数器,遇到大写字符计数器加1,遇到小写字符,计数器减1,当计数器为字符长度-1时,意味着都是剩余字符都是大写或者小写.

```java
class Solution {
    public boolean detectCapitalUse(String word) {
        if(word.length()==1){
            return true;
        }
        char frist = word.charAt(0);
        if(isLowerChar(frist)){
            for(int i=1;i<word.length();i++){
                if(isUpperChar(word.charAt(i))){
                    return false;
                }
            }
            return true;
        }else {
            int count =0;
            for(int i=1;i<word.length();i++){
                if(isUpperChar(word.charAt(i))){
                    count ++;
                }
                if(isLowerChar(word.charAt(i))){
                    count --;
                }
            }
     
            if(Math.abs(count) == word.length()-1){
                return true;
            }else {
                return false;
            }
        }
             
    }
    
    private boolean isLowerChar(char ch){
        if(ch >= 'a' && ch <= 'z'){
            return true;
        }
        return false;
    }
    
    private boolean isUpperChar(char ch){
        if(ch >= 'A' && ch <= 'Z'){
            return true;
        }
        return false;
    }
}
```

在上述基础上,再来抽象一下.正确的单词中,大写字符的数目等于单词长度(全是大写),或者等于0(全是小写),或者等于1且首字符为大写.同样,采用计数器来统计大写字符数量.

```java
class Solution {
    
    public boolean detectCapitalUse(String word) {
        if ( word == null || word == "" ) return false;
        if (word.length() == 1 ) return true;
        int count = 0;
        for (int i = 0; i < word.length(); i++) {
            if (word.charAt(i) <= 'Z' && word.charAt(i) >= 'A')
                count++;
        }
        
        if (count == word.length() || (count == 1 && word.charAt(0) >= 'A' && word.charAt(0) <= 'Z') || count == 0)
            return true;
        return false;
    }
}
```

## 最后一个单词的长度

给定一个仅包含大小写字母和空格 `' '` 的字符串，返回其最后一个单词的长度。

如果不存在最后一个单词，请返回 0 。

**说明：**一个单词是指由字母组成，但不包含任何空格的字符串。

**示例:**

```
输入: "Hello World"
输出: 5
```

解答思路:直接使用split()分割单词当然可以.但实际上我们要计算的是最后一个空格出现之后字符串的长度,所以一种简单写法如下:

```java
class Solution {
    public int lengthOfLastWord(String s) {
        if( s == null){
            return 0;
        }
        s = s.trim();
        int lastIndex = s.lastIndexOf(' ');
        if( lastIndex == 0){
            return s.length();
        }
        return s.substring(lastIndex+1).length();
    }
}
```

## 字符串中的单词树

统计字符串中的单词个数，这里的单词指的是连续的不是空格的字符。

请注意，你可以假定字符串里不包括任何不可打印的字符。

**示例:**

```java
输入: "Hello, my name is John"
输出: 5
```

解答思路:根据单词的特点,我们可以加入两个指针,一个用来标记进入一个单词,一个用来比较立刻一个单词.当遇到非空格时,表明进入单词,否则表明立开一个单词.需要注意,对于只有一个单词的情况,只有进入没有退出操作.

```java
class Solution {
    public int countSegments(String s) {
        if(s == null || s.length() == 0){
            return 0;
        }
        int count =0;	// 单词计数器
        char in = 0;	// 单词进入
        char out = 0;	// 单词离开
        for (int i=0;i<s.length();i++){
            char tmp = s.charAt(i);
            if(tmp != ' '){
                in = 1;
                out =1;
            }else{
                out = 0;
            }
            if(in ==1&&out ==0){
                count++;
                in =0;
            }         
        }
        // 对于"hello"这种字符串,由于无法标记离开操作,因此在in为1时,增加count值即可.
        return in == 1?count+1:count;
    }
}
```

除此之外,还可以借助正则表达式,对字符串进行切分:

```java
class Solution {
    public int countSegments(String s) {
        if(s == null || s.length() == 0){
            return 0;
        }
        return s.trim().length() == 0?0:s.trim().split("\\s+").length;
    }
}
```

## 学生出勤记录I

给定一个字符串来代表一个学生的出勤纪录，这个纪录仅包含以下三个字符：

1. **'A'** : Absent，缺勤
2. **'L'** : Late，迟到
3. **'P'** : Present，到场

如果一个学生的出勤纪录中不**超过一个'A'(缺勤)**并且**不超过两个连续的'L'(迟到)**,那么这个学生会被奖赏。

你需要根据这个学生的出勤纪录判断他是否会被奖赏。

**示例 1:**

```
输入: "PPALLP"
输出: True
```

**示例 2:**

```
输入: "PPALLL"
输出: False
```

解答思路:该题比较简单,理解题意后可知以下两种情况返回false即可:

- 对字符串中的字符A计数,出现两个以上A就返回false
- 对字符串中的连续L计数,连续两个以上L返回false

在该思路下,代码如下所示:

```java
class Solution {
    public boolean checkRecord(String s) {
        int countA =0;
        int countL =0;
        for(int i=0;i<s.length();i++){
            char ch =s.charAt(i);
            if(ch == 'A'){
                countA++;
                countL=0;
                if(countA>1){
                    return false;
                }
            }else if(ch == 'L'){
                countL++;
                if(countL > 2){
                    return false;
                }
            }else {
                countL =0;
            }
        }
        return true;
    }
}
```

此外重新解读这句话:**超过一个'A'(缺勤)**并且**不超过两个连续的'L'(迟到)**,当A不存在时,其在字符串的索引应该为-1,当A只有一个时,通过`indexOf()`和`lastIndexOf()`返回的同一个位置;此外,LLL在字符串的索引应该为-1.同时满足这两者时返回true,否则返回false.

```java
class Solution {
    public boolean checkRecord(String s) {
        return s.indexOf('A') == s.lastIndexOf('A') && s.indexOf("LLL") == -1;
    }
}
```

这两种思路本质上是一样,从正反两个条件进行思路,一个是或操作,一个是与操作.

## 二进制求和

给定两个二进制字符串，返回他们的和（用二进制表示）。

输入为**非空**字符串且只包含数字 `1` 和 `0`。

**示例 1:**

```
输入: a = "11", b = "1"
输出: "100"
```

**示例 2:**

```
输入: a = "1010", b = "1011"
输出: "10101"
```

解答思路:计算过程中最重要的是掌握进位操作.以十进制8+5为例,个位值为`8+5 % 10 = 3`,进位为`8+5 /10 =1`.这对于任何进制的加法都是成立的.此外需要注意在char类型进行加法时,需要进行`-'0'`操作,因为在计算式char会自动提升为int类型,而`'1'`在ascii中的值是49,如果不进行处理将得到错误的值.`'0'`在ascii中的值48,`'1'-'0'`,即`49-48=1`此时才会得到正确的值.

```java
class Solution {
    public String addBinary(String a, String b) {
        // 使用保持a是较长的字符串,以方便处理
        if(a.length() < b.length()){
            String tmp = a;
            a = b;
            b = tmp;
        }
        int indexA = a.length() - 1;
        int indexB = b.length() - 1;
        String result = "";
        int carries = 0;	// 保存进位
        // 从右往左扫描,先处理较短字符串,比如1101和10,此时处理01和10部分
        while(indexB >= 0){
            int sum = (int)(a.charAt(indexA) - '0') + (int)(b.charAt(indexB) - '0') + carries;
            result = String.valueOf(sum % 2) + result;
            carries = sum / 2;
            indexB--;
            indexA--;
        }
        // 从右往左,继续处理较长剩余份,此时处理11部分
        while(indexA >= 0){
            int sum = (int)(a.charAt(indexA) - '0') + carries;
            result = String.valueOf(sum % 2) + result;
            carries = sum / 2;
            indexA--;
        }
        // 最后处理进位,比如计算1和1
        if(carries == 1){
            result = "1" + result;
        }
        return result;
    }
}
```

## 字符串相乘

给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。

**示例 1:**

```
输入: num1 = "2", num2 = "3"
输出: "6"
```

**示例 2:**

```
输入: num1 = "123", num2 = "456"
输出: "56088"
```

**说明：**

1. `num1` 和 `num2` 的长度小于110。
2. `num1` 和 `num2` 只包含数字 `0-9`。
3. `num1` 和 `num2` 均不以零开头，除非是数字 0 本身。
4. **不能使用任何标准库的大数类型（比如 BigInteger）**或**直接将输入转换为整数来处**

解答思路:该题是典型的用来处理大数的情况,在解答该题时,可以按照计算法则,将惩罚运算转为加法运算即可.比如计算`123 * 456`时,分别计算`123 * 4 = 492,123 * 5 = 615,123* 6 =738 `,按照计算最终结果是:`492 * 100+615*10+738 =56088 `.由于该种思路是我们思考乘法的常规思路,因此比较容易写出来,如图所示:

![image-20181120104252533](https://ws1.sinaimg.cn/large/006tNbRwly1fxec63r8n5j30n20egweu.jpg)

```java
class Solution {
    public String multiply(String num1, String num2) {
        if ("0".equals(num1) || "0".equals(num2)) {
            return "0";
        }

        String result = "";
        // 将两个数的乘法操作转为加法操作.
        for (int i = num2.length() - 1; i >= 0; i--) {
            int times = num2.charAt(i) - '0';
            String tmpResult = "";
            for (int j = 0; j < times; j++) {
                tmpResult = add(num1, tmpResult);
            }
            // 补0操作
            for (int z = 0; z < num2.length() - i - 1; z++) {
                tmpResult = tmpResult + "0";
            }
            result = add(tmpResult, result);
        }
        return result;
    }
    // 处理两个String的加法操作
    private String add(String num1, String num2) {
        if (num2.length() <= 0) {
            return num1;
        }
        if (num1.length() < num2.length()) {
            String tmp = num1;
            num1 = num2;
            num2 = tmp;
        }
        int index1 = num1.length() - 1;
        int index2 = num2.length() - 1;
        int carries = 0;
        String result = "";
        while (index2 >= 0) {
            int sum = (num1.charAt(index1) - '0') + (num2.charAt(index2) - '0') + carries;
            result = sum % 10 + result;
            carries = sum / 10;
            index2--;
            index1--;
        }
        while (index1 >= 0) {
            int sum = (num1.charAt(index1) - '0') + carries;
            result = sum % 10 + result;
            carries = sum / 10;
            index1--;
        }
        if (carries > 0) {
            result = carries + result;
        }
        return result;

    }
}
```









# 数组

##  比特与2比特字符

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

## 数组的度

给定一个非空且只包含非负数的整数数组 `nums`, 数组的度的定义是指数组里任一元素出现频数的最大值。

你的任务是找到与 `nums` 拥有相同大小的度的最短连续子数组，返回其长度。

**示例 1:**

```
输入: [1, 2, 2, 3, 1]
输出: 2
解释: 
输入数组的度是2，因为元素1和2的出现频数最大，均为2.
连续子数组里面拥有相同度的有如下所示:
[1, 2, 2, 3, 1], [1, 2, 2, 3], [2, 2, 3, 1], [1, 2, 2], [2, 2, 3], [2, 2]
最短连续子数组[2, 2]的长度为2，所以返回2.
```

**示例 2:**

```
输入: [1,2,2,3,1,4,2]
输出: 6
```

**注意:**

- `nums.length` 在1到50,000区间范围内。
- `nums[i]` 是一个在0到49,999范围内的整数

解答思路:通过上述描述不难发现整个过程如下分三步： 

1. 找到数组的度（即最大出现次数） 
2. 找到和数组的度相同的数（即出现次数和度相同） 
3. 找到数组中这些数出现的连续子数组

对于问题 1,借助 Map 的特性,对每个出现的数计频数:key 保存数,value 保存频数,最大的频数就是数组的度了 

对于问题 2 和 3,遍历保存的 Map,当数组的度和当前数的度相等时,进入连续子数组长度判断,找到最小的,代码如下:

```java
class Solution {
    public int findShortestSubArray(int[] nums) {
        int max=1;
        int min=Integer.MAX_VALUE;
        Map<Integer,Integer> map=new HashMap<Integer,Integer>();
        for(int i=0;i<nums.length;i++){
            if(map.containsKey(nums[i])){
                int value=map.get(nums[i])+1;
                map.put(nums[i],value);
                max=Math.max(max,value);
            }else {
                map.put(nums[i],1);
            }
        }
        
        for (Integer tmp:map.keySet()){
            if(max==map.get(tmp)){
               min=Math.min(min,findSubArray(nums,tmp));
            }
        }
        return min;
    }
    
    public int findSubArray(int[] nums,int key){
        int left=0;
        int right=nums.length-1;
        while(left<nums.length-1&&left<right){
            if(nums[left]!=key){
                left++;
            }else {
                break;
            }
        }
        while(right>0&&right>left){
            if(nums[right]!=key){
                right--;
            }else{
                break;
            }
        }
        return right-left+1;
    }
}
```

## 公平的糖果交换

爱丽丝和鲍勃有不同大小的糖果棒：`A[i]` 是爱丽丝拥有的第 `i` 块糖的大小，`B[j]` 是鲍勃拥有的第 `j` 块糖的大小。

因为他们是朋友，所以他们想交换一个糖果棒，这样交换后，他们都有相同的糖果总量。*（一个人拥有的糖果总量是他们拥有的糖果棒大小的总和。）*

返回一个整数数组 `ans`，其中 `ans[0]` 是爱丽丝必须交换的糖果棒的大小，`ans[1]` 是 Bob 必须交换的糖果棒的大小。

如果有多个答案，你可以返回其中任何一个。保证答案存在。

 

**示例 1：**

```
输入：A = [1,1], B = [2,2]
输出：[1,2]
```

**示例 2：**

```
输入：A = [1,2], B = [2,3]
输出：[1,2]
```

**示例 3：**

```
输入：A = [2], B = [1,3]
输出：[2,3]
```

**示例 4：**

```
输入：A = [1,2,5], B = [2,4]
输出：[5,4]
```

 **提示：**

- `1 <= A.length <= 10000`
- `1 <= B.length <= 10000`
- `1 <= A[i] <= 100000`
- `1 <= B[i] <= 100000`
- 保证爱丽丝与鲍勃的糖果总量不同。
- 答案肯定存在

解答思路:要想使两边的糖果总量相等,需要先计算两边的差值.比如A=[1,1],B=[2,2],两边的差值为sum(B)-sum(A)=2,因此需要对A数组中的某个元素加2/1,且加完之后的值是数组B中的某个元素.根据以上思路,代码如下:

```java
class Solution {
    public int[] fairCandySwap(int[] A, int[] B) {
        int[] fair=new int[2];
        int sumA=sumArray(A);
        int sumB=sumArray(B);
        int diffValue=Math.abs(sumA-sumB)/2;
        if(sumA<sumB){
            for(int i=0;i<A.length;i++){
                if(isExist(B,A[i]+diffValue)){
                    fair[0]=A[i];
                    fair[1]=A[i]+diffValue;
                    break;
                }
            }
        }else if(sumA>sumB){
           for(int i=0;i<B.length;i++){
               if(isExist(A,B[i]+diffValue)){
                   fair[1]=B[i];
                   fair[0]=B[i]+diffValue;
                   break;
               }
           } 
        }
        return fair;
    }
    // 计算数组和
    public int sumArray(int[] arr){
        int sum=0;
        for(int i=0;i<arr.length;i++){
            sum+=arr[i];
        }
        return sum;
    }
    // 判断value是否在数组中存在
    public boolean isExist(int[] arr,int value){
        for(int i=0;i<arr.length;i++){
            if(arr[i]==value){
                return true;
            }
        }
        return false;
    }
}
```

## 找到所有数组中消失的数字

给定一个范围在  1 ≤ a[i] ≤ *n* ( *n* = 数组大小 ) 的 整型数组，数组中的元素一些出现了两次，另一些只出现一次。

找到所有在 [1, *n*] 范围之间没有出现在数组中的数字。

您能在不使用额外空间且时间复杂度为*O(n)*的情况下完成这个任务吗? 你可以假定返回的数组不算在额外空间内。

**示例:**

```
输入:
[4,3,2,7,8,2,3,1]

输出:
[5,6]
```

解答思路:在不使用额外空间的情况下,原数组要想表示更多的信息,最典型的做法是要将数组中的元素值和索引建立联系起来,但前提是元素值满足1 ≤ a[i] ≤ *n*,因此该题的解决思路就是讲元素值转为数组的索引,比如示例中的4变成索引值3,然后对索引值为3处的元素取反,即将元素值变为-7,此时原数组为[4,3,2,-7,8,2,3,1],通过这种方式,最终只需要遍历最后的数组,如果某个索引值index对应的元素是非负数,即原数组中不存在index+1这个数.

代码如下:

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> result=new ArrayList();
        for(int i=0;i<nums.length;i++){
            int index=nums[i]>0?nums[i]:-nums[i];
            // index等于元素值-1
            index--;
            if(nums[index]>0){
                nums[index]=-nums[index];
            }       
        }
        for(int i=0;i<nums.length;i++){
            if(nums[i]>0){
                result.add(i+1);
            }
        }
        return result;
    }
}
```

## 两个数组的交集

给定两个数组，编写一个函数来计算它们的交集。

**示例 1:**

```
输入: nums1 = [1,2,2,1], nums2 = [2,2]
输出: [2]
```

**示例 2:**

```
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [9,4]
```

**说明:**

- 输出结果中的每个元素一定是唯一的。
- 我们可以不考虑输出结果的顺序。

解答思路:第一种思路是将一个数组放到Map结构中,然后遍历第二个数组,利用Map结构不存在重复元素的特性来统计.另外一种是分别对数组进行排序,遍历其中一个数组,使用二分查找元素是否存在于另一个数组中.

```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        
        Set<Integer> result = new HashSet();
        Arrays.sort(nums1);
        Arrays.sort(nums2);
       
        for (int i=0;i<nums1.length;i++){
                if(Arrays.binarySearch(nums2,nums1[i])>=0){
                    result.add(nums1[i]);
                }
            }
        
        return toArray(result);
    }
    
    private int[] toArray(Set<Integer> set){
        int[] result =new int[set.size()];
        int index=0;
        for(Integer in: set){
            result[index]=in;
            index++;
        }
        return result;
    }
}
```





# 链表

## 删除链表中的节点

请编写一个函数，使其可以删除某个链表中给定的（非末尾）节点，你将只被给定要求被删除的节点。

现有一个链表 -- head = [4,5,1,9]，它可以表示为:

```
    4 -> 5 -> 1 -> 9
```

**示例 1:**

```
输入: head = [4,5,1,9], node = 5
输出: [4,1,9]
解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
```

**示例 2:**

```
输入: head = [4,5,1,9], node = 1
输出: [4,5,9]
解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
```

**说明:**

- 链表至少包含两个节点。
- 链表中所有节点的值都是唯一的。
- 给定的节点为非末尾节点并且一定是链表中的一个有效节点。
- 不要从你的函数中返回任何结果

解答思路:当要删除一个节点时,对需要该节点而言,我们只需要用下一个节点的数值赋值当前节点即可.

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public void deleteNode(ListNode node) {
       // node为要删除的节点,只需要用node节点的下一个节点元素的值覆盖node节点即可实现删除操作 
       node.val=node.next.val;
       node.next=node.next.next;
    }
}
```



# 二分查找

## 二分查找算法

给定一个 `n` 个元素有序的（升序）整型数组 `nums` 和一个目标值 `target`  ，写一个函数搜索 `nums` 中的 `target`，如果目标值存在返回下标，否则返回 `-1`。


**示例 1:**

```
输入: nums = [-1,0,3,5,9,12], target = 9
输出: 4
解释: 9 出现在 nums 中并且下标为 4
```

**示例 2:**

```
输入: nums = [-1,0,3,5,9,12], target = 2
输出: -1
解释: 2 不存在 nums 中因此返回 -1
```

 

**提示：**

1. 你可以假设 `nums` 中的所有元素是不重复的。
2. `n` 将在 `[1, 10000]`之间。
3. `nums` 的每个元素都将在 `[-9999, 9999]`之间。

解答思路:典型的二分查找,代码如下:

```java
class Solution {
    public int search(int[] nums, int target) {
        int left=0;
        int right=nums.length-1;
        while(left<=right){
            int partion=left+(right-left)/2;
            if(nums[partion]==target){
                return partion;
            }else if(target<nums[partion]){
                right=partion-1;
            }else if(target>nums[partion]){
                left=partion+1;
            }
        }
        return -1;
    }
}
```

## 搜索二维矩阵

编写一个高效的算法来搜索 *m* x *n* 矩阵 matrix 中的一个目标值 target。该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

**示例:**

现有矩阵 matrix 如下：

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

给定 target = `5`，返回 `true`。

给定 target = `20`，返回 `false`。

解答思路:有组每行每列的数组都是有序的,因此我们可以考虑二分查找的方式.

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int rows = matrix.length;
        for (int i=0;i<rows;i++){
            if(Arrays.binarySearch(matrix[i],target) >=0){
                return true;
            }
        }
        return false;
    }
    
}
```



# 二叉树

## 二叉树最大深度

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

**说明:** 叶子节点是指没有子节点的节点。

**示例：**
给定二叉树 `[3,9,20,null,null,15,7]`，

```
    3
   / \
  9  20
    /  \
   15   7
```

返回它的最大深度 3 。

解答思路:对于这种问题,递归是最简单的思想.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int maxDepth(TreeNode root) {
        if(root == null){
            return 0;
        }
        return Math.max(maxDepth(root.left),maxDepth(root.right))+1;
    }
}
```

- 时间复杂度：我们每个结点只访问一次，因此时间复杂度为O(N)， 其中 N是结点的数量。
- 空间复杂度：在最糟糕的情况下，树是完全不平衡的，*例如*每个结点只剩下左子结点，递归将会被调用 N次（树的高度），因此保持调用栈的存储将是 O(N)。但在最好的情况下（树是完全平衡的），树的高度将是 log(N)。因此，在这种情况下的空间复杂度将是 O(log(N))。

## 反转二叉树

翻转一棵二叉树。

**示例：**

输入：

```
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```

输出：

```
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

解答思路:对于树这种结构,递归是最简单的解题思路

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root == null){
            return null;
        }
        TreeNode tmp=root.left;
        root.left=root.right;
        root.right=tmp;
        invertTree(root.left);
        invertTree(root.right);
        return root;
    }
    
}
```

## 合并二叉树

给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。

你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则**不为** NULL 的节点将直接作为新二叉树的节点。

**示例 1:**

```
输入: 
	Tree 1                     Tree 2                  
          1                         2                             
         / \                       / \                            
        3   2                     1   3                        
       /                           \   \                      
      5                             4   7                  
输出: 
合并后的树:
	     3
	    / \
	   4   5
	  / \   \ 
	 5   4   7
```

**注意:** 合并必须从两个树的根节点开始。

解答思路:对于合并二叉树时,将问题范围缩小,先考虑如何合并两个TreeNode,在这期间需要考三种情况:_

- 两棵树对应节点都存在的情况
- 对应位置t1树存在,t2不存在
- 对应位置t1树不存在,t2存在

接下就是要用递归的方式写出来.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if(t1==null&&t2==null){ // 都不存在
            return null;
        }
        if(t1==null&&t2!=null){ //t1不存在,t2存在
            return t2;
        }
        if(t1!=null&&t2==null){ //t1存在,t2存在
            return t1;
        }
       
        t1.val+=t2.val;
        t1.left=mergeTrees(t1.left,t2.left);
        t1.right=mergeTrees(t1.right,t2.right);
        return t1;
    }
    
}
```

## N叉树的前序遍历

给定一个 N 叉树，返回其节点值的*前序遍历*。

例如，给定一个 `3叉树` :

 

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png)

 

返回其前序遍历: `[1,3,5,6,2,4]`。

 

**说明:** 递归法很简单，你可以使用迭代法完成此题吗?

解答思路:对于前序遍历而言,每次首先访问根节点,然后再访问子节点.

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public List<Node> children;

    public Node() {}

    public Node(int _val,List<Node> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
    public List<Integer> preorder(Node root) {
        if(root==null){
            return Collections.emptyList();
        }
        List<Integer> result =new ArrayList<Integer>();
        result.add(root.val);
        for (int i=0;i<root.children.size();i++){
            result.addAll(preorder(root.children.get(i)));
        }
        return result;
    }
}
```

当然也可以采用迭代的方式,使用栈结构模拟即可:

```java
class Solution {
    public List<Integer> preorder(Node root) {
        Stack<Node> stack = new Stack<Node>();
        List<Integer> result=new ArrayList();
        if(root==null){
            return result;
        }
        stack.push(root);
        while(!stack.isEmpty()){
            Node cur=stack.pop();
            result.add(cur.val);
            for(int i=cur.children.size()-1;i>=0;i--){
             stack.push(cur.children.get(i));
            }
        }
        return result;   
    }
}
```


给定一个 N 叉树，找到其最大深度。

最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。

例如，给定一个 `3叉树` :

 

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png)

 

我们应返回其最大深度，3。

**说明:**

1. 树的深度不会超过 `1000`。
2. 树的节点总不会超过 `5000`。

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public List<Node> children;

    public Node() {}

    public Node(int _val,List<Node> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
   
    public int maxDepth(Node root) {
        if(root == null){
            return 0;
        }
        int max=0;
        for(Node child :root.children){
            int childDepth=maxDepth(child);
            max=Math.max(max,childDepth);
        }
        return max+1;
    }
}
```

## 二叉树层次遍历

给定一个二叉树，返回其节点值自底向上的层次遍历。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

例如：
给定二叉树 `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其自底向上的层次遍历为：

```
[
  [15,7],
  [9,20],
  [3]
]
```

解答思路:有关层次遍历时,借助队列.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        List<List<Integer>> result =new ArrayList();
        if(root ==null){
            return result;
        }
        
        List<TreeNode> curNodeList =new ArrayList();
        curNodeList.add(root);
        while(!curNodeList.isEmpty()){
            List<Integer> curValList=new ArrayList();
            List<TreeNode> nextNodeList=new ArrayList();
            for(TreeNode cur : curNodeList){
                curValList.add(cur.val);
                if(cur.left!=null)nextNodeList.add(cur.left);
                if(cur.right!=null)nextNodeList.add(cur.right);
            }
            result.add(0,curValList);
            curNodeList=nextNodeList;   
        }
        
        return result;
    }  
    
}
```

## N叉树层序遍历

给定一个 N 叉树，返回其节点值的*层序遍历*。 (即从左到右，逐层遍历)。

例如，给定一个 `3叉树` :

 

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png)

 

返回其层序遍历:

```
[
     [1],
     [3,2,4],
     [5,6]
]
```

 

**说明:**

1. 树的深度不会超过 `1000`。
2. 树的节点总数不会超过 `5000`。

解答思路:和二叉树层序遍历思路一样

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public List<Node> children;

    public Node() {}

    public Node(int _val,List<Node> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
    public List<List<Integer>> levelOrder(Node root) {
        List<List<Integer>> result = new ArrayList<>();
        if(root ==null){
            return result;
        }
        List<Node> curNodeList =new ArrayList();
        curNodeList.add(root);
        while(!curNodeList.isEmpty()){
            List<Integer> curValList=new ArrayList();
            List<Node> nextNodeList = new ArrayList();
            for(Node cur : curNodeList){
                curValList.add(cur.val);
                nextNodeList.addAll(cur.children);
            }
            result.add(curValList);
            curNodeList=nextNodeList;
        }
        return result;
    }
}
```

## 二叉树的层平均值

给定一个非空二叉树, 返回一个由每层节点平均值组成的数组.

**示例 1:**

```
输入:
    3
   / \
  9  20
    /  \
   15   7
输出: [3, 14.5, 11]
解释:
第0层的平均值是 3,  第1层是 14.5, 第2层是 11. 因此返回 [3, 14.5, 11].
```

**注意：**

1. 节点值的范围在32位有符号整数范围内。

解答思路:该问题的本质是层次遍历.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<Double> averageOfLevels(TreeNode root) {
        List<Double> result = new ArrayList();
        if(root == null){
            return result;
        }
        List<TreeNode> curNodeList = new ArrayList();
        curNodeList.add(root);
    
        while(!curNodeList.isEmpty()){
            double sum=0d;
            List<TreeNode> list =new ArrayList();
            for(TreeNode cur: curNodeList){
                sum+=cur.val;
                if(cur.left!=null)list.add(cur.left);
                if(cur.right!=null)list.add(cur.right);
            }
            double avr=sum/curNodeList.size();
            result.add(avr);
            curNodeList=list;
        }
        return result;
    }
}
```

## 二叉树的所有路径

给定一个二叉树，返回所有从根节点到叶子节点的路径。

**说明:** 叶子节点是指没有子节点的节点。

**示例:**

```
输入:

   1
 /   \
2     3
 \
  5

输出: ["1->2->5", "1->3"]

解释: 所有根节点到叶子节点的路径为: 1->2->5, 1->3
```

解答思路:递归搜索的过程本身就是保证顺序的,因此用递归的方式可以方便的实现.只有当一个节点没有左右节点时,此时即认为到达了叶子节点,可以将路径添加到集合中了.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> pathArr=new ArrayList();
        if(root == null){
            return pathArr;
        }
        findAndAdd(pathArr,root,"");
        return pathArr;
    }
    
    private void findAndAdd(List<String> paths,TreeNode root,String basePath){
        if(root == null){
            return ;
        }
        String path=basePath+root.val;
        if(root.left!=null){
            findAndAdd(paths,root.left,path+"->");
        }
        
        if(root.right!=null){
            findAndAdd(paths,root.right,path+"->");
        }
        if(root.left==null&&root.right==null){
            paths.add(path);
        }
    }
    
    
}
```

## 相同的树

给定两个二叉树，编写一个函数来检验它们是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

**示例 1:**

```
输入:       1         1
          / \       / \
         2   3     2   3

        [1,2,3],   [1,2,3]

输出: true
```

**示例 2:**

```
输入:      1          1
          /           \
         2             2

        [1,2],     [1,null,2]

输出: false
```

**示例 3:**

```
输入:       1         1
          / \       / \
         2   1     1   2

        [1,2,1],   [1,1,2]

输出: false
```

解答思路:说白了就比比较对应位置的值是否相等.使用递归时,尤其需要注意递归和停止递归的条件.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if(p == null && q == null){
            return true;
        }
        if(p == null || q == null){
            return false;
        }
        if(p.val == q.val){
            if(isSameTree(p.left,q.left) && isSameTree(p.right,q.right)){
               return true;
           }
        }     
        return false;
    }
}
```

## 左叶子之和

计算给定二叉树的所有左叶子之和。

**示例：**

```
    3
   / \
  9  20
    /  \
   15   7

在这个二叉树中，有两个左叶子，分别是 9 和 15，所以返回 24
```

 解答思路:分析好左叶子的情况以及递归的情况.

```java
class Solution {
    int sum=0;
    public int sumOfLeftLeaves(TreeNode root) {
        if(root == null){
            return sum;
        }
        if(root.left!=null){
            if(root.left.left==null&&root.left.right==null){
                sum+=root.left.val;
            }
        }
        sumOfLeftLeaves(root.left);
        sumOfLeftLeaves(root.right);
        return sum;
    }   
}
```

## 路径总和

给定一个二叉树和一个目标和，判断该树中是否存在根节点到叶子节点的路径，这条路径上所有节点值相加等于目标和。

说明: 叶子节点是指没有子节点的节点。

示例: 
给定如下二叉树，以及目标和 sum = 22，

              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
返回 true, 因为存在目标和为 22 的根节点到叶子节点的路径 5->4->11->2。

解答思路,考虑三种情况,根节点为null的情况,只有一个跟节点以及正常树的情况.该问题非常类似查找二叉树中所有路径中问题.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int sum;
    public boolean hasPathSum(TreeNode root, int sum) {
        if(root==null){
            return false;
        }
        sum=sum-root.val;
        if(root.left==null&&root.right==null){
            return sum==0;
        } 
        return hasPathSum(root.left,sum)||hasPathSum(root.right,sum);
    }
}
```

# 图

## 钥匙和房间?

有 `N` 个房间，开始时你位于 `0` 号房间。每个房间有不同的号码：`0，1，2，...，N-1`，并且房间里可能有一些钥匙能使你进入下一个房间。

在形式上，对于每个房间 `i` 都有一个钥匙列表 `rooms[i]`，每个钥匙 `rooms[i][j]` 由 `[0,1，...，N-1]` 中的一个整数表示，其中 `N = rooms.length`。 钥匙 `rooms[i][j] = v` 可以打开编号为 `v` 的房间。

最初，除 `0` 号房间外的其余所有房间都被锁住。

你可以自由地在房间之间来回走动。

如果能进入每个房间返回 `true`，否则返回 `false`。



**示例 1：**

```
输入: [[1],[2],[3],[]]
输出: true
解释:  
我们从 0 号房间开始，拿到钥匙 1。
之后我们去 1 号房间，拿到钥匙 2。
然后我们去 2 号房间，拿到钥匙 3。
最后我们去了 3 号房间。
由于我们能够进入每个房间，我们返回 true。
```

**示例 2：**

```
输入：[[1,3],[3,0,1],[2],[0]]
输出：false
解释：我们不能进入 2 号房间。
```

**提示：**

1. `1 <= rooms.length <= 1000`
2. `0 <= rooms[i].length <= 1000`
3. 所有房间中的钥匙数量总计不超过 `3000`。

解答思路:拿到该题时,首先要读懂,本质上就是图的遍历.这里我们采用广度优先的方式.在遍历过程中,标记已经进去过的房间.

```java
class Solution {
    public boolean canVisitAllRooms(List<List<Integer>> rooms) {
        if(rooms == null || rooms.isEmpty()){
            return false;
        }
        Queue<Integer> queue=new LinkedList();
        boolean[] entryRooms = new boolean[rooms.size()];
        entryRooms[0]=true;
        
        List<Integer> defaultKeys = rooms.get(0);
        for (int i=0;i<defaultKeys.size();i++){
            queue.offer(defaultKeys.get(i));
        }
        
        while(!queue.isEmpty()){
            Integer key =queue.poll();
            if(entryRooms[key])continue;
            entryRooms[key]=true;
            for (Integer k : rooms.get(key)){
                queue.offer(k);
            }
        }
        
        for (int i=0;i<entryRooms.length;i++){
            if(!entryRooms[i]){
                return false;
            }
        }
        return true;
    }
}
```

