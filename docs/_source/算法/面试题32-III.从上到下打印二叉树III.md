# 面试题32 - III. 从上到下打印二叉树 III

## 题目链接

[面试题32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

## 题目描述

Difficulty: **中等**

请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

例如:  
给定二叉树: `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其层次遍历结果：

```
[
  [3],
  [20,9],
  [15,7]
]
```

**提示：**

1. `节点总数 <= 1000`

## Solution

Language: **Java**

```java
​/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        queue.add(root);
        int flag = 0;
        while(!queue.isEmpty()){
            int cnt = queue.size();
            List<Integer> list = new ArrayList<>();
            while(cnt > 0){
                cnt--;
                TreeNode t = queue.poll();
                if(t == null) continue;
                list.add(t.val);
                queue.add(t.left);
                queue.add(t.right);
            }
            if(list.size() > 0){
                if(flag++ % 2 == 1) Collections.reverse(list);
                res.add(list);
            }
        }
        return res;
    }
}
```

当然，还可以使用LinkedList代替反转

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
       List<List<Integer>> ans = new ArrayList<>();
        if(root==null) return ans;

        Queue<TreeNode> q=new LinkedList<>();
        q.add(root);
        int k=0;
        while(!q.isEmpty()){
            LinkedList<Integer> temp=new LinkedList<>();
            int n=q.size();
            for(int i=0;i<n;i++){
                TreeNode con=q.poll();

                if(k%2==0){
                     temp.addLast(con.val);
                }
                if(k%2==1){
                    temp.addFirst(con.val);
                }
                TreeNode left=con.left;
                TreeNode right=con.right;
                if(left!=null) q.add(left);
                if(right!=null) q.add(right);
            }
            k++;
            ans.add(temp);
        }

        return ans;
    }
}
```