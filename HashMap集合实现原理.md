# Java集合实现原理

### HashMap的实现原理概括

- HashMap是基于哈希表的Map接口的非同步实现，允许使用null值和null键，但不保证映射的顺序。

- 底层使用数组实现，数组中每一项是个链表，即数组和链表的结合体。

- HashMap在底层将key-value当成一个整体进行处理，这个整体就是一个Entry对象。HashMap底层采用一个Entry[]数组来保存所有的key-value对，当需要存储一个Entry对象时，会根据key的hash算法来决定其在数组中的存储位置，在根据equals方法决定其在该数组位置上的链表中的存储位置；当需要取出一个Entry时，也会根据key的hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。

  ```java
  static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    /**
           * Creates new entry.
           */
    Entry(int h, K k, V v, Entry<K,V> n) {
      value = v;
      next = n;
      key = k;
      hash = h;
    }

    public final K getKey() {
      return key;
    }

    public final V getValue() {
      return value;
    }

    public final V setValue(V newValue) {
      V oldValue = value;
      value = newValue;
      return oldValue;
    }

    public final boolean equals(Object o) {
      if (!(o instanceof Map.Entry))
        return false;
      Map.Entry e = (Map.Entry)o;
      Object k1 = getKey();
      Object k2 = e.getKey();
      if (k1 == k2 || (k1 != null && k1.equals(k2))) {
        Object v1 = getValue();
        Object v2 = e.getValue();
        if (v1 == v2 || (v1 != null && v1.equals(v2)))
          return true;
      }
      return false;
    }

    public final int hashCode() {
      return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
      return getKey() + "=" + getValue();
    }

    /**
           * This method is invoked whenever the value in an entry is
           * overwritten by an invocation of put(k,v) for a key k that's already
           * in the HashMap.
           */
    void recordAccess(HashMap<K,V> m) {
    }

    /**
           * This method is invoked whenever the entry is
           * removed from the table.
           */
    void recordRemoval(HashMap<K,V> m) {
    }
  }
  ```

- HashMap进行数组扩容需要重新计算扩容后每个元素在数组中的位置，很耗性能。源码如下：

  ```java
  void addEntry(int hash, K key, V value, int bucketIndex) {
          if ((size >= threshold) && (null != table[bucketIndex])) {
              resize(2 * table.length);
              hash = (null != key) ? hash(key) : 0;
              bucketIndex = indexFor(hash, table.length);
          }

          createEntry(hash, key, value, bucketIndex);
      }
  ```

  ```java
  void resize(int newCapacity) {      //传入新的容量
          Entry[] oldTable = table;   //应用扩容前的Entry数组
          int oldCapacity = oldTable.length;
          if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容钱的数组大小如果已经达到最大的[JDK1.8（2^30）][JDK1.7（1 << 30）]了
              threshold = Integer.MAX_VALUE;  //修改阀值为int 的最大值
              return;
          }

          Entry[] newTable = new Entry[newCapacity];
          transfer(newTable, initHashSeedAsNeeded(newCapacity));
          table = newTable;
          threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1); //修改阀值
      }
  ```

  ```java
  void transfer(Entry[] newTable, boolean rehash) {
          int newCapacity = newTable.length;
          for (Entry<K,V> e : table) {
              while(null != e) {
                  Entry<K,V> next = e.next;
                  if (rehash) {
                      e.hash = null == e.key ? 0 : hash(e.key);
                  }
                  int i = indexFor(e.hash, newCapacity);
                  e.next = newTable[i];
                  newTable[i] = e;
                  e = next;
              }
          }
      }
  ```

  ​

- 采用了Fail-Fast机制，通过一个modCount值记录修改次数，对HashMap内容的修改都将增加这个值。迭代器初始化过程中会将这个值赋给迭代器的expectedModCount，在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map，马上抛出异常。

