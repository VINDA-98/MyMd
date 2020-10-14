[TOC]



# HashMap

<br />



## 1、什么是HashMap？

​	Hashmap是基于哈希表（散列表）实现的Map接口的双列集合，数据结构是“列表散列”,也就是数组+链表，而在JDK1.8中，HashMap的存储结构多了红黑树，也就是链表长度大于八的时候，使用红黑树来存储数据。HashMap中key是唯一的，而value是允许重复，允许存储null K 和null V ，元素没有顺序。

JDK1.7：

![123](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/JDK1.7%E5%89%8D%E7%9A%84HaspMap%E5%9B%BE.png)





JDK1.8：



![](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/JDK1.8%E7%9A%84HaspMap%E5%9B%BE.png)

两张图来源于这位大佬的文章：https://www.cnblogs.com/xiaoxi/p/7233201.html

<br />



## 2、HashMap与HashTable的区别？

​	1、HashMap线程不安全，HashTable使用了synchronized关键字，是线程安全的

HashTable构造函数源码，putAll()方法被synchronized关键字修饰：

```java
public Hashtable(Map<? extends K, ? extends V> t) {
    this(Math.max(2*t.size(), 11), 0.75f);
    putAll(t);
}

public synchronized void putAll(Map<? extends K, ? extends V> t) {
        for (Map.Entry<? extends K, ? extends V> e : t.entrySet())
            put(e.getKey(), e.getValue());
    }
```

  	2、HashMap允许key和Value都为空值；HashTable的key和Value都不允许为空值；

​	  3、HashMap继承AbstractMap类，而Hashtable继承Dictionary类；

```javascript
public class HashMap<K,V> 
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable{}

public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable,Serializable {}
```

<br />



## 3、HashMap的put方法的具体流程？

​	从源码解，首先是HashMap的put方法实现如下，put方法内部主要是调用了putVal（）方法：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

​	putVal（）方法解析如下：

```java
/**
 * Implements Map.put and related methods.
 *
 * @param hash hash for key   通过哈希函数得到的哈希（散列）值
 * @param key the key		  	 传入进来的Key值
 * @param value the value to put 要存储的value
 * @param onlyIfAbsent 如果当前位置已存在一个值，是否替换，false是替换，true是不替换
 * @param evict  表示是否在创建模式，如果为false，则表示是在创建模式。
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    //参数初始化
    Node<K,V>[] tab;  //存储（位桶）的数组
    Node<K,V> p; 
    int n, i;
	//1、判断table数组是否为空，为空则就初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        //1.1 resize()完成
        n = (tab = resize()).length;
    //2、判断table数组以为n-1 & hash 是否为空
    if ((p = tab[i = (n - 1) & hash]) == null)
        //2.1 放入元素
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //3、如果桶中存在的元素key和hash值相等，会覆盖掉旧的value
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //3.1覆盖旧的value
            e = p;
        /*4、判断链表是否需要转为红黑树，如果需要转为红黑树，
        则将新的value直接插入，同时第三步是不成立的，因为hash值不相等*/
        else if (p instanceof TreeNode)
        	//4.1 存放到红黑树
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //5、第三步和第四步都不成立，元素直接插入到链表中
        else {
            //遍历链表
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //5.1判断插入元素的hash值和key是否与链表中某个元素内容相同，
                //5.2如果相同，就跳出遍历循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //6、链表中存在key值和hash值相等的，直接覆盖旧value
        if (e != null) { 
            // existing mapping for key //取出e的value
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                //覆盖旧的value
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    //7、将对HashMap操作记录+1
    ++modCount;
    //8、判断是否需要扩容
    if (++size > threshold)
        //扩容
        resize();
    //不需要扩容，插入元素后回调
    afterNodeInsertion(evict);
    return null;
}
```



![](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/put%E6%96%B9%E6%B3%95%E5%AE%9E%E7%8E%B0%E6%B5%81%E7%A8%8B.png)

流程图来自：https://blog.csdn.net/u011240877/article/details/53358305



<br />



put方法小结：

1、判断数组是否为空，是否要进行扩容操作

2、根据key值计算得到对应hash值，插入对应数组索引

3、对应数组索引元素为空，可以直接插入，不为空说明键值存在，直接覆盖掉原来的value

4、如果key不存在，再判断是否是红黑树结构，是就直接插入元素，不是红黑树结构则准备遍历链表插入

5、记录操作，检查是否需要扩容

​	

<br />



## 4、hash算法

直接上源码

