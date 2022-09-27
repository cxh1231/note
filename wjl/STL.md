## STL原理

### STL六大组件

#### 容器

```cpp
vecotr
list
deque
set
map
```

#### 迭代器

前向迭代器，双向迭代器，随机迭代器

逆迭代器

只读迭代器

输入迭代器，输出迭代器

#### 适配器

修饰容器或者仿函数迭代器之类的东西，改变原本class接口；设计模式中描述了适配器模式，将一个class的接口转换为另一个class的接口，使原本接口不兼容或者不能合作的class可以一起运作

比如deque，stack，priority_queue

#### 分配器

负责控件的配置和管理，地城是malloc获得new

#### 算法

```cpp
sort
find							//找到第一个
cppy(start, end, container.begin) //复制[statr,end)到container中，不开批控件
for_each
```



#### 仿函数

行为类似函数，可作为算法的某种策略，实现上看是从在了operator()的类或是模板类

###  STL优点

重用性、性能、移植性，将数据和操作分离

## pair

```cpp
pair<T1, T2> p;
```

map的元素是pair，但key的类型是const的

insert对不包含关键字的容器，冲入成功返回pair<迭代器，bool>迭代器指向关键字元素， bool指出是否成功

## vector容器扩容

### 底层逻辑

在堆上分配了一段连续的内存空间来存放元素

三个迭代器：

first起始字节

last最后一个字节的末尾字节

end内存末尾

### 扩容过程

内存不够的时候，分配更大的内存，复制原本数据，释放原本内存

`size()`存储元素个数

`capacity()`不分配新内存的时的大小

翻倍gnu/1.5倍vs增长

`reserve()`扩容控件，不会减少

## list链表

内存不连续，通过指针访问，本质是双向链表

> vector和list的区别

1. vector是动态数组，list是双向链表
2. vector顺序内存，随机访问，list不行
3. 中间插入和删除vector需要内存拷贝，list不会
4. vector有剩余空间，动态扩容，list每次都会申请内存
5. 前者随机访问，后者中间插入删除

## deque双端队列

支持随机访问，头尾都可以删除添加，复杂度O(1)

### deque中控器

deque是一段一段的定量连续空间构成，二级分配
**优点**：可以随机访问，头尾都可以删除和添加(map没有到达极限的情况下)
**缺点**：迭代器复杂



## stack与queue

stack栈与queue队列是适配器，默认是用==deque==实现的

## heap与priority_queue

### heap

建立在完全二叉树上，分为两种，大根堆和小根堆，任何顺序将元素推入容器，一定从优先权最高的元素开始取。

### priority_queue

有限队列适配器默认是max-heap，默认的容器是vector，这里的compare如果为true，会让元素下沉也就是说如果compare是小于，则是最大堆

```cpp
template<class T, class Sequence=vector<T>, class Compare = less<typename Sequence::value_type>>
class priority_queue;
```

## map&&set

关联容器，使用红黑树实现

set是集合

map是键值对映射，相当于字典

优点：珊茹删除和查询的复杂度是O(logn)；遍历时有序遍历

缺点插入和删除值会调整红黑树

## map&&unordered_map

map基于红黑树实现，元素顺序有序

unordered_map基于哈希表实现，元素是无序的

map查找删除增加复杂度为O(logn)

unordered_map查找删除增加的平均复杂度O(1)

缺点是哈希表的空间占有率高，删除增加的时间复杂度不稳定

map和set的迭代器是指向节点的指针，insert后会不会失效

> map和unordered_map的迭代器
>
> 红黑树节点迭代器；记录了左右子节点和父节点，中序遍历的方式访问
>
> 桶+链表迭代器；记录桶的指针和链表内的指针

## C++泛型编程

### 全特化和偏特化

模板分为类模板与函数模板，特化分为特例化(全特化)和部分特例化(偏特化)。

对模板特例化是因为特定类型，可以利用特定知识来提高效率，而不是通用模板

```cpp
template<typename T1, typename T2>
class Test
{
public:
Test(T1 i,T2 j):a(i),b(j){cout<<"ཛྷ຃ᔄ"<<endl;}
private:
T1 a;
T2 b;
};
template<>
class Test<int , char>
{
public:
Test(int i, char j):a(i),b(j){cout<<"قᇙ۸"<<endl;}
private:
int a;
char b;
};
```



