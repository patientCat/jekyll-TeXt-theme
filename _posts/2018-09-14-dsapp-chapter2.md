---
title: 邓俊辉数据结构学习-2-列表 
key: 20180914
tags: 数据结构 排序 列表
key: "dsapp-2-list-20180914"
---

#### github仓库
[patientCat/dsapp_learn](https://github.com/patientCat/dsapp_learn)
<!--more-->

# 列表
stl中的列表实际上是双向列表

## 列表节点ADT

| 成员 | 功能 |
| --- | --- |
| data | 数据域 |
| pred | 直接前驱 |
| succ | 直接后继 |

| 接口 | 功能 |
| --- | --- | 
| insertAsPred | 作为直接前驱插入 |
| insertAsSucc | 作为直接后继插入 |

* 将插入的接口写在了列表中。没有想到过的设计。
* 复习下双向列表的插入。

```cpp
template <typename T>
Pos(T) 
LinkNode<T>::insertAsPred(const T &e)
{
    Pos(T) pnode = new LinkNode<T>(e, _pred, this); 
    _pred->succ = pnode;
    _pred = pnode; 
    return pnode;
}
```
其实就是俩个点，第一，新插入节点的前缀指向原来的前缀，原来前缀的后缀指向新插入的点。
第二，新插入节点的后缀指向原来节点，原来节点的前缀指向新节点

## 列表ADT
### 成员

| 成员 | 功能 |
| --- | --- |
| \_size | 维护列表节点数目 |
| \_header | 头哨兵 |
| \_trailer | 尾哨兵 |

* 哨兵的作用在列表中可以帮我们判断很多事情。在做很多事情的时候都很方便。
* 但是对于列表的使用者来说，哨兵是隐藏的，是在构造列表对象的时候就会构造的。
* 即使这里构造的不是双向列表，而是单向列表，构造哨兵也是非常有意义的。

### 接口
#### 接口一览

| 操作接口 | 功能 | 适用对象 | 
| --- | --- | --- |
| size() | 返回节点数 | 列表 |
| first() | 返回首节点位置 | 列表 |
| last() | 返回末节点位置 | 列表 |
| insertAsFirst(e) | 插入首节点 | 列表 |
| insertAsLast(e) | 插入末节点 | 列表 |
| insertBefore(e) | 插入直接前驱 | 列表 |
| insertAftere(e) | 插入直接后继 | 列表 |
| 注|insert操作均返回新节点位置 |
| remove(p) | 删除节点p处的位置, 返回其数值 | 列表 |
| isdisordered() | 判断列表是否有序 | 列表 |
| sort() | 排序 | 列表 |
| find(e) | 查找, 失败返回NULL | 列表 |
| search(e) | 查找, 失败返回不大于e的节点 | 有序列表 |
| deduplicate() | 去重 | 列表 |
| uniquify() | 去重 | 有序列表 |
| traverse() | 遍历 | 列表 |

### 通用接口
#### 构造

因为头尾哨兵的存在，所以在构造的时候需要初始化哨兵, 且串成双向链表<br/> 
```cpp
_header = new LinkNode<T>();
_trailer = new LinkNode<T>();
_header->pred = nullptr; _header->succ = _trailer;
_trailer->pred = _header; _trailer->succ = nullptr;
```

#### insert
概念：
头尾节点：指的是哨兵节点。哨兵的存在大大简化了算法的实现<br/> 
首末节点：指的是列表可见节点。<br/> 
代码重用的思想在列表中得到非常大的体现。这里以列表的前插为例。<br/> 

```cpp
template <typename T>
Pos(T) 
List<T>::insertBefore(Pos(T) p, const T &e)
{
    return p->insertAsPred(e); 
}

template <typename T>
Pos(T) 
List<T>::insertAsLast(const T &e)
{
    insertBefore(_trailer, e); //[1]
}
```
* 在一开始的时候使用的是first和last，即`insertAfter(last(), e);`, 这样错误的问题在于first和last并不
能保证一直存在，比如最开始空列表的时候。
* 即使在一开始在首节点，即`insertBefore(first(), e)` 因为存在哨兵节点，不需要做特殊处理。

--- 

* 复制构造
    * 在有了插入之后，我们就可以构造一个类似向量中copyfrom的轮子
    * copyNodes();
    * 实现很简单
```cpp
template <typename T>
void 
List<T>::copyNodes(Pos(T) p, int n)
{
    while(n--)
    {
        insertAsLast(p->data);
        p->succ;  
    }
}

...

List(const List &l)
{
    copyNodes(l.first(), l.size());
}
```
#### find 
对于列表来说，在无序情况下和有序情况下查找情况是差不多的，原因是如果我们希望访问一个节点。只能通过它
的前缀的前缀的前缀...的前缀依次访问pos才可以得到（或者是后缀），所以当前节点仅仅知道它的前一个节点和
后一个节点。 <br/> 

#### remove
* 注意语义，以前自己在构造列表删除的时候往往删除完就没有了，其实返回删除位置的数据域是一个很好的选择
* 根据删除，可以用来定义析构

`~List ()    {   while(_size) remove(_header->succ); delete _tailer; delete _header; }  `

#### 唯一化
* 唯一化的时候从前向后遍历，从后向前遍历都是没有错的。这里讲下书中的思路。

```cpp
template <typename T>
int 
List<T>::deduplicate()
{
    int oldsize = _size;
    Pos(T) scan = _header, q;
    int r = 0;
    while((scan = scan->succ)  != _trailer)
    {
        if((q = find(scan->data, r, scan)))
        {
            remove(q);   
        }
        else{
            ++r;
        }
    } 
    return oldsize - _size;
}
```
书中的思路是一直保持scan的前缀子序列是有序的，如果遇到雷同，remove子序列中雷同节点，然后继续向下遍历。


#### 遍历
还是向量那点东西

### 有序接口
#### 查找

同无序查找的区别不同主要在于有序查找的语义不同，其返回的是不大于目标元素的的最后者 <br/> 
```cpp
template <typename T>
Pos(T) 
List<T>::search(const T &e, int n, Pos(T) p) const
{
    assert(0 <= n < _size); 
    while(n--)
    {
        if((p = p->pred) <= e) // 这里的判断不同
            return p;
    } 
    return p->pred;
}
```

#### 唯一化
在已经排序的情况下，进行唯一化仍然先尝试延用在向量使用的时候的思路
```cpp
template <typename T>
int
List<T>::uniquify()
{
    if(_size < 2)
        return 0;
    int oldsize = _size;
    Pos(T) curr = _header->succ; 
    Pos(T) succ = nullptr;
    while((succ = curr->succ) != _trailer)
    {
        if(succ->data == curr->data)
        {
            remove(succ);
        }
        else{
            curr = curr->succ; 
        }
    } 
    return oldsize - _size;
} 
```
* 与无序去重区别就在于，经过排序之后，列表中的相同元素会排列在一起，所以只需要判断相邻元素是否
相同即可。如果相同就删除后者，让succ指针继续指向当前指针的后继。如果不同，调整当前指针。


#### 排序
以下一次实现 列表版本的几种排序
* 冒泡排序
* 插入排序
* 选择排序
* 归并排序

##### 冒泡排序

原来bubble接口语义不用改变, 但是bubbleSort接口需要改变。因为此时来说，如果和之前向量相同语义，
我们就需要让trailer对外可见，但这不是我们想要的。因此使用如下接口
```cpp

class List{
...
    void bubbleSort(Pos(T) p, int n) // 采取位置配合size的方式。
    {
        Pos(T) high = p;
        while(n--)
            high = high->succ; 
        while(p != (high = bubble(p, high)));
    }
...
}

template <typename T>
Pos(T) 
List<T>::bubble(Pos(T) low, Pos(T) high)
{
    Pos(T) last = low;
    while((low = low->succ)  != high)
    {
        if(low->data < low->pred->data)
        {
            last = low;
            std::swap(low->data, low->pred->data);    
        }
    }
    return last;
}
```

##### 插入排序
新的排序， O(n2); 三大基础排序，冒泡排序，选择排序，插入排序。<br/> 
插入排序的思想是将数组分为有序前缀和无序后缀。我更喜欢叫它为插牌排序，就是模拟抓牌的一个过程。
每抓到一张牌的时候，总是要在手里的牌检索，自小到大，插入到第一个大于新牌的前面，否则插到最后。<br/> 

数组做法
```cpp
void insertSort(int *arr, int low, int high)
{
    int len = high - low;
    int i, j, k;
    for(k = 1; k < len; ++k)// k 维护无序队列，因为1个数字天然有序，所以从1开始
    {
        for(j = 0; j < k; ++j) // k 同时也是有序队列的上界，使用j在有序队列中遍历。
                               //寻找位置,必须从前向后寻找
        {
            if(arr[k] < arr[j]) //[1]
                break;
        }
        int tmp = arr[k]; // 接下来就是数组的插入过程
        for(i = k; i > j; --i)
        {
            arr[i] = arr[i-1];
        }
        arr[j] = tmp; 
    }
}
```
* \[1\] 注意为了保证稳定性，我们采取的比较方式

一个可以的优化
```cpp
int binSearch(int *arr, int value, int low, int high) // 采用binSearch的方式平衡最好最坏情况
{
    while(low < high)
    {
        int mid = (low + high) >> 1;
        value <= arr[mid] ? high = mid  : low = mid + 1;
    }
    return low;
}
void insertSort(int *arr, int low, int high)
{
    int len = high - low;
    int i, j, k;
    for(k = 1; k < len; ++k)
    {
        j = binSearch(arr, arr[k], 0, k);
        int tmp = arr[k];
        for(i = k; i > j; --i)
        {
            arr[i] = arr[i-1];
        }
        arr[j] = tmp; 
    }
}
```

列表版本
```cpp
void insertSort(Pos(T) p, int n)
{
    Pos(T) scan = p;
    Pos(T) high = p;
    Pos(T) ret;
    while(n--)
        high = high->succ;  
    int r = 0;
    while(scan != high)
    {
        auto pos = search(scan->data, r++, scan); //1
        insertAfter(pos, scan->data); 
        scan = scan->succ;//2
        remove(scan->pred); 
    }
}
```
* 1 列表版本可以通过search调用快速找到在有序前缀中需要插入的位置。
* 2 所示的remove方法，先走再删除是合理的，因为存在trailer，而且是比较好的方法，
如果是先删除再走的话，就需要一个新的指针来记录位置, 我的源代码中有这个实现。
 
##### 选择排序

```cpp
template <typename T>
void 
List<T>::selectSort(Pos(T) p, int n)
{
    Pos(T) high = p;
    while(n--)
        high = high->succ; 
    while(p != high)
    {
        std::swap(selectMax(p, high)->data , high->pred->data );
        high = high->pred; 
    }
}

template <typename T>
Pos(T) 
List<T>::selectMax(Pos(T) low, Pos(T) high)
{
    Pos(T) mx = low;
    while((low = low->succ) != high)
    {
        if(low->data >= mx->data) // [1]
          mx = low;  
    } 
    return mx;
}
```

* \[1\] 为了保持稳定性，采用了这样的比较方式

##### 归并排序

```cpp
template <typename T>
void 
List<T>::mergeSort(Pos(T) p, int n)
{
    if( n < 2 ) return ;
    Pos(T) p2 = p;
    int mid = n >> 1;
    int a = mid;
    while(a--) p2 = p2->succ; 
    mergeSort(p, mid);
    mergeSort(p2, n - mid);
    if(p2->pred->data > p2->data)
      merge(p, mid, n);  
}

template <typename T>
void
List<T>::merge(Pos(T) p, int mid, int n)
{
    int lenB = mid;
    int lenC = n - mid;
    Pos(T) A = p;
    T * B = new T[mid];
    for(int i = 0; i<mid; ++i)
    {
        B[i]= p->data;
        p = p->succ;  
    }
    Pos(T) C = p;
    for(int j = 0, k = 0; j < lenB;)
    {
        if( lenC <= k || B[j] <= C->data )
        {
            A->data = B[j++];  
            A = A->succ;
        }
        if( k < lenC && C->data < B[j])
        {
            A->data = C->data;  
            A = A->succ;
            C = C->succ; 
            ++k;
        }  
    }
    delete []B;
} 
```
* 手撸出来这个代码我觉得我已经对归并排序掌握的很深了。虽然刚开始的时候在递归中忘记添加递归基了。


#### 逆置列表

有俩种操作
1. 第一种修改前驱，后继指针。一次遍历完成。
2. 第一种修改前驱，后继指针。俩次遍历完成。
3. 交换指针的前驱后继
4. 交换数据域

代码不再提供，告诉大家思路。源码在我的github里。
