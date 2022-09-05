## 1、Redis 底层数据结构与数据类型对应关系

![image-20220826154832707](https://img.zxdmy.com/2022/202208261548081.png)

## 2、简单动态字符串（SDS）

虽然 Redis 是用 C 语言写的，但是 Redis 并没有使用 C 的字符串表示，而是自己构建了一种 **简单动态字符串**（`Simple Dynamic String`，`SDS`）。

相比于 C 的原生字符串，Redis 的 `SDS` 不光可以保存文本数据，还可以保存二进制数据，并且获取字符串长度复杂度为 `O(1)`（C 字符串为 O(N)），除此之外，Redis 的 `SDS API` 是安全的，**不会造成缓冲区溢出**。

其定义结构如下：

```c
struct sdshdr{
     //记录 buf 数组中已使用字节的数量，等于 SDS 所保存字符串的长度
     int len;
     
     //记录 buf 数组中未使用字节的数量
     int free;
     
     //字节数组，用于保存字符串
     char buf[];
}
```

![image-20220826155143034](https://img.zxdmy.com/2022/202208261551039.png)

## 3、双向链表

Redis 基于 C 定义的 `链表节点` 如下：

```c
typedef struct listNode {
    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;
```

![image-20220826155324815](https://img.zxdmy.com/2022/202208261553823.png)

在上面的`链表节点`的基础上， `Redis` 额外定义了一个 list 结构，用来保存该链表的更多信息：

```c
typedef struct list {
    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 链表所包含的节点数量
    unsigned long len;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void (*free)(void *ptr);

    // 节点值对比函数
    int (*match)(void *ptr, void *key);

} list;
```

示例：一个 list 结构和三个 listNode 结构组成的链表

![image-20220826155539420](https://img.zxdmy.com/2022/202208261555525.png)

+ Redis 的链表，双端，无环；
+ 带有 **表头指针** `head` 和 **表尾指针** `tail`，获取链表的**表头节点**和**表尾节点**的复杂度为 `O(1)` ；
+ 带有 **链表长度计数器** `len`，程序获取链表中**节点数量**的复杂度为 `O(1)` 。

## 4、压缩列表

压缩列表（ziplist）是列表键和哈希键的底层实现之一。

当一个列表键只包含少量列表项， 并且每个列表项要么就是小整数值， 要么就是长度比较短的字符串， 那么 Redis 就会使用压缩列表来做列表键的底层实现。

## 5、哈希表

`字典` 又称为符号表或者关联数组、或`映射`（`map`），是一种用于**保存键值对的抽象数据结构**。

`Redis` 中定义的 `哈希表`（字典）结构如下：

```c
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;

    // 哈希表大小
    unsigned long size;

    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;

    // 该哈希表已有节点的数量
    unsigned long used;

} dictht;

```

`哈希表节点` 使用 `dictEntry` 结构表示， 每个 `dictEntry` 结构都保存着一个键值对：

```c
typedef struct dictEntry {
    // 键
    void *key;

    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;

    // 指向下个哈希表节点，形成链表，用于解决哈希冲突
    struct dictEntry *next;

} dictEntry;
```

比如有一个大小为 `4` 的`哈希表`，有两个`哈希表节点` `k1` 和 `k2`，其`哈希值`均为 `2`，存在`哈希冲突`，这时，其存储结构如下图所示：

![image-20220826162016723](https://img.zxdmy.com/2022/202208261620404.png)

`Redis` 中的 **字典** 由 `dict` 结构表示：

```c
typedef struct dict {
    // 类型特定函数
    dictType *type;

    // 私有数据
    void *privdata;

    // 哈希表，是数组形式，一般只使用 ht[0]，在 rehash 时，才用 ht[1]
    dictht ht[2];

    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; 
    /* rehashing not in progress if rehashidx == -1 */

} dict;
```

对于上面的 类型特定函数，其定义如下：

```c
typedef struct dictType {
    // 计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);

    // 复制键的函数
    void *(*keyDup)(void *privdata, const void *key);

    // 复制值的函数
    void *(*valDup)(void *privdata, const void *obj);

    // 对比键的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);

    // 销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);

    // 销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);

} dictType;
```

最终的完整的字典结构如下图所示：

![image-20220826163044203](https://img.zxdmy.com/2022/202208261630689.png)

## 6、跳跃表

跳跃表（skiplist）是一种有序数据结构，它通过在每个节点中维持多个指向其它节点的指针，从而达到快速访问节点的目的。

具有如下性质：

1. 由很多层结构组成；
2. 每一层都是一个有序的链表，排列顺序为由高层到底层，都至少包含两个链表节点，分别是前面的head节点和后面的nil节点；
3. 最底层的链表包含了所有的元素；
4. 如果一个元素出现在某一层的链表中，那么在该层之下的链表也全都会出现（上一层的元素是当前层的元素的子集）；
5. 链表中的每个节点都包含两个指针，一个指向同一层的下一个链表节点，另一个指向下一层的同一个链表节点；

![image-20220826160949763](https://img.zxdmy.com/2022/202208261609053.png)

## 7、整数数组

整数集合（`intset`）是集合键的底层实现之一，当一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis 就会使用集合作为集合键的底层实现。

整数集合（intset）是Redis用于保存整数值的集合抽象数据类型，它可以保存类型为int16_t、int32_t 或者int64_t 的整数值，并且保证集合中不会出现重复元素。

```c
typedef struct intset {
    // 编码方式
    uint32_t encoding;

    // 集合包含的元素数量
    uint32_t length;

    // 保存元素的数组
    int8_t contents[];

} intset;
```

比如一个包含 5 个 int16_t 类型的整数值的整数集合：

![image-20220826161209221](https://img.zxdmy.com/2022/202208261612171.png)

