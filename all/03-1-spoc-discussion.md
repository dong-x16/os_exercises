# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  

>  答：最优匹配：最优匹配需要从开始到结束搜索所有的空闲区域，优点是可以避免将大块的空闲内存区域分割成小块的内存区域，提高了
大块内存区域的利用率，缺点是查询速度较慢，同时容易产生较多的内存碎片，并且由于碎片较小多数难以再次利用；
       最差匹配：最差匹配与最优匹配设计思路相反，是搜索所有空闲区域寻找最大的内存空闲区域，优点是分割后的空闲区域仍比较大，
方便后续使用，产生的内存碎片比较少，缺点是查询速度比较慢，并且将大块的内存区域分割后不利于之后保存需要较大内存区域的文件；
       最先匹配：最先匹配使用搜索到的第一个足够大的内存区域，优点是不需要全部变量比较，查询速度很快，缺点是既可能分割了大块的内存区域又容易产生内存碎片；
       buddy systemm：有点是避免了前面三种方法产生的大量内存碎片，分配与回收都比较高效，缺点是可能由于小块的内存未释放而影响
周围大块内存的合并，同时如果需要的内存大小不是2的幂次，会有页面浪费。

> 其他内存分配算法：可以从上次命中的内存区域之后循环搜索，这样可以保证空闲的内存区域分布的更均衡，能加快搜索速度，提高命中率，但会造成缺乏大块的内存区域。
       
       


## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)
```
如何表示空闲块？ 如何表示空闲块列表？ 
[(start0, size0),(start1,size1)...]
在一次malloc后，如果根据某种顺序查找符合malloc要求的空闲块？如何把一个空闲块改变成另外一个空闲块，或消除这个空闲块？如何更新空闲块列表？
在一次free后，如何把已使用块转变成空闲块，并按照某种顺序（起始地址，块大小）插入到空闲块列表中？考虑需要合并相邻空闲块，形成更大的空闲块？
如果考虑地址对齐（比如按照4字节对齐），应该如何设计？
如果考虑空闲/使用块列表组织中有部分元数据，比如表示链接信息，如何给malloc返回有效可用的空闲块地址而不破坏
元数据信息？
伙伴分配器的一个极简实现
http://coolshell.cn/tag/buddy
```
答：学号是2012011361；选择1：最差匹配 
```
#include <iostream>
#include <cstdlib>
#include <cstdio>
using namespace std;

struct Node {     //一个内存块
    int first;  //指向mem开始的地址
    int size;     //内存块的大小
    bool is_used; //内存块是否被占用
    Node *pre;    //前一个内存块
    Node *next;   //后一个内存块
};

Node *head = new Node; //内存链表

//内存区域初始化
void init(int first, int size) {  
    head->first = first;  
    head->size = size;  
    head->is_used = false;
    head->pre = NULL;
    head->next = NULL;  
}  


void malloc(int size) {
    Node *node = head;
    Node *aim = head;
    int max_length = -1;
    while (node != NULL) {
        if (node->size > max_length && node->is_used == false ) {
            max_length = node->size;
            aim = node;
        }
        node = node->next;
    }

    if (max_length < size) {
        printf ("没有找到合适大小的内存！");
        return;
    }
    else {
        Node *split = new Node;
        split->first = aim->first;
        split->size = size;
        split->is_used = true;
        split->next = aim;
        split->pre = aim->pre;
        aim->first = aim->first + split->size;
        aim->size = aim->size - size;
        aim->is_used = false;
        aim->pre = split;
    }
}

void free(Node *temp) {
    if ((temp->pre) != NULL) {
        bool empty_pre = (temp->pre)->is_used;
        if (empty_pre == false) {
            (temp->pre)->size = (temp->pre)->size + temp->size;
            (temp->pre)->next = temp->next;
            (temp->next)->pre = temp->pre;
        }
    }
    if ((temp->next) != NULL) {
        bool empty_next = (temp->next)->is_used;
        if (empty_next == false) {
            ((temp->next)->pre)->size = ((temp->next)->pre)->size  + (temp->next)->size;
            if ((temp->next)->next != NULL) {
                ((temp->next)->pre)->next = (temp->next)->next;
                ((temp->next)->next)->pre = temp->pre;
            }
            else 
                ((temp->next)->pre)->next = NULL;
        }
    }
}

void print() {
    Node *temp = head;
    printf("*****************************");
    while (temp != NULL) {
        printf(temp->first);
	printf(" ");
	printf(temp->size);
       	printf(" ");
       	printf(temp->is_used);
       	printf("\n");
        temp = temp->next;
    }
}

int main() {
    init(0, 1024);
    print();
    malloc(50);
    malloc(100);
    print();
    return 0;
}

```

--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



