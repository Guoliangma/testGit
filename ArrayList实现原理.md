# ArrayList实现原理要点概括

- ArrayList是List接口的可变数组非同步实现，并允许包括null在内的所有元素。

- 底层使用数组实现。

- 该集合是可变长度数组，数组扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量增长大约是其容量的1.5倍，这种操作的代价很高。

- 采用了Fail-Fast机制，面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

- ArrayList是一个动态数组，它提供了比数组Array更多的方法，动态增加，删除元素，改变容量。

- 代码：

  ```java
  public class ArrayListStudy {
      public static void main(String[] args) {
          ArrayList<Integer> arrayList = new ArrayList<Integer>();
          arrayList.add(1);
          arrayList.add(2);
          System.out.println("OUT1: " + arrayList);
          arrayList.set(0, 100);
          System.out.println("OUT2: " + arrayList);
          arrayList.remove(0);
          System.out.println("OUT3: " + arrayList);
          arrayList.clear();
          System.out.println("OUT4: " + arrayList);
      }
  }

  ----------
  OUT1: [1, 2]
  OUT2: [100, 2]
  OUT3: [2]
  OUT4: []
  ```

- ArrayList初始化的容量是0，查看源码可知：

  ```java
  //当容器为空的时候，用来初始化存储元素的数组，即为初始化elementData
  private static final Object[] EMPTY_ELEMENTDATA = {};
  //用来存储元素的数组
  private transient Object[] elementData;
  public ArrayList() {
          super();
          this.elementData = EMPTY_ELEMENTDATA;
      }
  ```

- 当前容量为0时，当我们开始添加元素，集合开始扩容到10

  ```java
  //记录当前容器内元素的个数
  private int size;
  public boolean add(E e) {
          ensureCapacityInternal(size + 1);  // Increments modCount!!
          elementData[size++] = e;
          return true;
      }
  private void ensureCapacityInternal(int minCapacity) {
          //如果容器的当前容量任然是0，那么就使用的容器默认容量10去扩容
          if (elementData == EMPTY_ELEMENTDATA) {
              minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
          }

          ensureExplicitCapacity(minCapacity);
      }
  private void ensureExplicitCapacity(int minCapacity) {
          //记录容器被操作的次数,该变量为父类AbstractList的变量
          modCount++;

          // overflow-conscious code
          if (minCapacity - elementData.length > 0)
              grow(minCapacity);
      }

  -----
    /**
       * Default initial capacity.
       */
      private static final int DEFAULT_CAPACITY = 10;
  ```

- 如果当前元素不是0，则扩容为当前容量的1.5倍

  ~~~java
  private void grow(int minCapacity) {
          // overflow-conscious code
          int oldCapacity = elementData.length;
          //新容量 = 旧容量 + （旧容量 / 2）即为扩容1.5倍【源码为了效率更快使用了位操作符】
          int newCapacity = oldCapacity + (oldCapacity >> 1);
          if (newCapacity - minCapacity < 0)
              newCapacity = minCapacity;
          if (newCapacity - MAX_ARRAY_SIZE > 0)
              newCapacity = hugeCapacity(minCapacity);
          // minCapacity is usually close to size, so this is a win:
          //调用工具类进行具体扩容操作
          elementData = Arrays.copyOf(elementData, newCapacity);
      }

  private static int hugeCapacity(int minCapacity) {
      if (minCapacity < 0) // overflow
          throw new OutOfMemoryError();
      //由此我们可以看出ArrayList的最大容量为2147483647【二十一亿四千七百四十八万三千六百四十七】
      //它是32位操作系统中最大的符号型整型常量
      return (minCapacity > MAX_ARRAY_SIZE) ?
          Integer.MAX_VALUE :
          MAX_ARRAY_SIZE;
  }
  ~~~

- 由此可知扩容时，容器重新创建了一个数组，容量为当前计算出来的容量，然后调用JDK底层的C++代码完成批量数组复制操作。 