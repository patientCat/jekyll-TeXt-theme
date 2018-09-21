---
title: 邓俊辉数据结构学习-5-树
tags: 数据结构 二叉树 
key: "dsapp-5-tree-20180914"
---

# 树
## 树的一些结论
> 度: 一个节点的孩子数称为度。 <br/> 
> 一颗树的边数 = 节点数n - 1  意义在于衡量算法复杂度时使用。<br/> 

<!--more-->

## 树的特性
> 路径(通路) + 环路  <br/> 
> 通路： a, b 之间通过节点连接成的路。<br/> 
>     * 长度：所有边的数目（有些文献使用节点定义长度）
> 
> 环路：通路的俩个节点彼此短路，即重合构成环路。eg. a - b - c - d - b  <br/> 

> 连通图：节点之间均有路径 <br/> 
> 无环图：没有环路 <br/>  

### 重要特性
1. 所谓的树就是无环连同图。极小连通图，极大无环图。 <br/> 
2. 任一节点到根的路径唯一。因此path(v,r) = path(r);  (v 就是vertex的意思), 因此我们可以通过
3. 路劲的长度来衡量书中的一个节点。 <br/> 
4. 等价类：长度相同的节点，我们称之为等价类。<br/> 

### 简化
路径，节点，子树均可以相互指代，原因是节点到根路径唯一。所以在后面树的遍历中，要明白访问节点和
访问子树的概念是不同的，虽然都是使用同一个节点进行代表<br/> 
深度: 一个节点的深度就是节点的长度。就是节点到根的路径的长度 <br/> 
半线性: 前驱唯一性得以满足，但后继唯一性不满足。<br/> 
叶子: 没有后代的节点称为叶子。<br/> 
树的高度: 一个深度最大的叶子节点的深度称为这颗树的高度。 这一概念可以推给子树的高度。 <br/> 
> 根节点的高度就是整树的高度。 某个子节点的高度是这个节点作为子树根节点的高度。 <br/> 
> 具有一个节点的树的高度为0。 没有节点的树，也就是空树的高度为-1。<br/> 
 
注意区分一个节点的高度和深度。 <br/> 
节点的深度是节点到根的路径。节点的高度是节点所代表的子树总，具有最大深度最大的某个节点深度 <br/> 

### 树为什么要这样表示
1. 父亲表示法： 向下查询孩子节点的时候，需要遍历整个表，看谁的父亲是这个节点，才能找到孩子节点。
2. 孩子表示法： 向上查询父亲的时候，同样需要遍历整个表，看谁的孩子是这个节点，才能找到父亲节点。
3. 长子+兄弟法：最能体现树的本质。

## 二叉树
### 重要概念
1. 真二叉树
    * 所有节点的出度均为偶数。将一颗二叉树转化为真二叉树，有模糊概念，需要到具体应用时才能理解。
2. 如何使用二叉树来描述多叉树
    * 没有错，真的可以。

### 遍历
#### 先序遍历
##### 递归版本
```cpp
void preTraverseRecur( BinNode * x, VST visit)
{
    if(!x)
        return ;
    visit(x->data);
    preTraverseRecur(x->lchild, visit);
    preTraverseRecur(x->rchild, visit);
}

```
有一个理解错误的概念，就是先序遍历，我们做的是先访问根节点，然后访问左`子树`，然后访问右`子树`。
注意这里标记出来的点，我之前理解的概念为访问左孩子这个节点。有什么不一样吗？因为一个节点可以表示
一个节点，同时也可以表示一颗树。（将其视为这颗树的根）。所以我们访问节点，就是访问节点，访问子树
则意味着访问子树内的所有节点。 <br/>
所以先序遍历的理解是：先访问root节点，然后访问左子树，访问右子树。 <br/>


##### 迭代版本
版本A <br/>
这个代码感觉自己已经背会了，不知道怎么写思路了。
```cpp
void preTraverseIter(BinNode * x, VST visit)
{
    stack<BinNode *> s;
    s.push(x); //根节点入栈
    while(s.empty())
    {
        auto ret = s.top(); s.pop();
        visit(ret->data);
        if(x->rchild)  //注意先将右子树入栈
            s.push(x->rchild);
        if(x->lchild)
            s.push(x->lchild);
    }
}
```
要点 <br/> 
1. 需要开始，因此需要将根节点压入栈，然后右子树进栈，然后左子树进栈，然后新一轮判断。
2. 必须先右再左，因为栈是filo。
3. 这个感觉其实很奇怪，就是利用栈来模拟整个递归过程。

版本B <br/>
```cpp
void preTraverseIter(BinNode * x, VST visit)
{
    stack<BinNode *>tmp;
    while(true)
    {
        visitAlongRC(x, visit, tmp); //有左树，走左树，同时将每个经历的右树入栈
        if(tmp.empty())
            return ;
        x = tmp.top(); //走向最近的右树
        tmp.pop();
    }
}

void visitAlongRC(BinNode *x, VST visit, stack<BinNode *> &tmp)
{
    while(x)
    {
        visit(x->data);
        tmp.push(x->rc);
        x = x->lc;
    }
}
```
慢慢有点感觉了，就是有左走左，没左就走一步右，然后继续循环。有点绕大圈的感觉是不是。

#### 中序遍历
##### 递归版本
```cpp
void inTraverseRecur( BinNode * x, VST visit)
{
    if(!x)
        return ;
    preTraverseRecur(x->lchild, visit);
    visit(x->data);
    preTraverseRecur(x->rchild, visit);
}

```

