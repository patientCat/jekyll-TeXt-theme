---
title: 邓俊辉数据结构学习-1-向量 
tags: 数据结构 排序 向量
key: "dsapp-1-vector-20180913"
---

#### github仓库
[patientCat/dsapp_learn](https://github.com/patientCat/dsapp_learn)
<!--more-->
# 向量
根据基本的概率论知识，要求样本随机，且元素需要足够大，来保证其分布。 <br/> 
实际过程中，插值查找往往使用在大规模下，中小规模使用二分查找或者顺序查找 <br/> 

## 简介
向量本身其是就是一个封装了的数组。或者说是抽象的数组。使我们在使用的时候不用在意数组的大小。

## ADT 与成员

事先说明
> Tips 这里主要注意语义的问题。即所有的数据结构接口的语义尽量相同，从而在掌握一个后对其他接口
的操作也很容易掌握 <br/> 
> 因为要适配不同的类型，甚至自定义类型，我们自然采用模板的方式来构造 <br/> 
> 因为在向量中，我们采取了一个新的概念秩，用来表示向量中的下标，其意思代表前面有多少元素。<br/> 

```cpp
using Rank = int; //这里采用C11新方法
//typedef int Rank;
```
思路 -> 成员 -> 构造 -> 接口 -> 析构
### 成员

```cpp
template <typename T>
class Vector 
{ 
private: /*成员*/ 
    Rank    _size;//维护向量实际的元素个数 
    int     _capacity;//维护向量的最大负载
    T *     _elem;//记录向量实际存储的位置
}; 
```

### 构造

向量的构造体现了数组抽象的本质。其中重复利用的一个轮子就是copyFrom函数。<br/> 
```cpp
// 语义很明确，不用过多解释
// 简洁明了，多开辟空间是一个好的习惯，既然使用了Vector，就有很大可能调用insert操作
template <typename T> 
void 
Vector<T>::copyFrom(int *arr, int low, int high)
{
    _elem = new T[2 * (high - low)];
    _size = 0;
    while(low< high)
    {
        _elem[_size++] = arr[low++];
    }
}
```
* 简单的四种构造函数
    * 可以升级的操作
    * 带有移动语义的复制构造函数
    * 初始化列表的构造函数 
    * 其他随自己口味

```cpp
template <typename T>
class Vector{
...
Vector (int capacity = DefaultCapacity); 
Vector (T const *A, Rank low, Rank high){   copyFrom(A, low, high); } 
Vector (T const *A, int n){ copyFrom(A, 0, n);  } 
Vector (const Vector<T> &rhs){  copyFrom(rhs._elem, 0, rhs._size);  } 
...
}

template <typename T>
Vector<T>::Vector(int capacity)
: _size(0)
, _capacity(capacity)
, _elem(new T[capacity])
{}
```
可以看到，利用copyFrom这个轮子，我们很轻松就可以实现构造

## 接口

### 通用接口
一般就是增删改差
* 增
    * 进入增之前，为了保证向量的动态扩容，需要一个新轮子expand()
```cpp
template <typename T>
void 
Vector<T>::expand()
{
    if(_size < _capacity)
       return ;
    _capacity = (_capacity > DefaultCapacity) ? _capacity : DefaultCapacity; //[1]
    _capacity <<= 1;//[3] 为什么使用倍乘的方式扩容，而不是倍加的方式
    T * oldelem = _elem;
    _elem = new T[_capacity];
    for(int idx = 0 ; idx < _size; ++idx)
    {
        _elem[idx] = oldelem[idx]; //[2] 这个for循环可不可以替换为memcpy
    }
    delete []oldelem;
}
```
* \[1\] 用来避免`_capacity = 0`时的情况
* \[2\] 不可以替换为memcpy, 如果是自定义元素memcpy一定是浅拷贝，=可能会被赋予深拷贝的情况
* \[3\] 最开始自己学习的时候也有这样一个疑问，原因是倍乘算下来的分摊时间复杂度为O(1)，而
倍加为O(n), 感兴趣的话可以看邓老师原书。

```cpp
template <typename T>
Rank 
Vector<T>::insert(Rank pos, const T &e)
{
    expand();
    for(int i = _size; i > pos; --i)
    {
        _elem[i] = _elem[i-1];
    }
    _elem[pos] = e;
    ++_size;
    return pos;
}
```
这里主要强调的就是语义问题，因为在C++中，我目前见到过的insert接口都是前插，而且返回新插入元素的位置。<br/> 
保持统一的语义是一个良好的习惯<br/> 
如下，我们再定义push\_back的时候就可以很好的借助insert来完成 <br/> 
```cpp
template <typename T>
Rank push_back(const T &e)  {   return insert(_size, e);   }
```

* 删
    * 我们将先定义范围删除，再定义单个删除
    * 先看代码，在具体解释为什么
    * 忘记说明了，本书如果不加强调的话，所以语义上的区间均为左闭右开，[low, high)

```cpp
class Vector
{
...
    Rank remove(Rank low, Rank high);
    Rank remove(Rank pos)   {   remove(pos, pos + 1); } //[1] 是否可以先定义单个删，然后去定义范围删
...
}
template <typename T>
Rank 
Vector<T>::remove(Rank low, Rank high)
{
    if(high == low) return 0;
    Rank oldsize = _size;
    while(high < _size)
    {
        _elem[low++] = _elem[high++];
    }
    _size = low;
    return oldsize - _size;
}
```
对于删除来说，巧妙的地方在于有时候只要`表现的看起来像删除`就可以了。事实我们并没有去管他们。仅仅是单词的将后面的元素移动到了前面。然后将\_size调整了。

* 查
    * 既然是适用于通用数组的查，自然想都不用想就是遍历
    * 只需要定义合适的语义就可以
    1. 从后向前找，就找第一个
    2. 找到就返回位置
    3. 找不到返回-1

```cpp
// 目标定了，自然是写的越简单越漂亮越好
template <typename T>
Rank 
Vector<T>::find(const T &e, Rank low, Rank high)const
{
    if(low == high) return low - 1;
    while(low < high--)//[1] 这边结束条件
    {
        if(_elem[high] == e)
            break;
    }
    return high;
}
```
* \[1\] 我刚开始学习的时候，对这个结束条件保持疑问。体现了基础功比较差劲。对于后置--来说，
需要先比较，然后再--，所以当high = low的时候判断条件=false，退出循环，还会将high--,恰好返回low-1 <br/> 

* 改
    * 改的方式直接使用运算符重载的机制
```cpp
T & operator[](Rank r)  {   return _elem[r];    }
```

* 遍历
    * 以模板的方式
    * 注册函数的方式
```cpp
// 模板的方式
template <typename T>
template <typename VST>
void 
Vector<T>::Traverse(int low, int high, VST visit)
{
    for(Rank i = low; i < high; ++i)
    {
        visit(_elem[i]);
    }
}

// 以注册函数的方式
template <typename T>
void 
Vector<T>::Traverse2(int low, int high, function<void (T)> visit)
{
    for(Rank i = low; i < high; ++i)
    {
        visit(_elem[i]);
    }
}
```

* 去重的操作
    * 比较简单
* 思路
    利用find和remove接口，

```cpp
template <typename T>
int 
Vector<T>::deduplicate()
{
    int oldsize = _size;
    for(int i = 0; i < _size;)
    {
        if(find(_elem[i], 0, i) > 0)
            remove(i);
        else
            ++i;
    } 
    return oldsize - _size;
}
```
* 可以的优化是，不进行真正的删除，而是先将要删除的元素位置进行标记。最后一次性删除。可以避免很多
不必要的移动

### 有序接口
1. 排序
2. 去重
3. 查找

#### 排序
显然排序是最重要的<br/> 

##### 冒泡排序
版本1: 最为简单的版本
```cpp
void bubble(int *arr, int low, int high);
void bubbleSort(int *arr, int low, int high)
{
    while(low + 1< high)
    {
        bubble(arr, low, high--);
    }
}
void bubble(int *arr, int low, int high)
{
    while(++low < high)
    {
        if(arr[low] < arr[low-1])
            std::swap(arr[low], arr[low-1]); 
    }
}
```
* 知识点
    * while(low < high)  -> [low,low) 区间内部不再含有任何元素
    * while(low + 1 < high)  -> [low, low+1) 区间内只含有一个元素
    * 要快速想出其最终的结束情况, 即退出while后的区间，high的位置
* 如果是其他形式，要转换为上面这种形式。
* 对于排序来说，当最终区间变为1个元素的时候，此时这个区间天然有序，不需要再进行处理，所以可以退出。

版本2
```cpp
bool bubble(int *arr, int low, int high);
void bubbleSort(int *arr, int low, int high)
{
    while(low + 1< high)
    {
        if(bubble(arr, low, high--))
            break;
    }
}
bool bubble(int *arr, int low, int high)
{
    bool sorted = true;
    while(++low < high)
    {
        if(arr[low] < arr[low-1])
        {
            sorted = false;
            std::swap(arr[low], arr[low-1]); 
        }
    }
```
* 优化了在前缀已经有序的情况下，直接退出，不再进行多余比较

版本3
```cpp
int bubble(int *arr, int low, int high);
void bubbleSort(int *arr, int low, int high)
{
    while(low + 1< (high = bubble(arr, low, high--)));
}
int bubble(int *arr, int low, int high)
{
    int last = low;
    while(++low < high)
    {
        if(arr[low] < arr[low-1])
        {
            last  = low;
            std::swap(arr[low], arr[low-1]); 
        }
    }
    return last;
}
```
* 返回最后一个逆序对的位置，仔细理解直接可以让high = last的原因
* 在bubble中，最后一个swap一定是将[low, high)中最大的数推到后面。即推到了last这个位置。此时这个
数将不再参与下次bubble。

##### 选择排序
* 选择排序是冒泡排序经过优化的版本，因为说白了，选择冒泡排序就相当于每次将最大的元素一点一点
冒泡上来，但是要做很多次交换操作。既然如此，我将这个位置进行标记，最后做一次交换就ok
```cpp
template <typename T>
Rank 
Vector<T>::max(int low, int high) // 在[low, high] 中找最大的位置
{
    Rank mx = high;
    while(low < high--)
    {
        if(_elem[high] > _elem[mx]) //[1]
            mx = high;
    }
    return high;
}
template <typename T>
void 
Vector<T>::selectSort(int low, int high)
{
    while(low < --high) //[2]
    {
        swap(_elem[max(low, high)], _elem[high]);
    }
}
```
* \[1\] 这里使用大于是为了保证算法的稳定性，因为使倒着找，所以在遇到相同元素后，找到第一个后，之后
的相同元素将不再导致发生交换
* \[2\] while(low < --high) 的比较方式其实就是将区间从[low, high) 转化为了[low, high-1], 结束条件
依然为high = low 。注意这里不可以改成low + 1 < --high 因为下面max做的比较是[low, high]的区间。
也就是说当 high = low + 1的时候结束while， 但是high = low + 1在max中是俩个数，而不是一个数
* 选择排序的记忆方法是要明白其和冒泡排序一样，每次进行一轮比较完后，规模会减1。所以每次比较完
后high 都会减1, 相当于将数组划分为无序前缀和有序后缀。前缀规模递减的同时会导致后缀规模递增。
* 算法时间复杂度为O(n2), 其主要减少的是swap次数，比较次数没有发生减少。

##### 归并排序
```cpp
template <typename T>
void 
Vector<T>::mergeSort(int low, int high)
{
    if(high - low <= 1)
        return ;
    int mid = (low + high) >> 1;
    mergeSort(_elem, low, mid);
    mergeSort(_elem, mid, high);
    merge(_elem, low, mid, high);
}

template <typename T>
void 
Vector<T>::merge(int low, int mid, int high)
{
    int lenB = mid - low;
    int lenC = high - mid;
    int *A = _elem + low;
    int *B = new int(lenB);
    int *C = _elem + mid;
    for(int i = 0; i<lenB; ++i)
        B[i] = A[i];
    for(int i = 0, j = 0, k = 0; j < lenB || k < lenC; )
    {
        if(j < lenB && (B[j] <= C[k] || lenC <= k)) //[1]
            A[i++] = B[j++];
        if(k < lenC && (C[k] < B[j] || lenB <= j))
            A[i++] = C[k++];
    }
    delete b;
}
```
* \[1\] 可以看到这俩个比较并非对称的，目的是为了保证算法的稳定性，很简单，其实，对于B路中的元素
我们要比C路优先选择，因为B路一定在C路的前面
* 这是最初版本的, 可以做的优化
* 第一种，对于二路归并中，C路其实使我们虚拟出来的，因此如果在B路耗尽的情况下可以直接结束
```
for(int i = 0, j = 0, k = 0; j < lenB; )
{
    if((B[j] <= C[k] || lenC <= k))
        A[i++] = B[j++];
    if(k < lenC && C[k] < B[j])
        A[i++] = C[k++];
}
```
* 第二种优化，new，delete太耗费时间，可以事先开辟一个和A同样大小的数组。但是不太实际，因为
这样的设计就很难看

* 第三种优化, 有时候，俩边的子序列已经有序的情况下，此时不需要合并
```
if(_elem[mid - 1] > _elem[mid])
    merge(_elem, low, mid, high);
```
* 归并排序时间复杂 NlogN
* 是稳定的

对于一般的归并来说，不论是什么情况，都需要经历O(NlogN)的时间，来完成这些。即最好情况和最坏情况
都需要进行这些步骤。经过优化好，在某些最好的情况或者局部最好的情况可以完成优化。变成线性时间

#### 查找
在很多时候，将数组或者向量进行排序之后，然后进行接下来的操作的会大大减少时间，查找就是其中一项 <br/> 

##### 二分查找
当向量中元素为sorted之后，秩将变的比较有意义。<br/> 
假设所有元素互异，则r代表小于s[r]的元素有r个，这是一个很容易忽略的意义。 <br/> 
一般地，若小于、等于S[r]的元素各有i，k个，则该元素以及雷同元素分布在[i, i+ k) <br/> 
即，只要知道小于这个元素有多少个，就可以完全确定这个元素位置(在元素互异情况下) <br/> 
对于元素不互异的情况下，任意元素s[x] ,则[low, x)均<= s[x], 且(x,high)均大于等于s[x] <br/> 

* 版本A
```cpp
int find(int *arr, int value, int low, int high)
{
    while(low < high)
    {
        int mid = (low + high) >> 1;
        //int mid = low + (high - low) >> 1; //为了防止数字溢出的一种保护措施，目前没遇到过
        if(value < arr[mid])
        {
            high = mid;
        }
        else if(arr[mid] < value)
        {
            low = mid+1;
        }
        else
            return mid;
    }
    return -1;
    }
}
```
通常情况下，我们会为数组设置俩个哨兵 <br/> 
* arr[high] = 无穷大
* arr[low-1] = 无穷小

因此对于任意一个value来说，我们始终认为arr[low-1] < value < arr[high]。<br/> 
接下来看if中的比较。如果value < arr[mid]，我们就可以认为这个mid = high, 因此value < arr[mid] ，value
< arr[high] 依然保持了算法的不变性。所以这是一个合理的决策。<br/> 
下一分支, 如果arr[mid] < value ，那么，我们就可以认为mid = low - 1 , low = mid + 1, 依然保持着arr[low-1] < value的不变性。<br/> 
因此，直到最终我们所要的这个value会无限缩小，最终变成一条边界，其左侧的值均小于它，右侧的值均大于它。
代表数组中并不存在我们所要的这个元素，我们可以返回low(最开始的 - 1 或者 -1 来代表我们没有找到他。<br/> 
相反，在区间不断缩小的过程中，只要数组中含有=value的这个元素，我们就一定可以找到它。因为凡是大于它和小于它的均会被我们抛弃。 <br/> 

* 缺点
    * 比较分支不平衡。可以通过适当增加前面的子序列，减少后面的子序列完成平衡
    * 可以的方法有版本B或者Fibonacci查找


```cpp
struct Fib
{
    Fib(int n); //构造fib序列，直到fib(n) > n， 即如果n = 10 ，则构造到 1 1 2 3 5 8 13 这里

    int get(); // 返回当前fib，此例中为13
    void pre(); // 返回上一fib, 此例中为8
    void next(); // 返回下一fib, 此例中为21
}

int fibSearch(int *arr, int value, int low, int high)
{
    Fib fib(high - low);
    while(low < high)
    {
        while(high - low < fib.get()) fib.pre(); // 找到小于等于high - low的一个fib值
        int mid = low + fib.get() - 1; // 使用黄金分割点
        if(value < arr[mid])
            high = mid;
        else if(arr[mid] < value)
            low = mid + 1;
        else
            return mid;
    }
    return -1;
}

```
* 平衡了两边的比较。但是失败时不能指示失败的位置。

* 版本B
```cpp
int binSearch(int *arr, int value, int low, int high)
{
    while(1 < high - low)
    {
        int mid = (high + low) >> 1;
        value < arr[mid] ? high = mid : low = mid;
    }
    return arr[low] == value ? low : -1;
}
```
如果对二分掌握的比较好的话，基本可以看出什么意思。 <br/> 
1. while条件，代表结束时，区间包含一个元素。
2. value的判断条件，arr[low] <= value < arr[high] 始终保持这个条件，
3. 最终条件为包含一个元素，如果数组中存在value，则value一定在这个区间内，且就是low这个位置。否则就是不存在

* 同上面fibSearch近乎等价的版本，缺点一样

* 版本C
```cpp
int binSearch(int *arr, int value, int low, int high)
{
    while( low < high)
    {
        int mid = (high + low)>>1;
        value < arr[mid] ? high = mid : low = mid + 1;
    }
    return --low;
}
```
快速判断
* 根据while条件，得到最终结束时是一个边界，不包含任何元素。最终会归于这么一个区间[low, low)
* value所在的区间为[low, high)， 如果value < arr[mid] 我们令high = mid 则新的区间依然保持[high，size)这个区间的内值均大于原来的区间。
* value >= arr[mid] ，我们将新区间收缩为low = mid + 1, high 依然保持任意值在[0, low) 均不大于原区间。
* 注意，我们每次进行排除，都会讲mid这个点也进行排除。
* 假设我们不排除mid这个点，令low = mid ; 此时我们将无法保证任意值在[0,low)上是小于还是小于等于原区间内的值。因为如果令low = mid； 对于arr[mid-1]这点是小于原区间，还是小于等于原区间我们无法确定，因为没有判断。
* 而对于mid这点，因为我们使用了arr[mid] <= value这个判断，所以可以确定

* 同样，根据语义如果发生转换，我们也可以找到不小value的位置。
```cpp
int binSearch(int *arr, int value, int low, int high)
{
    while(low < high)
    {
        int mid = (high + low) >> 1;
        value > arr[mid] ? low = mid + 1 : high = mid;
    }
   //return ++ (high - 1); 类推上面的等价写法
    return high;
}
* 换个思路就可以。
* 只要保证最终的区间左侧均< 边界，右侧均大于边界即可。
* 对于左边界，很简单
* 对于右边界， value <= arr[mid] , high = mid 我们发现，
对于临界边界即开区间很好判断，我们直接使用high, low - 1 俩个哨兵相对于arr[mid]进行判断即可。
* 如果value > arr[mid] 令low - 1 =mid ，因为开区间是取不到的。
* value < arr[mid] 令 high = mid 依然保持原区间value < arr[high]
* arr[mid] < value 令 low -1 = mid  依然保持 value > arr[low -1]
* 当high = low的时候结束循环，此时区间为(low -1, low)依然是保持0元素的
* 啰嗦了半天，其实还是有点没有通达的感觉，但是比起最开始的只是记忆，二分基本被我搞的差不多懂了。
递归或者说迭代的本质上还是保持算法的不变性。
```

#### 其他查找
其他查找比如插值查找，其实就是在mid中点上做了一些手脚
```cpp
int mid = low + (high - low) * (value - _elem[low]) / (_elem[high] - _elem[low]);
```
根据基本的概率论知识，要求样本随机，且元素需要足够大，来保证其分布。 <br/> 
实际过程中，插值查找往往使用在大规模下，中小规模使用二分查找或者顺序查找 <br/> 

### 唯一化
```cpp
template <typename T>
int 
Vector<T>::uniquify()
{
    int oldsize = _size;
    int i = 0, j = 0;
    while(++j < _size)
    {
        if(_elem[j] != _elem[i])
            _elem[++i] = _elem[j];
    }
    _size = i + 1;
    return oldsize - _size;
}
```
* 特别特别喜欢这个唯一化，用看起来像是`删除`的方式完成删除。不需要多余的元素移动。时间复杂度O(n)
