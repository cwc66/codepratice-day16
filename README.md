# codepratice-day16
代码随想录第16次练习

二叉树中，遍历数值的问题。
递归、回溯

[二叉搜索树的最小绝对差](https://leetcode.cn/problems/minimum-absolute-difference-in-bst/description/)
给你一个二叉搜索树的根节点 root ，返回 树中任意两不同节点值之间的最小差值 。
差值是一个正数，其数值等于两值之差的绝对值。


二叉搜索树是有序的。
二叉搜索树采用中序遍历，就是一个有序数组。再求最小差值。
这段代码没有在遍历中直接计算答案
```CPP
class Solution {
private:
vector<int> vec;
void traversal(TreeNode* root) {
    if (root == NULL) return;
    traversal(root->left);
    vec.push_back(root->val); // 将二叉搜索树转换为有序数组
    traversal(root->right);
}
public:
    int getMinimumDifference(TreeNode* root) {
        vec.clear();
        traversal(root);
        if (vec.size() < 2) return 0;
        int result = INT_MAX;
        for (int i = 1; i < vec.size(); i++) { // 统计有序数组的最小差值
            result = min(result, vec[i] - vec[i-1]);
        }
        return result;
    }
};
```

可以在二叉搜索树中序遍历的过程中，直接计算答案。需要一个Pre节点记录一下cur节点的前一个节点。
```CPP
class Solution {
private:
    int result=INT_MAX;
    TreeNode* pre=nullptr;
    void traversal(TreeNode* cur)
    {
        if(cur==nullptr) return;
        traversal(cur->left);//左
        if(pre!=nullptr)    //中
        {
            result=min(result,cur->val - pre->val);
        }
        pre=cur;//记录前一个节点
        traversal(cur->right);//右
    }
public:
    int getMinimumDifference(TreeNode* root) {
        traversal(root);
        return result;
    }
};
```

迭代法

```CPP
class Solution {
public:
    int getMinimumDifference(TreeNode* root) {
        stack<TreeNode*> st;
        TreeNode* cur=root;
        TreeNode* pre=nullptr;
        int result=INT_MAX;
        while(cur!=nullptr || !st.empty())
        {
            if(cur!=nullptr)
            {//指针来访问节点，访问到最底层
                st.push(cur);//将访问的节点放进栈
                cur=cur->left;//左
            }
            else
            {
                cur=st.top();
                st.pop();
                if(pre!=nullptr)
                {
                    result=min(result,cur->val - pre->val);
                }
                pre=cur;
                cur=cur->right;
            }
        }
        return result;
    }
};
```
遇到在二叉搜索树上求什么最值，求差值之类的，都要思考一下二叉搜索树可是有序的，要利用好这一特点。


[二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)
给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

说明:
所有节点的值都是唯一的。
p、q 为不同节点且均存在于给定的二叉树中



二叉树如何自底向上查找。
回溯

后序遍历（左右中）是天然的回溯过程，可以根据左右子树的返回值，来处理中节点的逻辑。

如何判断一个节点是q和p的公共祖先？
1.如果找到一个节点，发现左子树出现结点p，右子树出现节点q，或者 左子树出现结点q，右子树出现节点p，那么该节点就是节点p和q的最近公共祖先。
2.节点本身(p或q)，它拥有一个子孙节点(q或p).

递归三部曲
1.确定递归函数返回值以及参数
需要返回最近公共节点，如果找到了p或者q，就返回，返回值不为空说明找到了。

2.确定终止条件
遇到空，树为空了，返回空

如果root==p或root==q，说明找到q,p，则将其返回。

3.确定单层递归逻辑
在递归函数有返回值的情况下：如果要搜索一条边，递归函数返回值不为空的时候，立刻返回，如果搜索整个树，直接用一个变量left、right接住返回值，这个left、right后序还有逻辑处理的需要，也就是后序遍历中处理中间节点的逻辑（也是回溯）。

如果left 和 right都不为空，说明此时root就是最近公共节点。这个比较好理解

如果left为空，right不为空，就返回right，说明目标节点是通过right返回的，反之依然。

```CPP
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == q || root == p || root == nullptr) return root;
        TreeNode* left=lowestCommonAncestor(root->left,p,q);
        TreeNode* right=lowestCommonAncestor(root->right,p,q);
        if(left != nullptr && right != nullptr) return root;

        if(left == nullptr && right != nullptr) return right;
        else if(left != nullptr && right == nullptr) return left;
        else
        {
            return nullptr;
        }
    }
};

```
精简
```CPP
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == q || root == p || root == nullptr) return root;
        TreeNode* left=lowestCommonAncestor(root->left,p,q);
        TreeNode* right=lowestCommonAncestor(root->right,p,q);

        if(left != nullptr && right != nullptr) return root;
        if(left == nullptr) return right;
        return left;
    }
};
```

回溯的过程，以及结果如何一层一层传上去的。
3点
1.求最小公共祖先，需要从底向上遍历，那么二叉树，只能通过后序遍历（即：回溯）实现从底向上的遍历方式。

2.在回溯的过程中，必然要遍历整棵二叉树，即使已经找到结果了，依然要把其他节点遍历完，因为要使用递归函数的返回值（也就是代码中的left和right）做逻辑判断。

3.要理解如果返回值left为空，right不为空为什么要返回right，为什么可以用返回right传给上一层结果。


