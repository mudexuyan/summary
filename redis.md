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
   1. 连锁更新问题，ziplist更新效率低
   2. linkedlist存在大量内存碎片
![quicklist](图片/quicklist.png)
```
typedef struct quicklistNode {
    struct quicklistNode *prev;
    struct quicklistNode *next;
    unsigned char *zl;
    unsigned int sz;             /* ziplist占用byte数*/
    unsigned int count : 16;     /* count of items in ziplist */
    unsigned int encoding : 2;   /* RAW==1 or LZF==2 */
    unsigned int container : 2;  /* NONE==1 or ZIPLIST==2 */
    unsigned int recompress : 1; /* was this node previous compressed? */
    unsigned int attempted_compress : 1; /* node can't compress; too small */
    unsigned int extra : 10; /* more bits to steal for future usage */
} quicklistNode;

typedef struct quicklistLZF {
    unsigned int sz; /* LZF size in bytes*/
    char compressed[];
} quicklistLZF;

typedef struct quicklist {
    quicklistNode *head;
    quicklistNode *tail;
    unsigned long count;        /* 所有ziplist中entry的数量 */
    unsigned long len;          /* quicklistNode的数量 */
    int fill : 16;              /* fill factor for individual nodes */
    unsigned int compress : 16; /* depth of end nodes not to compress;0=off */
} quicklist;

```

# 对象
1. 字符串，底层实现：整数值、embstr编码的SDS字符串、SDS
2. 列表，底层实现：ziplist、双端链表；quicklist
3. 哈希，底层实现：ziplist、字典
4. 集合，底层实现：intset、字典
5. 有序集合，底层实现：ziplist、跳跃表和字典
   
## 字符串对象，可以唯一被其它四个对象嵌套的对象
1. 存的是整数，并且可以用long表示，底层用int存储
2. 存的是浮点数，等同于存字符串
3. 存的是字符串，长度大于32字节，用SDS存储
4. 存的是字符串，长度小于等于32字节，用embstr编码的SDS存储
5. 当int、或embstr的对象改变时，会进行编码转换成SDS
6. set get append strlen...

## 列表对象
1. 3.2之前
   1. ziplist，所有字符串元素长度小于64字节，且元素数量小于512。
      1. 插入删除时需要申请和释放内存，同时会发生内存复制，更新效率低
   2. linkedlist，其它情况
      1. 每个节点保存前后指针、且都是独立的内存块，地址不连续，容易产生碎片
2. 3.2之后，quicklist
3. rpush lpush rpop lpop linsert len...


## 哈希对象
1. ziplist，将键和值作为两个节点放到表尾
2. hashtable
3. 编码转换：键值对小于512个，且键值对的长度都小于64字节用ziplist；否则用hashtable
4. hdel hset hget hlen...

## 集合对象
1. intset
2. hashtable
3. 编码转换，所有元素是整数，且数量不超过512，用intset；否则用hashtable
4. sadd spop sinter scard...


## 有序集合
1. ziplist，分别用两个节点存放成员和分值，这两个节点紧挨一起
2. skiplist（范围操作o(n)）+字典（查找操作o(1)），字典和跳跃表共享元素的成员和分值
3. 编码转换：数量小于128，长度小于64字节用ziplist，否则用skiplist
4. zadd zcard zrank zscore

## 基于类型多态命令
1. del expire rename type object...

## 内存回收
1. 构建对象的引用计数属性refcount，为0时内存释放
2. 对象生命周期：创建对象、操作对象、释放对象

## 对象共享
1. 引用计数属性也可用于共享，每次共享计数加1
2. 只共享整数，共享需要验证对象相等
   1. 验证整数字符串，o（1）
   2. 验证字符串，o（n）
   3. 验证多个值对象，o（n2）
3. redis初始化服务器时，会创建一万个字符串对象，0~9999，当需要用到这些对象时，服务器会共享这些对象，而不是创建新对象。redis只会对着一万个对象进行共享，这一万个对象的refcount 为INT_MAX，表示不被销毁的全局对象。同时，OBJ_SHARED_INTEGERS（10000）可以改变

## 对象空转
1.  redis对象包含属性：type（查看类型）、encoding（查看底层结构）、ptr、refcount（计数引用）、lru（最后一次访问时间）

# 单机数据库的实现
## 数据库
数据库由dict（保存键值对）和expires（保存过期时间）两个字典构成
1. 有16个数据库，0~15
   1. 添加键
   2. 删除
   3. 更新
   4. 取值
   5. dbsize exists rename keys
2. 设置键的生存时间或过期时间
   1. 命令
      1. expire，以秒为单位设置生存时间
      2. pexpire，以毫秒为单位设置生存时间
      3. expireat，设置过期的时间戳，到时间则过期，unix时间戳，long long 保存
      4. pexpireat，同上
      5. persist，移除过期时间
      6. ttl、pttl，查看键的剩余时间
   2. 过期判定过程
      1. 键是否存在于过期字典中；如果在，取得过期时间
      2. 根据当前unix时间戳判断是否大于过期时间
   3. 过期删除策略
      1. 定时删除，设置键的同时，创建定时器，时间一到立即删除。对内存友好，对cpu时间不友好（不现实）
      2. 惰性删除，只有在获取键的时候，查看键有没有过期，过期就删除。对cpu时间友好，对内存不友好
      3. 定期删除，上面的折中。每隔一段时间对数据库进行一次检查，删除过期键
   4.  redis采用惰性删除和定期删除的结合
       1.  惰性删除实现，读写数据库的命令，通过惰性删除检查键
       2.  定期删除实现，规定时间内，多次访问各个数据库，从数据库的过期字典中随机检查间检查一部分键的过期时间，并删除其中的过期键 
   5.  AOF、RDB和复制功能对过期键的处理
       1.  RDB文件
           1.  save、bgsave在生成RDB文件时，会检查数据库中的键，过期间不会被保存到RDB文件中
           2.  载入RDB文件时，
               1.  当前服务器以主服务器模式运行时，会对文件中保存的键进行检查，过期的则忽略
               2.  以从服务器模式运行时，无论有没有过期都会被载入
       2. AOF文件
          1. 文件写入，服务器持久化模式运行（AOF）时，键过期没有被删除，没有影响；删除了就向AOF文件追加一条del命令
          2. AOF重写，类似于RDB生成，对数据库的键进行检查，过期的不保存
       3. 复制
          1. 过期键的删除由主服务器控制
          2. 主服务器删除过期键时，会向所有从服务器追加del命令
          3. 从服务器只有在接受del命令，才会删除过期键
   6.  数据库通知
       1.  
## RDB持久化
    


# 多机数据库的实现
15~17


# 独立功能呢的实现
18~24


# redis相关指令
1. 查看底层结构，object encoding name
2. 查看对象类型，type name