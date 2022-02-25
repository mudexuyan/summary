# 数据结构和对象
键是字符串对象
值是字符串对象String、列表对象List、哈希对象Hash、集合对象Set、有序集合对象Zset
特殊数据类型：Geospatial（地理位置）、Hyperloglog（基数统计）、BitMaps（位图）
## 简单动态字符串SDS
1. 与c语言字符串相比：  
   1. 都以空字符结尾
   2. sds记录自身长度，c字符串不记录
2. sds的优势
   1. 获取长度不需遍历整个字符串（length）
   2. 在拼接时不会造成缓冲区溢出（length）
   3. 减少修改字符串带来的内存分配次数（free）
   4.  sds二进制安全，c字符串由于编码限制，不能保存图片、音频、视频压缩文件这样的二进制数据
   5.  可以使用部分'<string.h>'库中的函数
3. sds free字段的作用
   1. 实现空间预分配（优化字符串增长）和惰性空间释放（优化字符串减少），避免每次修改字符串都需要重新分配内存
```
struct sds{
    //记录字符的长度
    int len;
    //buf数组还能使用的字节数
    int free;
    //保存字符串，以'\0'结尾
    char buf[];
}
```
## 链表，无环双向
用于列表键、发布与订阅、慢查询、监视器等
```
typedef struct listNode{
    struct listNode* pre;
    struct listNode* next;
    void *val;//各种类型
}listNode;


typedef struct list{
    listNode *head;
    listNode* tail;
    //结点个数
    unsigned long len;
    //复制函数
    void dup(listNode* p);
    //释放函数
    void free(listNode* p);
    //对比函数
    int match(listNode* p,listNode* q);

}list;

```
## 字典
redis数据库底层实现  
```
//哈希表节点
typedef struct dictEntry{
    //键
    void *key;
    //值
    union{
        void *val;
        int val1;
        unsigned_int val2;
    }v;
    //哈希表节点，形成链表，指向下一个哈希值相同的键值对，解决哈希冲突
    struct dictEntry *next;
}dictEntry;


//哈希表

```
### 通过链地址法解决哈希冲突
### rehash，维持哈希因子在合理范围

## 跳跃表
Zset的底层实现之一、集群节点
查找复杂度，平均lgN，最差N

跳跃表节点： 
header：表头节点
tail：表尾节点
level：层数最大节点的层数，每层含有前进指针和跨度
length：跳跃表长度
后退指针：指向前一个节点
分值
对象

## 整数集合intset
```
typedef struct intset{
    //编码方式
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组
    int8_t contents[];
}intset;
```

# 单机数据库的实现
9~14
# 多机数据库的实现
15~17
# 独立功能呢的实现
18~24