- 在[Java](http://lib.csdn.net/base/javase)编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的[数据结构](http://lib.csdn.net/base/datastructure)都可以用这两个基本结构来构造的，HashMap也不例外。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结构，但是在jdk1.8里加入了红黑树的实现，当链表的长度大于8时，转换为红黑树的结构。

  ![HashMap数据结构](http://img.blog.csdn.net/20161222113920705?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmpzZTUx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 从上图中可以看出，java中HashMap采用了链地址法。链地址法，简单来说，就是数组加链表的结合。在每个数组元素上都一个链表结构，当数据被Hash后，得到数组下标，把数据放在对应下标元素的链表上。

- ### HashMap的put方法实现

  - ### put函数大致的思路为：

    - 对key的hashCode()做hash，然后再计算index;
    - 如果没碰撞直接放到bucket里；
    - 碰撞了，以链表的形式存在buckets后；
    - 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；
    - 如果节点已经存在就替换old value(保证key的唯一性)
    - 如果bucket满了(超过load factor*current capacity)，就要resize。

  - 具体代码实现如下：

    ```java
     public V put(K key, V value) {

            return putVal(hash(key), key, value, false, true);
        }
        /**
        *生成hash的方法
        */
        static final int hash(Object key) {
            int h;
            return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
        }

        final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
            Node<K,V>[] tab; Node<K,V> p; int n, i;
            //判断table是否为空，
            if ((tab = table) == null || (n = tab.length) == 0)
                n = (tab = resize()).length;//创建一个新的table数组，并且获取该数组的长度
            //根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加   
            if ((p = tab[i = (n - 1) & hash]) == null)
                tab[i] = newNode(hash, key, value, null);
            else {//如果对应的节点存在
                Node<K,V> e; K k;
                //判断table[i]的首个元素是否和key一样，如果相同直接覆盖value
                if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                    e = p;
                //判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对
                else if (p instanceof TreeNode)
                    e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
               // 该链为链表
                else {
                //遍历table[i]，判断链表长度是否大于TREEIFY_THRESHOLD(默认值为8)，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；
                    for (int binCount = 0; ; ++binCount) {
                        if ((e = p.next) == null) {
                            p.next = newNode(hash, key, value, null);
                            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                                treeifyBin(tab, hash);
                            break;
                        }
                        if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                            break;
                        p = e;
                    }
                }
                // 写入
                if (e != null) { // existing mapping for key
                    V oldValue = e.value;
                    if (!onlyIfAbsent || oldValue == null)
                        e.value = value;
                    afterNodeAccess(e);
                    return oldValue;
                }
            }
            ++modCount;
            // 插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容
            if (++size > threshold)
                resize();
            afterNodeInsertion(evict);
            return null;
        }
    ```

- ### HashMap的get方法实现

  - bucket里的第一个节点，直接命中；

  - 如果有冲突，则通过key.equals(k)去查找对应的entry 
    若为树，则在树中通过key.equals(k)查找，O(logn)； 
    若为链表，则在链表中通过key.equals(k)查找，O(n)。

    ```java
    public V get(Object key) {
            Node<K,V> e;
            return (e = getNode(hash(key), key)) == null ? null : e.value;
        }

    final Node<K,V> getNode(int hash, Object key) {
            Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
            if ((tab = table) != null && (n = tab.length) > 0 &&
                (first = tab[(n - 1) & hash]) != null) {
                // 直接命中
                if (first.hash == hash && // 每次都是校验第一个node
                    ((k = first.key) == key || (key != null && key.equals(k))))
                    return first;
               // 未命中
                if ((e = first.next) != null) {
                // 在树中获取
                    if (first instanceof TreeNode)
                        return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                    // 在链表中获取
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                            return e;
                    } while ((e = e.next) != null);
                }
            }
            return null;
        }
    ```

    ​

### LinkedList实现原理要点概括  

- LinkedList是List接口的双向链表非同步实现，并允许包括null在内的所有元素。
- 底层的数据结构是基于双向链表的，该数据结构我们称为节点。
- 双向链表节点对应的类Entry的实例，Entry中包含成员变量：previous，next，element。其中，previous是该节点的上一个节点，next是该节点的下一个节点，element是该节点所包含的值。

### Hashtable实现原理要点概括  

- Hashtable是基于哈希表的Map接口的同步实现，不允许使用null值和null键。
- 底层使用数组实现，数组中每一项是个单链表，即数组和链表的结合体。
- Hashtable在底层将key-value当成一个整体进行处理，这个整体就是一个Entry对象。Hashtable底层采用一个Entry[]数组来保存所有的key-value对，当需要存储一个Entry对象时，会根据key的hash算法来决定其在数组中的存储位置，在根据equals方法决定其在该数组位置上的链表中的存储位置；当需要取出一个Entry时，也会根据key的hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。
- synchronized是针对整张Hash表的，即每次锁住整张表让线程独占。

### ConcurrentHashMap实现原理要点概括  

- ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术。
- 它使用了多个锁来控制对hash表的不同段进行的修改，每个段其实就是一个小的hashtable，它们有自己的锁。只要多个并发发生在不同的段上，它们就可以并发进行。
- ConcurrentHashMap在底层将key-value当成一个整体进行处理，这个整体就是一个Entry对象。Hashtable底层采用一个Entry[]数组来保存所有的key-value对，当需要存储一个Entry对象时，会根据key的hash算法来决定其在数组中的存储位置，在根据equals方法决定其在该数组位置上的链表中的存储位置；当需要取出一个Entry时，也会根据key的hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。
- 与HashMap不同的是，ConcurrentHashMap使用多个子Hash表，也就是段(Segment)。
- ConcurrentHashMap完全允许多个读操作并发进行，读操作并不需要加锁。如果使用传统的技术，如HashMap中的实现，如果允许可以在hash链的中间添加或删除元素，读操作不加锁将得到不一致的数据。ConcurrentHashMap实现技术是保证HashEntry几乎是不可变的。

### HashSet实现原理要点概括  

- HashSet由哈希表(实际上是一个HashMap实例)支持，不保证set的迭代顺序，并允许使用null元素。
- 基于HashMap实现，API也是对HashMap的行为进行了封装，可参考HashMap。

### LinkedHashMap实现原理要点概括  

- LinkedHashMap继承于HashMap，底层使用哈希表和双向链表来保存所有元素，并且它是非同步，允许使用null值和null键。
- 基本操作与父类HashMap相似，通过重写HashMap相关方法，重新定义了数组中保存的元素Entry，来实现自己的链接列表特性。该Entry除了保存当前对象的引用外，还保存了其上一个元素before和下一个元素after的引用，从而构成了双向链接列表。

### LinkedHashSet实现原理要点概括  

- 对于LinkedHashSet而言，它继承与HashSet、又基于LinkedHashMap来实现的。LinkedHashSet底层使用LinkedHashMap来保存所有元素，它继承与HashSet，其所有的方法操作上又与HashSet相同。