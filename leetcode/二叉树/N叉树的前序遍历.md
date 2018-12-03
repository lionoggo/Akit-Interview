# 589. [N叉树的前序遍历](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/description/)

给定一个 N 叉树，返回其节点值的*前序遍历*。

例如，给定一个 `3叉树` :

 

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png)

 

返回其前序遍历: `[1,3,5,6,2,4]`。

 

**说明:** 递归法很简单，你可以使用迭代法完成此题吗?

## 解答思路

对于前序遍历而言,每次首先访问根节点,然后再访问子节点.

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