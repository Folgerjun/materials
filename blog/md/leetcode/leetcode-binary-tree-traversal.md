---
title: LeetCode 之二叉树的各种遍历（Binary Tree Traversal）
date: 2018-11-27 10:35:34
categories: [开发,算法]
tags: [Java,算法,LeetCode]
---

在计算机科学中，二叉树是每个结点最多有两个子树的树结构。通常子树被称作“左子树”（left subtree）和“右子树”（right subtree）。二叉树常被用于实现二叉查找树和二叉堆。

想必大家对二叉树也不陌生，被各种二叉树面试题支配的恐惧仍记忆犹新……

这篇就总结一下二叉树的各种遍历，包括前、中、后序遍历还有层次遍历。

让我们来想象，大脑是个无底洞，这个栈它没有深度，所以我们要时而把栈底那些强行挖上来，以防痴呆！你不想痴呆吧！go go go.

## 层次遍历
先来这个层次遍历，二叉树一层又一层，它有深度。既然是一层接着一层，那很清楚了，我们逐层遍历就行。

题目描述：
```
给定一个二叉树，返回其按层次遍历的节点值。 （即逐层地，从左到右访问所有节点）。

例如:
给定二叉树: [3,9,20,null,null,15,7],

    3
   / \
  9  20
    /  \
   15   7
返回其层次遍历结果：

[
  [3],
  [9,20],
  [15,7]
]
```

清晰得不能再清晰了。递归调用 + 循环节点。代码如下：
```
public class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }
    }

    public List<List<Integer>> levelOrder(TreeNode root) {

        List<List<Integer>> list = new ArrayList<>();
        List<TreeNode> l = new ArrayList<>();
        l.add(root);
        helper(list, l);
        return list;

    }

    /**
     * 递归 层层遍历
     * 
     * @param list
     * @param treeList
     */
    private void helper(List<List<Integer>> list, List<TreeNode> treeList) {
        if (treeList.size() == 0)
            return;
        List<Integer> listInt = new ArrayList<>();
        List<TreeNode> treeL = new ArrayList<>();
        // 逐层添值
        for (TreeNode node : treeList) {
            if (node != null) {
                listInt.add(node.val);
                treeL.add(node.left);
                treeL.add(node.right);
            }
        }
        if (listInt.size() > 0)
            list.add(listInt);

        helper(list, treeL);
    }
```

或者是使用队列 + while 循环搞定：
```
/**
     * 使用队列 queue
     * 
     * @param root
     * @return
     */
    public List<List<Integer>> levelOrder1(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        List<List<Integer>> wrapList = new LinkedList<List<Integer>>();

        if (root == null)
            return wrapList;

        queue.offer(root);
        while (!queue.isEmpty()) {
            // 有值就塞入集合 同时将其左右子节点添加到队列中
            int levelNum = queue.size();
            List<Integer> subList = new LinkedList<Integer>();
            for (int i = 0; i < levelNum; i++) {
                if (queue.peek().left != null)
                    queue.offer(queue.peek().left);
                if (queue.peek().right != null)
                    queue.offer(queue.peek().right);
                subList.add(queue.poll().val);
            }
            wrapList.add(subList);
        }
        return wrapList;
    }
```

这个问题不大。

## 中序遍历
中序遍历（LDR）是二叉树遍历的一种，也叫做中根遍历、中序周游。在二叉树中，中序遍历首先遍历左子树，然后访问根结点，最后遍历右子树。

也就是根结点遍历位置在中间： 左 -> 根 -> 右

题目描述：
```
给定一个二叉树，返回它的中序 遍历。

示例:

输入: [1,null,2,3]
   1
    \
     2
    /
   3

输出: [1,3,2]
进阶: 递归算法很简单，你可以通过迭代算法完成吗？
```

既然都说了递归算法很简单，确实也很简单，先上递归算法：
```
public class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }
    }

    /**
     * 递归
     * 
     * @param root
     * @return
     */
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> list = new ArrayList<>();
        helper(list, root);
        return list;
    }

    private void helper(List<Integer> list, TreeNode node) {
        if (node != null) {
            if (node.left != null)
                helper(list, node.left);
            list.add(node.val);
            if (node.right != null)
                helper(list, node.right);
        }
    }
```

**这个递归算法在前、中、后序遍历都可以套用的，着实好用也好记。下面就不重复该代码了，无非就是结点的顺序换一换。**