##### 迭代版本
```cpp
void inTraverseIter(BinNode * x, VST visit)
{
    stack<BinNode *> S;
    while(true)
    {
        {//goAlongLc
        while(x)
        {
            S.push(x);
            x = x->lc;
        }
        }
        if(S.empty())
            break;
        x = S.top();
        S.pop();
        visit(x);
        x = x->rc;
    }
}
```
还是无法具体描述这种感觉，需要仔细和递归版本比较。递归版本来看。就是有左边就一直走左边。
走到左边不能走为止，然后回朔，访问最近的左节点（也就是不能左边空节点的根），然后走向这个节点
的右子树。


#### 后序遍历
##### 递归版本
```cpp
void postTraverseRecur( BinNode * x, VST visit)
{
    if(!x)
        return ;
    preTraverseRecur(x->lchild, visit);
    preTraverseRecur(x->rchild, visit);
    visit(x->data);
}

```

##### 迭代版本
最喜欢的就是迭代版本的后序遍历，因为真的很巧妙。它恰好是先根，再右子树，再左树的倒序输出。
明显就是使用俩个栈就可以实现了。
```cpp
void postTraverseIter(BinNode * x, VST visit)
{
    stack<BinNode * > helper;
    stack<BinNode * > output;
    helper.push(x);
    while(!helper.empty())
    {
        x = helper.top();
        helper.pop();
        output.push(x);
        if(HasLc(x))
            helper.push(x->lc);
        if(HasRc(c))
            helper.push(x->rc);
    }
    while(!output.empty())
    {
        visit(output.top());
        output.pop();
    }
}
```
注意这里是先压左树再压右树，和先序遍历的方式刚好相反。

#### 层次遍历
##### 层次遍历，自上而下
```cpp
void levelTraverse(BinNode * x, VST visit)
{
    queue<BinNode *> Q;
    Q.push(x);
    while(!Q.empty())
    {
        x = Q.front();
        Q.pop();
        visit(x);
        if(HasLc(x))
            Q.push(x->lc);
        if(HasRc(x))
            Q.push(x->rc);
    }
}
```

##### 层次遍历，自下而上
哇！这个实现也非常巧妙
```cpp
void levelReverseTraverse(BinNode * x, VST visit)
{
    queue<BinNode *> Q;
    stack<BinNode *> S;
    Q.push(x);
    while(!Q.empty())
    {
        x = Q.front();
        Q.pop();
        S.push(x);
        if(HasRc(x))
            Q.push(x->rc);
        if(HasLc(x))
            Q.push(x->lc);
    }
    while(!S.empty())
    {
        visit(S.top());
        S.pop();
    }
}
```
#### 根据前序或者后序配合中序还原树
这个代码自己花了3个小时才搞出来。<br/>

```cpp
BinNode * rebuild(vector<int> preorder, vector<int> inorder)
{
    return buildHelper(preorder.begin(), preorder.end(), inorder.begin(), inorder.end());
}
using iter = vector<int>::iterator;
BinNode * buildHelper(iter p1, iter p2, iter i1, iter i2)
{
    if(p1 >= p2 || i1 >= i2)
        return nullptr;
    BinNode * root = new BinNode();
    root->val = *p1;
    auto ret = find(i1, i2, root->val);
    int len = ret - i1;
    p1++;
    root->lc = buildHelper(p1, p1 + len, i1, ret);
    root->rc = buildHelper(p1+len, p2, ret + 1, i2); 
    return root;
}
```
时刻要记得自己是`[low, high)` 的方式，还是`[low, high]`的方式在操纵。大部分情况下是`[low, high)`的方式
因为在c++中的迭代器就是给你这样的方式。 <br/> 

这里可以优化的一个点就是find这里。在leetcode上看到的做法是直接使用散列表。因为preorder和inorder
中的那个数都是相同的，所以讲preorder中的数字映射到inorder中的位置。这样以`O(1)`的时间得到 <br/> 

同样，使用后序遍历结合中序遍历的情况下也能建树
```cpp
BinNode * rebuild(vector<int> postorder, vector<int> inorder)
{
    return buildHelper(postorder.begin(), postorder.end(), inorder.begin(), inorder.end());
}
using iter = vector<int>::iterator;
BinNode * buildHelper(iter p1, iter p2, iter i1, iter i2)
{
    if(p1 >= p2 || i1 >= i2)
        return nullptr;
    BinNode * root = new BinNode();
    root->val = *(p2 - 1);
    auto ret = find(i1, i2, root->val);
    int len = ret - i1;
    root->lc = buildHelper(p1, p1 + len, i1, ret);
    root->rc = buildHelper(p1+len, p2 - 1, ret + 1, i2); 
    return root;
}
```

### 插入和删除
#### 插入节点
这个其实就是基本功夫的考验。这里主要注意的是我们这次有parent节点。

#### succ，计算后继节点
如何计算后继节点，首先必须要深深理解中序遍历。所谓后继节点，就是当前节点中序遍历的下一个节点。
<br/>

```cpp
BinNode * succ()
{
    BinNode * s = this;
    if(this->rc) // 如果有右树，就一定在右树之中
    {
        s = rc;
        while(s) s = s->lc; 
    }
    else{ // 如果该节点没有右树
        while(s == s->parent->rc)  s = s->parent; // 那么该节点一定是最右边的孩子。因此首先需要找到
                                                  // 其右树开始的分叉
        s = s->parent;                            // 此时的s一定是在上一节点的左树之中，它的parent
                                                  // 就是后继 
    } 
}
```
不是特别特别的明白。但是这里其实也不是那么重要。
