# bigDecimal
## 浮点数没有办法用二进制精确表示，因此存在精度丢失的风险。
```
float aa = 2.0f - 1.9f;
float bb = 1.8f - 1.7f;
System.out.println(aa);// 0.100000024
System.out.println(bb);// 0.099999905
System.out.println(aa == bb);// false
```
## 比较大小
a.compareTo(b) : 返回-1表示a小于b，0表示a等于b，1表示 a 大于 b。
## 常见加减乘除
```
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
System.out.println(a.add(b));// 1.9
System.out.println(a.subtract(b));// 0.1
System.out.println(a.multiply(b));// 0.90
System.out.println(a.divide(b));// 无法除尽，抛出 ArithmeticException 异常
System.out.println(a.divide(b, 2, RoundingMode.HALF_UP));// 1.11

```

# 集合
![集合](图片/集合.jpg)  
1. 集合接口，存放单一元素
   1. list
   2. set
   3. queue
2. map接口，存放键值对
   1. map
## list
特征：有序，可重复
1. arrayList，线程不安全，底层用的是object数组
2. vector，线程安全，底层用的是object数组
3. LinkedList，线程不安全，底层使用的是双向链表，JDK1.6之前为循环链表，JDK1.7取消循环
4. 总结1:arrayList扩容机制：
   1. 当容量等于0的情况，根据构造方法改变。无参构造，容量变10；传容量参数构造，当参数为0时，添加1个元素后，容量为1，再正常扩容（参数大于0则容量为当前参数大小）；传列表构造，列表为空时，添加1个元素后，容量变1，正常扩容（列表不为空则容量和传入列表长度相等）。
   2. 当容量大于0时，扩容为当前容量的1.5倍
   
## set
特征：无序、不可重复
1. hashSet，基于hashMap实现
2. LinkedHashSet，基于LinkedHashMap实现
3. TreeSet，红黑树，有序，唯一

## queue
特征：有序，可重复
1. priorityQueue：优先队列，二叉堆
2. arrayQueue，数组+双指针，可实现栈

## map
特征：键有序，不可重复；值无序、可重复
1. hashMap
   1. jdk1.8之前，有数组+链表组成，链表用于解决哈希冲突
   2. jdk1.8后，由数组+链表+红黑树组成
2. LinkedHashMap，双向链表哈希表
3. HashTable
4. TreeMap，红黑树
5. 总结1-hashMap冲突，当发生哈希冲突后，链表长度加1，当长度超过阈值时（默认为8），将链表转化为红黑树，减少搜索时间。（转换前，先检测当前数组的长度，如果当前数组的长度小于64，会先选择将数组扩容，而不是转换成红黑树）。当长度低于阈值时，会从红黑树转化成链表。
6. 总结2-hashMap使用数组（自定义2倍扩容），不使用linkedList（不能随机访问，实现O（1）查找），不使用arrayList（固定是1.5倍扩容，原因1.5 倍可以充分利用移位操作，减少浮点数或者运算时间和运算次数）。数组扩容时，同时已插入的key会rehash到新数组下标，当哈希冲突时根据头插法插入链表中，此时可能导致前后节点的链表翻转，在多进程情况下形成环，导致死循环。
7. HashMap与Hashtable的区别
   1. HashMap：数组+链表+红黑树，非线程安全、效率高、key和value可以是null（key为null只能有一个）。（1）不指定初始容量时，数组+链表，初始容量为16，每次扩容为之前的2倍（2）指定初始容量时，将其扩容为2的幂次方大小
   2. Hashtable：key和value不可以是null；如果指定了初始容量，则直接使用作为初始容量，否则初始容量为11，每次扩容变为原来的2n+1
8. 实现Hashmap线程安全
   1. 不安全的原因：多线程rehash会导致环形连接死循环
   2. 使用hashtable、使用concurrenthashMap、使用collections将hashMap包装成线程安全的map
9. concurrentHashMap与hashtable的异同：都保证线程安全
   1.  concurrentHashMap（分段锁）：jdk1.8分段数组+链表/红黑树组成；分段锁保证线程安全：jdk1.7对数组分段，每把锁只锁一部分数据，多线程访问不同段的数据；jdk1.8，使用node数组+链表+红黑树，并发控制使用synchronized（悲观锁）和cas（乐观锁）操作
   2.  hashtable（全表锁）：数组+链表组成，使用synchronized保证线程安全


# 并发

# Java新特性




