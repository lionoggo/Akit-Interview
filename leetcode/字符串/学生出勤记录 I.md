# 551. [学生出勤记录 I](https://leetcode-cn.com/problems/student-attendance-record-i/description/)

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

## 解答思路

该题比较简单,理解题意后可知以下两种情况返回false即可:

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