```java
/*
	Object：传入key值
*/
static final int hash(Object key) {
    int h;  //int 类型 -（2^31）~ 2^31-1
    //key为null,返回0，h为根据key计算的hash值，先右移十六位，再进行异或运算，两次扰动
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



<br />



## 5、HashMap怎么解决hash冲突？

==1.使用散列表（使用链地址法）来链接拥有相同hash值的数据；==

==2、JDK8中引入红黑树降低遍历集合时间复杂度，使得遍历更快，并且相对于1.7的九次扰动，1.8中HashMap使用了两次扰动函数来降低哈希冲突的概率，使得hash值高低位都参与运算，使得数据分布更加均匀==



<br />



## 6、HashMap在JDK1.7和JDK1.8中有哪些不同？

**JDK 1.7 ：**

存储结构： 数据+链表

插入方法 ：链表头插

hash值计算 ：4次位运算 + 5次异或运算

存放数据 ：无冲突时，存放数组；冲突时，存放链表

扩容机制：全部按照原来方法进行计算（即hashCode ->> 扰动函数 ->> (h&length-1)）

<br />



**JDK 1.8 ：**

存储结构：  数组 + 链表 + 红黑树

插入方法 ：链表尾插

hash值计算 ：  1次位运算 + 1次异或运算

存放数据 ：  无冲突时，存放数组；冲突 & 链表长度 < 8：存放单链表；冲突 & 链表长度 > 8：树化并存放红黑树

扩容机制：  按照扩容后的规律计算（即扩容后的位置=原位置 or 原位置 + 旧容量）全部按照原来方法进行计算（即hashCode ->> 扰动函数 ->> (h&length-1)）



<br />



## 7、ConcurrentHashMap和Hashtable的区别？

​		ConcurrentHashMap 底层采用了数组+链表，线程安全，HashMap 没有考虑同步，HashTable 考虑了同步的问题。但是 HashTable 在每次同步执行时都要锁住整个结构。 ConcurrentHashMap 锁的方式相对于HashTable粒度要小， 允许多个修改操作并发进行。



<br />



## 8、ConcurrentHashMap的具体实现

 		HashMap线程不安全,而Hashtable是线程安全,但是它使用了synchronized进行方法同步,插入、读取数据都使用了synchronized,当插入数据的时候不能进行读取(相当于把整个Hashtable都锁住了,全表锁),当多线程并发的情况下都要竞争同一把锁,导致效率极其低下.而在JDK1.5后为了改进Hashtable的痛点,ConcurrentHashMap应运而生.

**jdk1.7分段锁的实现**

​		和hashmap一样，在jdk1.7中ConcurrentHashMap的底层数据结构是数组加链表。和hashmap不同的是ConcurrentHashMap中存放的数据是一段段的，即由多个Segment(段)组成的。每个Segment中都有着类似于数组加链表的结构。

<br />



​	**JDK1.8中的实现**
​		ConcurrentHashMap取消了segment分段锁,而采用CAS和synchronized来保证并发安全.原先table数组＋单向链表的数据结构，变更为table数组＋单向链表＋红黑树的结构。对于hash表来说，最核心的能力在于将key hash之后能均匀的分布在数组中。如果hash之后散列的很均匀，那么table数组中的每个队列长度主要为0或者1。但实际情况并非总是如此理想，虽然ConcurrentHashMap类默认的加载因子为0.75，但是在数据量过大或者运气不佳的情况下，还是会存在一些队列长度过长的情况，如果还是采用单向列表方式，那么查询某个节点的时间复杂度为O(n)；因此，对于个数超过8(默认值)的列表，jdk1.8中采用了红黑树的结构，那么查询的时间复杂度可以降低到O(logN)，可以改进性能。

​	<br />



**关于Segment**

ConcurrentHashMap有3个参数：

- `initialCapacity`：初始总容量，默认16

- `loadFactor`：加载因子，默认0.75

- `concurrencyLevel`：并发级别，默认16

  其中并发级别控制了Segment的`个数`，在一个ConcurrentHashMap创建后`Segment`的个数是`不能变`的，`扩容`过程过改变的是`每个Segment的大小`。

  <br />

  

**关于分段锁**

​	段Segment继承了重入锁`ReentrantLock`，有了锁的功能，每个锁控制的是一段，当每个Segment越来越大时，`锁的粒度`就变得有些大了。

​	分段锁的优势在于保证在操作不同段 map 的时候可以并发执行，操作同段 map 的时候，进行锁的竞争和等待。这相对于直接对整个map同步synchronized是有优势的。

​	缺点在于分成很多段时会比较浪费内存空间(不连续，碎片化); 操作map时竞争同一个分段锁的概率非常小时，分段锁反而会造成更新等操作的长时间等待; 当某个段很大时，分段锁的性能会下降。	jdk1.8的map实现和hashmap一样,jdk 1.8中ConcurrentHashmap采用的底层数据结构为数组+链表+红黑树的形式。数组可以扩容，链表可以转化为红黑树。

<br />



**什么时候扩容？**

- 当前容量超过阈值
- 当链表中元素个数超过默认设定（8个），当数组的大小还未超过64的时候，此时进行数组的扩容，如果超过则将`链表转化成红黑树`

**什么时候链表转化为红黑树？**

- 当`数组大小已经超过64`并且`链表中的元素个数超过默认设定（8个）`时，将链表转化为红黑树

<br />



腾讯云大佬的上面代码的讲解
![在这里插入图片描述](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/20200415041158669.png)

==把数组中的每个元素看成一个桶。可以看到大部分都是`CAS`操作，加锁的部分是对`桶的头节点进行加锁`，`锁粒度很小`。==



**为什么不用ReentrantLock而用synchronized ?**

减少内存开销:如果使用ReentrantLock则需要节点`继承AQS`来获得同步支持，增加内存开销，而1.8中只有头节点需要进行同步。
内部优化:synchronized则是JVM直接支持的，JVM能够在运行时作出相应的优化措施：锁粗化、锁消除、锁自旋等等。





## 9、ConcurrentHashMap为什么高效？

　　Hashtable低效主要是因为所有访问Hashtable的线程都争夺一把锁。如果容器有很多把锁，每一把锁控制容器中的一部分数据，那么当多个线程访问容器里的不同部分的数据时，线程之前就不会存在锁的竞争，这样就可以有效的提高并发的访问效率。

　　这也正是ConcurrentHashMap使用的分段锁技术。将ConcurrentHashMap容器的数据分段存储，每一段数据分配一个Segment（锁），当线程占用其中一个Segment时，其他线程可正常访问其他段数据。

