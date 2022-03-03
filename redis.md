# 数据结构
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
1. 字典组成
   1. 类型特定函数
   2. 私有数据
   3. 哈希表
   4. rehash索引，未进行rehash值为-1
2. 哈希表
   1. 哈希表数组
   2. 哈希表大小，size
   3. 哈希表大小掩码，用于计算索引，等于size-1
   4. 已使用的节点数量，used
3. 节点
   1. 键
   2. 值
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
### rehash，维持哈希因子在合理范围，对哈希表大小进行扩展和收缩

## 跳跃表
Zset的底层实现之一、集群节点
查找复杂度，平均lgN，最差N
1. 组成
   1. 节点
   2. zskiplist
2. 跳跃表节点： 
   1. level，层数
      1. 前进指针
      2. 跨度
   2. 后退指针：指向前一个节点
   3. 分值
   4. 成员对象，SDS
3. zskiplist
   1. 表头节点、表尾节点
   2. length，节点数量
   3. level，层数最大的节点的层数

## 整数集合intSet，有序、无重复
当只包含整数值元素，且数量不多时，redis采用整数集合作为set的底层实现。
```
typedef struct intset{
    //编码方式
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组，虽然声明为int8，但实际类型根据encoding属性的值。encoding为INTSET_ENC_INT32，contents数组类型为int32。contents有序、无重复
    int8_t contents[];
}intset;
```
### 升级，intset不支持降级
当添加的新元素类型比集合现有元素的类型都要长时，需要对intset升级，这样才能添加新元素
1. 根据新元素的类型扩展intset空间大小，并分配空间
2. 将intset现有元素的类型都转换成新元素的类型
3. 添加新元素
提升灵活性（随意添加类型），节约内存（只在特殊情况进行升级）


## 压缩列表ziplist，节约内存
1. 列表键、哈希键的实现之一。当一个哈希键只包含少量列表项，并且每个列表项是小整数或短字符串，使用ziplist作为列表键的底层实现
![ziplist](图片/ziplist.png)  
2. ziplist节点
   1. previous_entry_length，前驱节点的长度。前驱节点长度小于254字节，当前节点用1字节保存这个值；不小于254字节，用5个字节
   2. encoding，保存内容的类型及长度，不同的编码区分整数和数组
   3. content，整数或字节数组
3. 连锁更新
   1. 每个节点记录了前驱节点的长度，当添加节点的长度超过254字节，可能会导致之后的所有节点previous_entry_length都需要扩展为5字节（原本为1字节）
   2. 删除节点也会导致同样情况
## 快速列表quicklist
1. 列表键的实现之一

# 对象
1. 字符串，底层实现：整数值、embstr编码的SDS字符串、SDS
2. 列表，底层实现：ziplist、双端链表
3. 哈希，底层实现：ziplist、字典
4. 集合，底层实现：intset、字典
5. 有序集合，底层实现：ziplist、跳跃表和字典

# 单机数据库的实现
9~14
# 多机数据库的实现
15~17
# 独立功能呢的实现
18~24


# redis相关指令
1. 查看底层结构，object encoding name
2. 查看对象类型，type name