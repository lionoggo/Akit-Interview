# 429. [N叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/description/)

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

## 解答思路

和二叉树层序遍历思路一样

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