---
title: 邓俊辉数据结构学习-4-队列
tags: 队列
key: "dsapp-4-queue-20180918"
---

# 队列的学习
## 队列的模拟
使用数组对循环队列进行模拟的时候，必须放弃一个位置。<br/> 
<!--more-->

> 关于模板类继承之后派生类使用基类接口，需要使用基类的作用域限定。 <br/> 
> 队列的底层，如果不考虑空间的话，使用vector比较好，否则使用双向链表，主要delete太耗时间。<br/> 
## 队列的应用
### RoundRobin轮盘机制
RoundRobin <br/> 
对某项资源进行轮流分配 <br/> 
```cpp
void RoundRobin()
{
    Queue<int> que;
    while(!serve.close())
    {
        e = que.dequeue();
        serve(e);
        que.enqueue(e);
    }
}
```
### 归并排序
其实在做二路归并的时候，我们就使用了队列的思想。在做B和C比较的时候，每次使用的都是头部进行比较。
其中就暗含队列的思想。这次我们直接使用队列的接口，可以更加清晰明了的看出归并排序的思想
```cpp
void merge(int *arr, int low, int mid, int high)
{
    queue<int> b;
    queue<int> c;
    queue<int> a;
    for(int i = low; i < mid; ++i)
    {
        b.push(arr[i]);
    }
    for(int i = mid; i < high; ++i)
    {
        c.push(arr[i]);
    }
    size_t len = high - low;
    while(a.size() != len)
    {
        if(b.size() && (c.empty() || b.front() <= c.front()))
        {
            a.push(b.front());
            b.pop();
        }
        if(c.size() && (b.empty() || c.front() < b.front()))
        {
            a.push(c.front());
            c.pop();
        }
    }
    for(int i = low; i < high; ++i)
    {
        arr[i] = a.front();
        a.pop();
    }
}

void mergeSort(int * arr, int low, int high)
{
    if(high - low < 2) 
        return ;
    int mid = (low + high) >> 1;
    mergeSort(arr, low, mid);
    mergeSort(arr, mid, high);
    merge(arr, low, mid, high);
}


int main()
{
    const int maxI = 10;
    int arr[maxI] ;
    srand(time(NULL));
    for(int i = 0; i < maxI; ++i)
    {
        arr[i] = rand() % 100;
    }
    for(int i = 0; i < maxI; ++i)
    {
        printf("%3d", arr[i]);
    }
    printf("\n");
    mergeSort(arr, 0, maxI);
    for(int i = 0; i < maxI; ++i)
    {
        printf("%3d", arr[i]);
    }
    printf("\n");
    return 0;
}
```
唯一要注意的是这里递归基的问题。我们让其在high = low + 1的时候结束递归，因此
mergeSort进行到队列中只有一个元素的时候回去merge。
