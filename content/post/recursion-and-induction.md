---
title: "借助数学归纳法理解递归"
date: 2018-09-04T07:52:19+08:00
draft: false
---

递归程序通常需要考虑两点：

1. 递归步，也就是如何使原问题收敛。
2. 基本条件（/终止条件），即返回值的斟酌；

以典型的二叉树遍历为例，前序遍历的递归代码如下：

```java
public List<Integer> preorderTraverse(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    
    if (root != null) {
        result.add(root.val);
        result.addAll(preorderTraverse(root.left));
        result.addAll(preorderTraverse(root.right));
    }

    return result;
}
```

即使没有受过数据结构与算法课程的洗礼，应该也容易理解这段代码：先添加根节点到遍历结果、再依次添加左、右子树的遍历结果。

但是正因为看上去太简单，我有时候反而会疑惑，它究竟怎样完成了每个节点的遍历。

这大概也是递归程序不好写和不好理解的地方。非递归的算法，通常只要把算法所表达的步骤“翻译”成代码，稍加调试，就可以正确运行了。可是这一点并不适用于递归算法，因为频繁的进栈出栈让它的中间步骤变得难以把握。以上述代码为例，`call(root.left)`, `call(root.right)` 果真能给出左右子树的前序遍历结果吗？

其实，可以借助数学归纳法来回答这个问题。数学归纳法告诉我们，当一个命题对于 n = 1 成立，并且由 n = k 时命题成立的假设，能够推导出 n = k + 1 时也成立，那么这个命题为真。

这样去看刚才的代码——当一棵树只有一个节点时，前序遍历就是它自己，代码的返回值显然正确。接着，假设它对于 root 的左右子树能给出正确结果，那么对于 root 的返回值自然就是正确的。