迭代算法，这里我们思考一下，我们需要先读其左结点，读完之后再读其根结点，左结点若是存在，那它不就是下一层的根结点吗？然后我们再一层层往上读。此时脑壳一抖，栈！

Stack 来解决：
```
    /**
     * 栈 stack 来解决
     * 
     * @param root
     * @return
     */
    public List<Integer> inorderTraversal1(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode curr = root;
        while (curr != null || !stack.isEmpty()) {
            while (curr != null) {
                // 将其左节点依次放入 因为先读左节点
                stack.push(curr);
                curr = curr.left;
            }
            // 将栈顶弹出
            curr = stack.pop();
            // 塞值
            res.add(curr.val);
            // 将右节点赋值给它 完美呈现了中序遍历 ： 左 -> 根 -> 右
            curr = curr.right;
        }
        return res;
    }
```

代码也不长，该过程可以用大脑无底栈来走一遍。闭上眼睛，冥想。

## 前序遍历
前序遍历（DLR），是二叉树遍历的一种，也叫做先根遍历、先序遍历、前序周游，可记做根左右。前序遍历首先访问根结点然后遍历左子树，最后遍历右子树。

即根结点遍历位置在前面： 根 -> 左 -> 右

题目描述：
```
给定一个二叉树，返回它的 前序 遍历。

 示例:

输入: [1,null,2,3]  
   1
    \
     2
    /
   3 

输出: [1,2,3]
进阶: 递归算法很简单，你可以通过迭代算法完成吗？
```

递归算法不说了，见上。

迭代这里用双向链表 LinkedList，数据量大时存取数据性能好点。

代码如下：
```
    /**
     * 双向队列
     * 
     * @param root
     * @return
     */
    public List<Integer> preorderTraversal1(TreeNode root) {
        LinkedList<TreeNode> stack = new LinkedList<>();
        LinkedList<Integer> output = new LinkedList<>();
        if (root == null) {
            return output;
        }
        stack.add(root);
        while (!stack.isEmpty()) {
            // 弹出队列中最后一个
            TreeNode node = stack.pollLast();
            output.add(node.val);
            if (node.right != null) {
                stack.add(node.right);
            }
            if (node.left != null) {
                // 该节点就是下一个要读的根节点
                stack.add(node.left);
            }
        }
        return output;
    }
```

这里需要注意的就是 `stack.add()` 先添加右结点在添加左结点，因为 `stack.pollLast()` 取出的是最后一个数据，这样就是左结点先弹出，满足前序遍历的顺序。

## 后序遍历
后序遍历（LRD）是二叉树遍历的一种，也叫做后根遍历、后序周游，可记做左右根。后序遍历有递归算法和非递归算法两种。在二叉树中，先左后右再根，即首先遍历左子树，然后遍历右子树，最后访问根结点。

即根结点遍历位置在前面： 左 -> 右 -> 根

题目描述：
```
给定一个二叉树，返回它的 后序 遍历。

示例:

输入: [1,null,2,3]  
   1
    \
     2
    /
   3 

输出: [3,2,1]
进阶: 递归算法很简单，你可以通过迭代算法完成吗？
```

同样，递归算法不重复，见上。

这里的迭代算法我是看了官网的解法。秒。利用双向链表每次在起始位置添加值，顺序为根 -> 右 -> 左，然后遍历完后整个顺序从前往后就是左 -> 右 -> 根，后序遍历。

双向链表 LinkedList + 栈 Stack，代码如下：
```
    public class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }
    }

    /**
     * 根 -> 右 -> 左 存入，遍历完全后从前往后即是 左 -> 右 -> 根
     * 
     * @param root
     * @return
     */
    public List<Integer> postorderTraversal(TreeNode root) {
        LinkedList<Integer> ans = new LinkedList<>();
        Stack<TreeNode> stack = new Stack<>();
        if (root == null)
            return ans;

        stack.push(root);
        while (!stack.isEmpty()) {
            TreeNode cur = stack.pop();
            // 将值塞到最前面
            ans.addFirst(cur.val);
            // 先左结点入栈
            if (cur.left != null) {
                stack.push(cur.left);
            }
            // 再是右结点入栈
            if (cur.right != null) {
                stack.push(cur.right);
            }
        }
        return ans;
    }
```

从后往前推，一气呵成。

---
其实，还是递归最好使……