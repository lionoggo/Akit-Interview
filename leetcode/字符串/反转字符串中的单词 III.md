#  557. [反转字符串中的单词 III](https://leetcode-cn.com/problems/reverse-words-in-a-string-iii/description/)

给定一个字符串，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

**示例 1:**

```
输入: "Let's take LeetCode contest"
输出: "s'teL ekat edoCteeL tsetnoc" 
```

**注意：**在字符串中，每个单词由单个空格分隔，并且字符串中不会有任何额外的空格。

## 解答思路

在反转单个字符串的基础上进行的扩展.分割成字符串数组后,依次反转即可.

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

